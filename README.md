# how2pwn
some pwn methods
## setresuid提权

setresuid（0,0,0）可以用来设置进程的EUID,实现为当前进程提权的目的。但是普通用户直接调用并不能实现提取，原因如下：
```
![](https://201579.image.myqcloud.com/201579/0/babb12d6-08df-4ae2-977a-699792268eab/original）
```
上图中对应了内核文件中setresuid的部分代码信息，通过分析可以发现，函数在真正进行setresuid之前会对调用者拥有的权限进行检查，如上图红框中的内容，满足调用权限时，R0的值为#0，对于普通用户的调用，R0是一个非零值，所以如果我们把比较的对象#0改成一个非零值，那么setresuid的可以成功调用进行置位。
至此，归结起来，我们要利用setresuid实现提权，需要解决如何修改内核文件的问题。步骤：
Step1：找到目标代码虚拟地址
Step2：找到目标代码的物理地址
Step3：利用漏洞修改内核文件
Step4：调用setresuid(0,0,0)提权
