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


    - Use-after-free: yes
    
        ```c
        #include<stdlib.h>
        #include<stdio.h>
        #include<string.h>

        int main(){
            char *str = malloc(4);
            str[3] = 'a';
            free(str);
            printf("%c\n", str[3]);
            return 0;
        }
        ```


        ```
        $ gcc -o t4 test4.c
        $ valgrind ./t4

        ==7637== Memcheck, a memory error detector
        ==7637== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
        ==7637== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
        ==7637== Command: ./t4
        ==7637== 
        ==7637== Invalid read of size 1
        ==7637==    at 0x10870F: main (in /home/oceane/software-testing/part1/t4)
        ==7637==  Address 0x522f043 is 3 bytes inside a block of size 4 free'd
        ==7637==    at 0x4C32D3B: free (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
        ==7637==    by 0x108706: main (in /home/oceane/software-testing/part1/t4)
        ==7637==  Block was alloc'd at
        ==7637==    at 0x4C31B0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
        ==7637==    by 0x1086EB: main (in /home/oceane/software-testing/part1/t4)
        ==7637== 
        a
        ==7637== 
        ==7637== HEAP SUMMARY:
        ==7637==     in use at exit: 0 bytes in 0 blocks
        ==7637==   total heap usage: 2 allocs, 2 frees, 1,028 bytes allocated
        ==7637== 
        ==7637== All heap blocks were freed -- no leaks are possible
        ==7637== 
        ==7637== For counts of detected and suppressed errors, rerun with: -v
        ==7637== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)

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






