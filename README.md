# ST-2022-Lab6
Program Security Detect


## Part 1

spec:

- 下面是常見的記憶體操作問題，請分別寫出有下列記憶體操作問題的簡單程式，並說明 Valgrind 和 ASan 能否找的出來

    - Heap out-of-bounds read/write
    - Stack out-of-bounds read/write
    - Global out-of-bounds read/write
    - Use-after-free
    - Use-after-return

<br />

ans:

- Valgrind

    - stack out-of-bounds r/w: no

        test2.c

        ```c
        #include<stdlib.h>
        #include<stdio.h>
        #include<string.h>

        int main(){
            int a[8] = {0};
            int b[8] = {0};
            a[8] = 0xcafe;
            return 0;
        }
        ```
        
        result

        ```
        $ gcc -o t3 test2.c
        $ valgrind ./t3

        ==7374== Memcheck, a memory error detector
        ==7374== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
        ==7374== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
        ==7374== Command: ./t3
        ==7374== 
        ==7374== 
        ==7374== HEAP SUMMARY:
        ==7374==     in use at exit: 0 bytes in 0 blocks
        ==7374==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
        ==7374== 
        ==7374== All heap blocks were freed -- no leaks are possible
        ==7374== 
        ==7374== For counts of detected and suppressed errors, rerun with: -v
        ==7374== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)

        ```





ASAN





<br />

## Part 2

spec:

- 寫一個簡單程式 with ASan，Stack buffer overflow 剛好越過 redzone(並沒有對 redzone 做讀寫)，並說明 ASan 能否找的出來？


![](https://i.imgur.com/xYJeRn5.png)


test.c

```c
#include<stdlib.h>
#include<stdio.h>
#include<string.h>

int main(){
    int a[8] = {0};
    int b[8] = {0};
    a[17] = 0xcafe;
    return 0;
}
```


```sh
$ gcc -o t5 test3.c
$ ./t5  # no error

$ gcc -fsanitize=address -o t6 test3.c
./t6    # no error 
```

Ans: ASAN 無法找出這種 error 






