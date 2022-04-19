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
        - ASAN 要加 flag 才找得到

<br />

ans:

- Valgrind

    - Heap out-of-bounds read/write: yes
    
        ```c
        #include<stdlib.h>
        #include<stdio.h>
        #include<string.h>
        #define BUFSIZE 5

        int main(int argc, char **argv) {
            char *buf;
            buf = (char *)malloc(sizeof(char)*BUFSIZE);
            strcpy(buf, argv[1]);
        }
        ```

        ```
        $ gcc -o heap heap.c
        $ valgrind ./heap helloworld

        ==8014== Memcheck, a memory error detector
        ==8014== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
        ==8014== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
        ==8014== Command: ./heap helloworld
        ==8014== 
        ==8014== Invalid write of size 1
        ==8014==    at 0x4C34E00: strcpy (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
        ==8014==    by 0x1086C0: main (in /home/oceane/software-testing/part1/heap)
        ==8014==  Address 0x522f045 is 0 bytes after a block of size 5 alloc'd
        ==8014==    at 0x4C31B0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
        ==8014==    by 0x1086A2: main (in /home/oceane/software-testing/part1/heap)
        ==8014== 
        ==8014== Invalid write of size 1
        ==8014==    at 0x4C34E0D: strcpy (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
        ==8014==    by 0x1086C0: main (in /home/oceane/software-testing/part1/heap)
        ==8014==  Address 0x522f04a is 5 bytes after a block of size 5 alloc'd
        ==8014==    at 0x4C31B0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
        ==8014==    by 0x1086A2: main (in /home/oceane/software-testing/part1/heap)
        ==8014== 
        ==8014== 
        ==8014== HEAP SUMMARY:
        ==8014==     in use at exit: 5 bytes in 1 blocks
        ==8014==   total heap usage: 1 allocs, 0 frees, 5 bytes allocated
        ==8014== 
        ==8014== LEAK SUMMARY:
        ==8014==    definitely lost: 5 bytes in 1 blocks
        ==8014==    indirectly lost: 0 bytes in 0 blocks
        ==8014==      possibly lost: 0 bytes in 0 blocks
        ==8014==    still reachable: 0 bytes in 0 blocks
        ==8014==         suppressed: 0 bytes in 0 blocks
        ==8014== Rerun with --leak-check=full to see details of leaked memory
        ==8014== 
        ==8014== For counts of detected and suppressed errors, rerun with: -v
        ==8014== ERROR SUMMARY: 6 errors from 2 contexts (suppressed: 0 from 0)
        ```




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


    - Global out-of-bounds read/write: no

        ```c
        #include<stdlib.h>
        #include<stdio.h>
        #include<string.h>

        int a[8] = {0};
        int b[8] = {0};

        int main(){
            a[9] = 0;
            return 0;
        }
        ```

        ```
        $ gcc -o global global.c
        $ valgrind ./global


        ==7768== Memcheck, a memory error detector
        ==7768== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
        ==7768== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
        ==7768== Command: ./global
        ==7768== 
        ==7768== 
        ==7768== HEAP SUMMARY:
        ==7768==     in use at exit: 0 bytes in 0 blocks
        ==7768==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
        ==7768== 
        ==7768== All heap blocks were freed -- no leaks are possible
        ==7768== 
        ==7768== For counts of detected and suppressed errors, rerun with: -v
        ==7768== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)

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

    - Use-after-return: no

        ```c
        #include<stdio.h>

        // stack-use-after-return error
        char* x;

        void foo() {
            char stack_buffer[42];
            x = &stack_buffer[13];
        }

        int main() {

            foo();
            *x = 42; // Boom!

            return 0;
        }
        ```

        ```
        $ valgrind ./use-after-return

        ==8092== Memcheck, a memory error detector
        ==8092== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
        ==8092== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
        ==8092== Command: ./use-after-return
        ==8092== 
        ==8092== 
        ==8092== HEAP SUMMARY:
        ==8092==     in use at exit: 0 bytes in 0 blocks
        ==8092==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
        ==8092== 
        ==8092== All heap blocks were freed -- no leaks are possible
        ==8092== 
        ==8092== For counts of detected and suppressed errors, rerun with: -v
        ==8092== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
        ```




- ASAN

    - Heap out-of-bounds read/write: yes
    
    
        ```
        $ gcc -fsanitize=address -o heap heap.c
        $ ./heap

        ASAN:DEADLYSIGNAL
        =================================================================
        ==8142==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x7fbaa9239461 bp 0x7fffaa8b7f10 sp 0x7fffaa8b7678 T0)
        ==8142==The signal is caused by a READ memory access.
        ==8142==Hint: address points to the zero page.
            #0 0x7fbaa9239460  (/lib/x86_64-linux-gnu/libc.so.6+0x18e460)
            #1 0x7fbaa9501ff5  (/usr/lib/x86_64-linux-gnu/libasan.so.4+0x65ff5)
            #2 0x562a0e65e8e0 in main (/home/oceane/software-testing/part1/heap+0x8e0)
            #3 0x7fbaa90ccc86 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x21c86)
            #4 0x562a0e65e7a9 in _start (/home/oceane/software-testing/part1/heap+0x7a9)

        AddressSanitizer can not provide additional info.
        SUMMARY: AddressSanitizer: SEGV (/lib/x86_64-linux-gnu/libc.so.6+0x18e460) 
        ==8142==ABORTING

        ```








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






