---
title: Pwnable.kr writeup
description: an record of pwn learning
tags:
 - pwn
 - ongoing
---

<!-- write excerpt here -->

pwnable.kr题目记录

<!--more-->

 
[TOC]
## Rookiss
### brain fuck
利用对指针的上下越界，使得原本指向.bss段的指针p最终可以越过.data段，指向.got.plt表，从而修改对应的跳转地址，使得调用某函数时转而执行另一个函数的代码。
注意其中先要执行一次libc其中的函数，使得程序将libc加载进内存，这样便可以根据该函数的实际内存地址和其在libc中的偏移计算出libc加载到内存中的基址，进而可以得到如system等libc函数在内存中的地址
偏移可以直接读取libc.so，查看相应的符号的偏移
Nothing can't be filled in, no matter main() / system() / basic func...
### md5 calculator
问题出在process_hash函数对字符串长度的检查上，最大0x400长度的字符串经过一次base64解码后塞进了位置在ebp-0x20C的栈上，因此可以想到使用buffer overflow来完成这次攻击，但是又遇到了一个问题：
==使用了canary==
因此又要想如何绕过canary，主要就是两种办法：
1. 一种是想办法得到canary的值，这样就可以在overflow的时候填入正确的canary值绕过检查
2. 另一种是修改系统发现canary错误时调用的___stack_chk_fail函数所指向的地址，使得其执行其他的代码

本题发现较难修改.got.plt表，所以采用第一种
> 一个程序一次启动中使用的canary值都是相同的

发现在另一个函数my_hash中计算MD5值时正好还使用到了canary值计算=.=，所以同步服务器时间，得到相同的srand种子，对my_hash中的值进行逆运算就可以得到canary值
### simple login
输入为一个经过base64解码后长度不超过12字节的字符串，填到一个位置在ebp-0x8的栈上
只能覆盖到ebp
> leave和retn表示的实际操作

