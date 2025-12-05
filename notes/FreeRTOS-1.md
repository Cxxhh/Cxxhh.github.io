# RTOS学习-堆与栈
##  堆
堆是一个空闲的内存，用户可以手动申请和释放空间。
我们创建的 如：`char a[10];` 就是放置在堆中。
```

char heap_buf[1024]; // 分配堆内存
int pos = 0; //堆指针
void *my_malloc(int size){
    int old_pos = pos ; //保存当前堆位置
    pos = pso +size; //更新堆位置
    retur $heap_buf[old_pos]; //返回申请的空间
}

int main(){
    char ch = 65; //'A'
    char *buf = my _malloc(100); //分配100字节 0x200~0x264
    char *buf2 = my_malloc(100);//0x265~0x2c4
    uint8_t uch  =200;
    for(int i = 0; i < 100; i++)
    {
        buf[i] = ch+i;   //往堆内存写入数据
    }
    forr(int i = 0; i < 100; i++)
    {
        buf2[i] = uch+i; //往堆内存写入数据
    }
}
```
## 栈
 栈是函数运行时使用的内存，栈内存是自动分配的，函数调用结束后，栈内存会自动释放。

 栈内存是存放函数的参数，局部变量，返回地址等信息。
主函数在调用函数时，会将这个函数下一条指令的地址存入栈中。
然后跳转到调用函数的地址执行。完成后会将栈中的地址取出，跳转到调用函数的下一条指令继续执行。 
```
void AFUN(void) {
    int a = 10;
    int b = 20;
    
    // 假设这条指令的地址是 0x1000
    BFUN();  // ← 调用BFUN
    
    // 假设这条指令的地址是 0x1004 ← 这就是返回地址！
    int c = a + b;  // BFUN返回后，继续执行这里
    
    // 地址 0x1008
    return;
}

void BFUN(void) {
    int x = 5;
    // ... 执行一些操作
    return;  // 返回到 0x1004（AFUN中调用BFUN的下一条指令）
}
```