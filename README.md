# ST-2022-Lab6
Program Security Detect


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