覆盖ebp，使得leave时的esp为输入的input地址，在经过pop一次ebp之后，最后retn的地址实际在input addr+0x4，把程序中调用system('/bin/sh')的语句地址填进去就可以了
Like the flag said:  control EBP, control ESP, control EIP, control the world~
### otp
感觉这个和Toddler's battle的题目一样，都有点套路=.=
主要是利用没有对fopen、fread、fwrite、fclose这几个函数的返回值做检查的漏洞，通过调用一下bash命令限制程序的文件创建，导致文件无法创建，最后读取到的密码为空，就可以直接通过检查
限制创建文件个数命令：ulimit -f [maxnum]
为什么最后还需要将程序运行的stderr重定向到stdout？ 
**Emmm... That's a question, wait for me testing it.**
### ascii_easy
思路似乎是利用的bof，需要采用输入可见字符的方式进行填充
而且由于CVE-2016中的dirtycow漏洞，pwnable.kr已经更换了服务器，这道题的binary也与一年前的不同，exp也和之前的writeup中的有所变化。
同时，不能再使用ulimit -s unlimited这个方法关闭ASLR了，因为这会造成mmap发生错误
**目前还没有实现成功=.=**
**目前仍没有实现成功=.= 2018.2.2**
### tiny_easy
连续经历几个问题之后总算又能参考着做出一道题，自己是真的菜
程序很简单，开头就是两个pop，argc和argv*都出栈，最后是执行了argv[0][0..3]这一段所指向的地址的代码，因此一般的程序直接执行是会报segment fault错误的（一般的argv[0]为程序执行的路径）
用gdb打开，输入m指令，可以发现stack没有开NX，因此可以利用将之前所说的那一段地址改为栈上的地址，就可以利用输入的程序参数如argv，envp来在栈上执行代码了
但是如何确定地址？一般的程序都开启了ASLR，因此可以采用一种非常暴力的方法：
程序执行中，0x90为NOP，CPU在执行NOP时不进行任何操作，直接继续执行下一条指令
因此，可以构造一个非常长的argv[1] or envp，在payload的前段设为nop（一般长度可以为几千到上十万，因为栈的大小为0x21000（在我的电脑一次实验中）），最后加上shellcode。这样，一次跳到栈上时，就很容易执行到nop，然后一直跑到shellcode，执行system('/bin/sh')
因为需要ssh远程登录，所以这里就不使用pwntools了，可以使用如下一行来设置argv（or envp）执行程序：
```python
p = subprocess.Popen([jumpto,payload], executable="/home/tiny_easy/tiny_easy")
myenv={"code":payload}
p = subprocess.Popen([jumpto], executable="/home/tiny_easy/tiny_easy",env=myenv)
```
第一个列表为argv**，这里可以对argv[0]进行设置，也可以改变len(list)来对argv[1]/[2]/...进行设置，也可以对env参数进行设置，从而改变（添加）程序执行时的envp
[python的subprocess.Popen参考](blog.csdn.net/g457499940/article/details/17068277)
Example：
```
[-------------------------------------code-------------------------------------]
   0x804804e:	add    BYTE PTR [eax],al
   0x8048050:	add    BYTE PTR [eax],dl
   0x8048052:	add    BYTE PTR [eax],al
=> 0x8048054:	pop    eax
   0x8048055:	pop    edx
   0x8048056:	mov    edx,DWORD PTR [edx]
   0x8048058:	call   edx
   0x804805a:	add    BYTE PTR [eax],al
[------------------------------------stack-------------------------------------]
0000| 0xffffd110 --> 0x2 
0004| 0xffffd114 --> 0xffffd2ee ("/home/cpegg/practice/Rookiss/tiny_easy/tiny_easy")
0008| 0xffffd118 --> 0xffffd31f ("iu4htiuwrgbiuerbfdsuigaerukgaeuorgbauodgbadfuogbadfoglda")
0012| 0xffffd11c --> 0x0 
0016| 0xffffd120 --> 0xffffd358 ("CLUTTER_IM_MODULE=ibus")
0020| 0xffffd124 --> 0xffffd36f ("LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc"...)
0024| 0xffffd128 --> 0xffffd92b ("LC_MEASUREMENT=zh_CN.UTF-8")
0028| 0xffffd12c --> 0xffffd946 ("LESSCLOSE=/usr/bin/lesspipe %s %s")
[------------------------------------------------------------------------------]
```
断点设在了EP，这里的
- 0xffffd110为argc
- 0xffffd114为argv[0] (一般为path)
- 0xffffd118为argv[1] (一般为命令行执行时后面附带的参数，可有多个)
- 0xffffd120往后则是envp （一般为系统变量），可以通过sh命令env查看
### fsb
Format String Vulnerability
 主要还是要知道format string的几个重要的用法：
 - %n 能够将printf中已经输出的字符的个数放到参数所指的地址空间中（即将参数视为指针，修改指针所指的值）
 - %hn 表示对所指的空间的低4个字节做处理，这个常发生在对高地址不需做修改的情况下，可以大幅度减少输出字符的个数，从而加快exp的速度，但是在加[x]$时计算偏移仍按4字节计算偏移
 - %[x]$n 则表示对第x个参数做处理，这里的n也可以是x，d等等

做法应该是比较多的，毕竟也给出了4次输入fs的机会，很多writeup都只用到了3次就足够了
另外注意send与process的输出是不同步的，在这一题中就很明显了
我参考的做法，是利用main的ebp做跳板：
 - 得到fsb在执行printf时的esp以及此时的main的ebp，计算二者的offset
 - 修改main的ebp所指向的地址为sleep的got表
 - 再修改sleep的got表（利用第一步得到的offset就可以对ebp指向的got表指向的地址作修改），使其指向程序中执行execve('/bin/sh')的地方
 - 最后一次输入随意，那么程序就会在输入结束后执行sleep时跳到执行get shell的地方了
 最后修改got表的时候因为二者的地址高4位都是0x0804，因此可以使用%hn来修改，大幅加快了执行速度

