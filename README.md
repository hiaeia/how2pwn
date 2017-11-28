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


## 内核bug越界执行
### 模式
```
static int __sock_diag_rcv_msg(struct sk_buff *skb, struct nlmsghdr *nlh)
{
    int err;
    struct sock_diag_req *req = NLMSG_DATA(nlh);
    struct sock_diag_handler *hndl;

    if (nlmsg_len(nlh) < sizeof(*req))
        return -EINVAL;

    hndl = sock_diag_lock_handler(req->sdiag_family);
    if (hndl == NULL)
        err = -ENOENT;
    else
        err = hndl->dump(skb, nlh); //越界执行
        sock_diag_unlock_handler(hndl);
    return err;
}
static const inline struct sock_diag_handler *sock_diag_lock_handler(int family)
{
        if (sock_diag_handlers[family] == NULL)
                request_module("net-pf-%d-proto-%d-type-%d", PF_NETLINK,
                                NETLINK_SOCK_DIAG, family);

        mutex_lock(&sock_diag_table_mutex);
        return sock_diag_handlers[family];//越界.
}
```
### 漏洞利用
如果没有开启PXN的情况下：
    通过调试内核，找到sock_diag_handlers附近取值范围固定为一个较小的值[A , B]，这样将[A , B]这段内存mmap到用户空间，并在B端写入提权code，只要取合适的family，满足sock_diag_handlers[family]->dump刚好为那个固定的较小的值，利用滑梯到B执行提权code：[提权](https://my.oschina.net/fgq611/blog/181812)
    
如果开启PXN的情况下：
    可以尝试在内核态调用set_fs或者其他提权函数

## 
