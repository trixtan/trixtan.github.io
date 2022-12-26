---
layout: post
title:  "Understanding DWARF - Function arguments"
tags: [debug, DWARF]
---
In the [previous post][previous-post] we analyzed global and local variables.\
Now is the turn of function arguments.\
We will use this simple C program:
```c
int a = 0;

int my_func(int x);

void core0_main(void)
{
    
    int b = my_func(a);
    a = b;
    
    while(1)
    {
    }
}

int my_func(int x) {
   int y = x+1;
   return y;
}
```
This time we will focus on the argument `int x` of `my_func`.\
Its DWARF Debugging Information Entry looks like this:
```
< 2><0x00000395>      DW_TAG_formal_parameter
                        DW_AT_name                  x
                        DW_AT_decl_file             0x00000001 C:/Users/...
                        DW_AT_decl_line             0x0000002e
                        DW_AT_decl_column           0x00000011
                        DW_AT_type                  <0x0000012b>
                        DW_AT_location              len 0x0002: 0x8e74: 
                            DW_OP_breg30-12
``` 
We already know that the `DW_AT_type 0x0000012b` tells us that the type of `x` is `int` (see [this post][previous-post]), and that `DW_OP_breg30-12` means the value of `x` is stored at the address contained in register 30 (which corresponds to TriCore address register `a[14]`) minus an offset of 12.\
We also know that it makes only sense to read `x` when the function `my_func` is executed.\
It seems function arguments are no different from any other local variable when it goes to their organization in memory, and actually this is exactly what happens. The only difference is that DWARF informs us about them being arguments by classifying them as DW_TAG_format_parameter.
## Passing arguments
It is interesting to look at the generated assembly to see how `x` is passed to `my_func` by the caller `core0_main`. The initialization of `x` happens in this very significant single instruction:
```assembly
80000a42:	59 e4 f4 ff 	st.w [%a14]-12,%d4
```
Knowing that the address of `x` is `[%a14]-12`, we see that its value is initialized with the content of the data register `d[4]`.\
Looking at the [ TriCore TC1.6.2 core architecture manual Volume 1](https://www.infineon.com/dgdl/Infineon-AURIX_TC3xx_Architecture_vol1-UserManual-v01_00-EN.pdf?fileId=5546d46276fb756a01771bc4c2e33bdd), chapter 3.1, we see `d[4]` belongs to the so called "Lover Context". This is a set of registers which are NOT saved in the Context Save Area automatically when a function call is performed. This means their values may be retained between function calls, so they can be used to pass values from the caller to the callee and back. This is also what happens in our example: `my_func` knows its caller put in `d[4]` the value of its only argument, and takes it from there. If we look at the assembly code of `core0_main` executed right before the call to `my_func` we see:
```assembly
80000a22:	02 f4       	mov %d4,%d15
80000a24:	6d 00 0d 00 	call 80000a3e <my_func>
```
Here the value of `d[15]` is placed in `d[4]` and then `my_func` is called. `d[15]` contains the value of the variable `a`, which, if you look at the C code, is passed as argument to `my_func`.

[previous-post]: {% link debugger/_posts/2022-12-21-understanding-dwarf-1.markdown%} 