附可以使用format string的函数：
|  Func_name |  Func_util |
| :--------: | :--------: |
|  fprint  |  Writes the printf to a file  |
|printf |	Output a formatted string
|sprintf| 	Prints into a string|
|snprintf| 	Prints into a string checking the length|
|vfprintf| 	Prints the a va_arg structure to a file|
|vprintf| 	Prints the va_arg structure to stdout|
|vsprintf| 	Prints the va_arg to a string|
|vsnprintf| 	Prints the va_arg to a string checking the length|
### dragon
主要还是自己做题不够耐心吧......总是不能把程序的基本流程都好好看完，也算是给自己留个教训。
首先发现在选英雄时候可以选3进入secret mode，但是没什么卵用，因为输入的字符串最长只有10bytes，但是却要和一个30多个字节的字符串比较，因此比较是肯定过不了的。唯一有用的是出现了system('/bin/sh')，考虑从别的地方直接跳转过来
然后在KnightAttack和PriestAttack中都有发现call eax的存在，因此去找eax的来源，发现是来自malloc出来的堆空间，因此考虑修改eax的来源为shell的地址
再纵观整个游戏过程，发现在击败dragon之后有字符串输入（16bytes），但是这个字符串居然要输入到一个malloc出来的堆空间中，并且堆大小还和monster_info malloc出的一样都是16bytes，而且之后又从monster_info中取地址call eax
因此猜想是不是可以先释放掉monster_info的chunk，再malloc出同样大小的chunk，这样就可以把输入的16bytes字符串中的前8位作为eax的值，再call eax进行跳转了。果然在两种attack的结束部分都发现了free，再找一下就知道是free掉了monster_info的chunk
所以思路其实比较简单，战胜dragon之后输入shell的地址就可以了
战胜dragon则又涉及到了整数溢出的问题：龙会回血，所以当血量过高时就会小于0。从整个游戏过程中似乎并没有自己不掉血而龙可以一直加血加到爆掉的办法，但是...
```
.text:08048C54                 mov     eax, [ebp+arg_1]
.text:08048C57                 movzx   eax, byte ptr [eax+8] ; Move with Zero-Extend
.text:08048C5B                 mov     edx, eax
.text:08048C5D                 mov     eax, [ebp+arg_1]
.text:08048C60                 movzx   eax, byte ptr [eax+9] ; Move with Zero-Extend
.text:08048C64                 add     eax, edx        ; Add
.text:08048C66                 mov     edx, eax
.text:08048C68                 mov     eax, [ebp+arg_1]
.text:08048C6B                 mov     [eax+8], dl
```
这里用的是dl，低8位，所以血量上限其实只有128，而不是一般的32768，而且在之后是使用的jg的有符号比较而不是ja无符号比较，因此有当eax等于0x80时test al,al 之后不进行jg跳转
因此就只要找一种能使血量爆掉的办法了：通过打Mama Dragon，血量初始为80，每回合加4点，再选priest，有mp就用holyshield，mp耗尽就用clarity恢复mp，这样刚好能撑过dragon的血量上限，之后就再输入shellcode的地址0x8048DBF就可以咯
参考：[各种跳转指令](http://www.cnblogs.com/del/archive/2010/04/16/1713886.html)
### fix
这个题目的原因还是很容易可以gdb出来：虽然把shellcode成功写道了栈上，但是由于shellcode里太多push语句，导致push进去的值改写掉了在栈上的code
所以关键是需要对shellcode做一点修改，涉及到 int 0x80 (syscall) 的相关执行
原来的shellcode：
```
xor    %eax,%eax
push   %eax
push   $0x68732f2f
push   $0x6e69622f
mov    %esp,%ebx
push   %eax
push   %ebx
mov    %esp,%ecx
mov    $0xb,%al
int    $0x80
```
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"
23bytes
网上有很多人采用第一种办法：
将shellcode[15]改成92（0x5C），这样代码会变成：
```
xor    eax,eax
push   eax
push   0x68732f2f
push   0x6e69622f
mov    ebx,esp
pop    esp
push   ebx
mov    ecx,esp
mov    al,0xb
int    0x80
```
这样居然也是一个shellcode，但是要注意此时要开 ulimit -s unlimited
这里附官网的intended_solution：
> 1. change 15th index of shellcode into 0xc9 (201)
> 2. create symlink for a shell script input file:
> ln -s /tmp/fix8/a.sh `perl -e 'print"\x83\xc4\x10\x83\xec\x0c\x50\xe8\x4d\x61\x01"'`` (去掉最后一个点号)
> 3. put whatever commands in a.sh. these commands will be executed as fix_pwn

此时的shellcode为：
```
   0xffb89dcc:	xor    eax,eax
   0xffb89dce:	push   eax
   0xffb89dcf:	push   0x68732f2f
   0xffb89dd4:	push   0x6e69622f
   0xffb89dd9:	mov    ebx,esp
   0xffb89ddb:	leave  
   0xffb89ddc:	push   ebx
   0xffb89ddd:	mov    ecx,esp
   0xffb89ddf:	mov    al,0xb
   0xffb89de1:	int    0x80
```
### syscall
系统内核级别的漏洞利用
在虚拟机环境中添加了一个新的系统调用：sys_upper(char* in, char* out)，但是其并没有做任何的检查，因此我们可以通过控制in中的内容和out的指针地址来修改到out指向的地址中的内容
首先，根据/proc/kallsys中的内核符号表得到几个关键的内核函数的地址
- vmalloc-exec：可以在内核地址中分配一块指定大小的可执行内存空间
- PREPARE_KERNEL_CRED & COMMIT_CREDS：可以说是内核exploit的最终固定动作了：从内核获取root的执行权限表(cred)，再为现在执行的程序注册该权限表(即给予当前程序的root权限)

因此可以采取如下的思路（方法不止一种，除了vmalloc-exec的方法还有很多）：
1. 首先在内核中分配一块可执行空间，作为shellcode
shellcode的内容为一段汇编代码：
```
    stmfd sp!, {r4, lr}
    mov r0, #0
    ldr r4, =0x8003f924 # PREPARE_KERNEL_CRED 地址
    blx r4
    ldr r4, =0x8003f56c # COMMIT_CREDS 地址
    blx r4
    ldmfd sp!, {r4, pc}
```
即获取权限表-注册权限表
2. 然后将这个写入了shellcode的buffer的地址写到系统调用表中（任意取一个没用过的系统调用号）
3. 最后调用这个系统调用即可
### crypto1
真正的brute-force
题目中要求先输入id和pwd，之后对一个字符串：id-pwd-cookie作AES的CBC加密，cookie未知。然后将加密的字符串发送给服务器验证，服务器解密，先验证cookie是否相同，再验证id+cookie的sha256值是否和pwd一样，一样就可以拿到flag
所以思路就是先想办法得到cookie，然后计算一下id+cookie的sha256值当作pwd就可以通过验证。
由于采用的是CBC分组加密模式，初始向量IV是写死的，因此只要保证字符串的前16位相同，那么加密出来的结果的前16位就是相同的；保证前32位相同，出来的结果前32位就是相同的
因此可以将pw置空，通过一位位置换id的末尾，比较加密出来的结果和不置换的结果就可以逐渐得到所有的cookie
### echo1
关键在于.bss段的cs:id这个地址是不变的，之前试用ulimit卡aslr没成功，system地址还是随机的
一开始还以为是利用的uaf做......gdb调试，发现栈和.bss段都可写可执行，于是思路如下：
bof到cs：id，在这里放一个汇编语句jmp rsp
然后在bof的时候在rip的后面放shellcode，反正可以读80h长度的字符串，shellcode可以放0x80-44=84 bytes
在 http://shell-storm.org/shellcode 上找一下64位的shellcode，很容易找到符合长度的
other points:
context(arch='amd64')设置64位环境
使用p64而不是p32
###echo2
题目里很明显的告知了使用UAF来做，关键是怎么用
选择4但不退出可以构成一个迷途指针，echo3可以对迷途指针指向的堆空间的前0x20位作修改，实际上刚好可以覆盖到是堆中的greetings地址，但是不能再像echo1里面一样覆盖后面为shellcode了
所以要想办法跳转到0x20个bytes的前面来，就需要获得rsp/rbp，但是rsp又是随机的
继续看echo2，发现里面有格式化字符串的输出printf(str)，那么就有办法了：
首先调用echo2，输出此时的rbp，这个应与echo3的相同
输出rbp使用%p或者%lx，参数需要多试几次，最后试到%8$lx的样子差不多就到栈上了
计算得到最开始输入名字的栈addr，因此在程序开始输入名字时输入shellcode (23bytes<32bytes)
另一个shellcode网站https://www.exploit-db.com/ 这里有一个长度仅为21bytes的code
释放掉id的chunk，调用echo3，填入shellcode并修改greetings地址为input_buffer_addr
再跑一次echo2或者3，get shell

这里最开始我写错了地方，写到了echo2的栈上input_buffer (rbp-0x60)，结果发现main函数里的puts会改掉我的shellcode，就比较无奈了(sigh)
经验就是盯住输入长度至少在0x18字节以上的输入=.=
### rsa_calculator
其实就是对format string作了一个rsa的外包装。
首先，%d和%u要区分开来，一个是读入有符号整数，另一个是无符号的，因此在之后进行有/无符号大小比较时要注意是否发生错乱。（这里同时参考tiny_easy的各种跳转指令）
这又会带来怎样的问题呢？
- 原本有符号比较小于的结果（如-1<1），到了无符号比较结果就反了过来（此时为maxint>1）
- 循环变量使用有符号比较，结果初始值就为-1，导致循环次数远超过原本的次数
- ...==(待补充)==

这道题目中有两处涉及到了这个问题：一个是在目录选择上使用了jle，导致选择0和-1也可以通过，但这里没办法exp；另一处则是在encrypt函数中，输入的字符串长度是有符号整数，比较也是用的有符号比较最大值，但是循环读入的方式却是循环变量逐渐减一，最后比较自己是不是为零，这样当我们输入字符串长度为-1时，就可以完成一次bof
同时，这个程序没有开NX，所以shellcode直接写在栈上
最后通过fsb得到推算出shellcode的addr，同时也可以得到canary的值，然后就可以在encode的输入环节一波带走了
### note
在代码note.c中，可以看到使用了MAP_FIXED这样的强制mmap的选项，同时再加上从/dev/urandom中读随机数的随机产生办法，这样mmap出来的空间会在整个内存空间中分布
> 那会带来什么影响?

最重要的影响来自栈上，如果mmap到了栈上会发生什么？
可能有人会觉得mmap会map到stack相邻的内存区域上，实则不然，`man mmap`有：
> MAP_FIXED
> Don't  interpret  addr  as  a hint: place the mapping at exactly that address.  addr must be a multiple of the page size.  *If the memory region specified by addr and len overlaps  pages of any existing mapping(s), then the overlapped part of the existing mapping(s) will be discarded.*  If the specified address cannot be used, mmap()  will fail.   Because requiring a fixed address for a mapping is less ortable, the use of this option is discouraged.
> 
> MAP_STACK (since Linux 2.6.27)
> Allocate the mapping at an address suitable for a process  or  thread  stack.   This flag is currently a no-op, but is used in the glibc threading implementation so that if some architectures require special treatment for stack allocations,  support  can later be transparently implemented for glibc.

因此mmap可以到栈上，并且栈也会继续使用mmap中的空间，那么思路就出来了：
首先由于它把mmap的地址告诉你了，那么如果这个地址在栈地址附近（由于系统开了aslr，实际上栈地址还是有随机），那么就全部填上某个返回地址，让其返回到某个其他地址的mmap空间x中
然后在x里填上shellcode即可

*plus 这里可以通过多次重复调用目录函数select_menu()来增大栈内存分布，从而让mmap到栈上的概率更大 *
*同时，由于可能判断的范围错误，程序可能会崩溃，所以写一个while try catch循环来跑一下*
### alloca
对alloca()函数的exploit
该函数是在栈上分配与输入参数大小相应的内存空间作为调用alloca的函数的局部变量，但是有一个很致命的缺陷：
不检查输入参数的正负，即便其声明时使用的是size_t(unsigned int)
但是如果你给一个负数进去，esp会向高地址走，设置得当，就可以改到返回地址（和输入参数，32b）了
这里有一种比较新奇的做法：
在p=process('./alloca')时，可以在后面加上程序运行的环境变量env：
`p=process('./alloca',env={'xxx':xxx})`
那么如果我们设置几个环境变量和其值为如下：
` 'x1':p32(call_system_addr)*30000 `
` 'x2':p32(call_system_addr)*30000 `
` 'x3':p32(call_system_addr)*30000 `
` 'x4':p32(call_system_addr)*30000 `
` 'x5':p32(call_system_addr)*30000 `
`... `
这样多来几下，最后在栈上的很高的地址处会出现大面积连续的p32(call_system_addr)这样只要我们将返回地址设为一个接近环境变量区域的值，就基本可以避免aslr的影响了

*注意环境变量不影响对栈上返回地址的修改，只是对其具体修改的值有影响，返回地址的修改必须由对alloca输入数据的精心构造来完成，如本题中为alloca(-68)，其他的值都修改不到返回地址* 