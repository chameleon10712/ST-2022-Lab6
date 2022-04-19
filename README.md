# ST-2022-Lab6
Program Security Detect


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
gcc -o t5 test3.c
./t5
# no error

gcc -fsanitize=address -o t6 test3.c
./t6
# no error 

```


Ans: ASAN 無法找出這種 error 






