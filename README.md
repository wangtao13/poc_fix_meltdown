# poc_fix_meltdown
# A POC of fix to CPU MELTDOWN in Linux x86_64.

## Detailed description

NOTE: This fix POC could NOT fix MELTDOWN attack from Intel tsx.

This mainly focuses on the typical MELTDOWN POCs in the Internet, the basic logics is as follows.

1. Allocate memory buffer of 256 * PAGESIZE in user space.
2. FLUSH cache of the memory allocated.
3. Trigger CPU speculative execution.
4. Access kernel address, which will trigger SEGV later.
5. Read memory buffer indexed by kernel data (1 byte), which cause the user space address to be loaded into CPU cache.
6. Read that memory address, and check the access time, to know if it is cached or not. If it is cached, the address (index) is the byte of kernel data.
7. go to 2. to repeat reading next kernel address.

POC of the fix provided here is leveraging the point of 2 and 4.
As described before, without Intel tsx, the invalid access of kernel space from user space will be trapped to CPU and Linux kernel, which then send SIGSEGV to the mis-behaved process. So it means, kernel is aware of this.

Then, like point of 2, kernel can FLUSH CPU cache of the mis-behaved process, including the cache loaded in the point of 5.
With cache FLUSHing in kernel space, user space process could NOT peek kernel data from the point of 6.

## Testing the POC of fix.
* without the fix to MELTDOWN, running meltdown-exploit in my Ubuntu 16.10 VM (with VMware Workstation 14.1.0), and Ubuntu 16.04.2 LTS (in VMware ESXi 6.5), it shows followings.

    root@t-virtual-machine:/home/t/test/meltdown/meltdown-exploit-master# ./run.sh  
    looking for linux_proc_banner in /proc/kallsyms  
    cached = 33, uncached = 273, threshold 94  
    read ffffffffbd200060 = 25 % (score=77/1000)  
    read ffffffffbd200061 = 73 s (score=79/1000)  
    read ffffffffbd200062 = 20   (score=116/1000)  
    read ffffffffbd200063 = 76 v (score=73/1000)  
    read ffffffffbd200064 = 65 e (score=77/1000)  
    read ffffffffbd200065 = 72 r (score=108/1000)  
    read ffffffffbd200066 = 73 s (score=51/1000)  
    read ffffffffbd200067 = 69 i (score=47/1000)  
    read ffffffffbd200068 = 6f o (score=47/1000)  
    read ffffffffbd200069 = 6e n (score=16/1000)  
    read ffffffffbd20006a = 20   (score=30/1000)  
    read ffffffffbd20006b = 25 % (score=44/1000)  
    read ffffffffbd20006c = 73 s (score=71/1000)  
    read ffffffffbd20006d = 20   (score=38/1000)  
    read ffffffffbd20006e = 28 ( (score=8/1000)  
    read ffffffffbd20006f = 62 b (score=23/1000)  
    VULNERABLE  
    PLEASE POST THIS TO https://github.com/paboldin/meltdown-exploit/issues/19  
    VULNERABLE ON  
    4.8.0-59-generic #64-Ubuntu SMP Thu Jun 29 19:38:34 UTC 2017 x86_64  
    processor       : 0  
    vendor_id       : GenuineIntel  
    cpu family      : 6  
    model           : 78  
    model name      : Intel(R) Core(TM) i5-6300U CPU @ 2.40GHz  
    stepping        : 3  
    microcode       : 0x88  
    cpu MHz         : 2495.151  
    cache size      : 3072 KB  
    physical id     : 0  

* With the fix, it shows,  
    root@t-virtual-machine:/home/t/test/meltdown/meltdown-exploit-master# ./run.sh  
    looking for linux_proc_banner in /proc/kallsyms  
    cached = 37, uncached = 287, threshold 103  
    read ffffffffbd200060 = ff   (score=0/1000)  
    read ffffffffbd200061 = ff   (score=0/1000)  
    read ffffffffbd200062 = ff   (score=0/1000)  
    read ffffffffbd200063 = ff   (score=0/1000)  
    read ffffffffbd200064 = ff   (score=0/1000)  
    read ffffffffbd200065 = ff   (score=0/1000)  
    read ffffffffbd200066 = ff   (score=0/1000)  
    read ffffffffbd200067 = ff   (score=0/1000)  
    read ffffffffbd200068 = ff   (score=0/1000)  
    read ffffffffbd200069 = ff   (score=0/1000)  
    read ffffffffbd20006a = ff   (score=0/1000)  
    read ffffffffbd20006b = ff   (score=0/1000)  
    read ffffffffbd20006c = ff   (score=0/1000)  
    read ffffffffbd20006d = ff   (score=0/1000)  
    read ffffffffbd20006e = ff   (score=0/1000)  
    read ffffffffbd20006f = ff   (score=0/1000)  
    NOT VULNERABLE  
    PLEASE POST THIS TO https://github.com/paboldin/meltdown-exploit/issues/22  
    NOT VULNERABLE ON  
    4.8.0-59-generic #64-Ubuntu SMP Thu Jun 29 19:38:34 UTC 2017 x86_64  
    processor       : 0  
    vendor_id       : GenuineIntel  
    cpu family      : 6  
    model           : 78  
    model name      : Intel(R) Core(TM) i5-6300U CPU @ 2.40GHz  
    stepping        : 3  
    microcode       : 0x88  
    cpu MHz         : 2495.151  
    cache size      : 3072 KB  
    physical id     : 0  

## TODO:
1. May optimize kernel code for better performance.
2. Check if other SIGXXX should also FLUSH process cache in kernel space.

## References
### MELTDOWN
https://meltdownattack.com/meltdown.pdf

### MELTDOWN POCs
https://github.com/raphaelsc/Am-I-affected-by-Meltdown  
https://github.com/paboldin/meltdown-exploit

## Copyright notice
The code and idea herein is: Copyright Tao Wang, 2018
This POC of fix to CPU MELTDOWN in Linux X86_64 is indepdently developed by Tao Wang.

This fix is tested with above two MELTDOWN POCs, copyrights are owned by the POC contributors.
