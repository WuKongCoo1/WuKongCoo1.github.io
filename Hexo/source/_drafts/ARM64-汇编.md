---
title: ARM64 汇编
tags:
---

## 指令

bl：记录lr到pc 

## 寄存器

## fp（frame pointer）也就是x29

fp存储的是什么？通过fp来访问局部变量，栈帧的栈底指针

## lr（link register）也是就x30

用于存储指令的返回指令地址

## sp (stack pointer) 栈顶指针

sp来访问局部变量

## pc（program counter）

用于存储cpu要执行的下一条指令的地址



![image-20191211131509655](/Users/bytedance/Documents/Working/文档/images/image-20191211131509655.png)

#### 源码：

```c
void haha()
{
    int a = 5;
    int c = 6;
    return;
}

void hehe()
{
    int g = 5;
    int x = 6;
    haha();
    return;
}
```

#### 汇编

```assembly
_haha:                                  ; @haha
	sub	sp, sp, #16             ; =16
	mov	w8, #5
	str	w8, [sp, #12]
	.loc	2 14 9                  ; WKBacktraceLogger/CTest.c:14:9
	orr	w8, wzr, #0x6
	str	w8, [sp, #8]
	.loc	2 15 5                  ; WKBacktraceLogger/CTest.c:15:5
	add	sp, sp, #16             ; =16
	ret

_hehe:                                  ; @hehe
	sub	sp, sp, #32             ; =32
	stp	x29, x30, [sp, #16]     ; 16-byte Folded Spill
	add	x29, sp, #16            ; =16
Ltmp3:
	mov	w8, #5
	stur	w8, [x29, #-4]
	orr	w8, wzr, #0x6
	str	w8, [sp, #8]
	bl	_haha
	ldp	x29, x30, [sp, #16]     ; 16-byte Folded Reload
	add	sp, sp, #32             ; =32
	ret
```

分析图结果：


![image-20191211134850694](/Users/bytedance/Documents/Working/文档/images/image-20191211134850694.png)

### 参考资料

[ARM64 汇编——寄存器和指令](https://www.jianshu.com/p/2f4a5f74ac7a)

[ARMv8-aarch64寄存器和指令集](https://winddoing.github.io/post/7190.html)

[ARMv8-aarch64寄存器和指令集](https://winddoing.github.io/post/7190.html)

