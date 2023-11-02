# 编译原理课程设计报告
## 一、课程设计要求
    对PL/0作以下修改扩充：
    （1）	必做内容
    增加条件语句的ELSE子句（实现语法语义目标代码）
    （2）选做内容1
    运算符 *=，/=

## 二、实验环境
    实验程序源语言：PL/0语言
    目标语言：类P-code代码
    实现工具：Borland C++ Buider
    操作系统：WIN10
## 三、结构设计说明
### 1. PL/0编译程序
PL/0语言是PASCAL语言的子集，其编译过程采用一趟扫描方式，以语法、语义分析程序为核心。
\
词法分析程序和代码生成程序都作为一个过程，当语法分析需要读单词时就调用词法分析程序，而当语法、语义分析正确，需要生成相应的目标代码时，则调用代码生成程序。
\
表格管理程序实现变量，常量和过程标识符的信息的登录与查找。
\
出错处理程序，对词法和语法、语义分析遇到的错误给出在源程序中出错的位置和与错误性质有关的编号，并进行错误恢复。
![编译程序结构设计](/pict/编译程序结构设计.png "编译程序结构设计")

![编译程序总体流程](/pict/编译程序总体流程.png "编译程序总体流程")

![语法调用关系图](/pict/语法调用关系图.png "语法调用关系图")

### 2. 功能模块描述
|      过程名或函数名      |      功能说明      |
|:-----------------:|:--------------|
|       TERM	       |       项处理      |
|      Error	       |      出错处理      |
|      GetCh	       |    忽略空格读取字符    |
|      GetSym	      |      词法分析      |
|       GEN	        | 生成目标代码，送入目标程序区 |
|       TEST	       |  测试当前单词符号是否合法  |
|      ENTER	       |     登陆名字表      |
|     POSITION	     | 查找标识符在名字表中的位置  |
| ConstDeclaration	 |      常量定义      |
|  VarDeclaration	  |      变量说明      |
|     ListCode	     |     目标代码清单     |
|      FACTOR	      |      因子处理      |
|    EXPRESSION	    |     表达式处理      |
|    CONDITION	     |      条件处理      |
|    STATEMENT	     |      语句处理      |
|      Block	       |   分程序分析处理过程    |
|       BASE	       | 通过静态链求出数据区的基地址 |

## 四、主要成分描述
### 1. 符号表
符号表中存储以下数据：定义的变量、定义的常量、定义的过程；

定义的变量需要存储：变量的标识符，变量定义所在层次，相对于该层次基地址的偏移量（对于基地址将在后面活动记录中详细说明）

定义的常量需要存储：常量的标识符，常量的值，定义所在层次

定义的过程需要存储：过程名，过程处理语句的开始地址（处理语句不是说明语句，说明语句中涉及到符号表的操作，而处理语句中涉及到产生目标代码的操作），过程定义所在层次；

### 2. 运行时存储组织和管理
对于源程序的每一个过程（包括主程序），在被调用时，首先在数据段中开辟三个空间，存放静态链SL、动态链DL和返回地址RA。静态链记录了定义该过程的直接外过程（或主程序）运行时最新数据段的基地址。动态链记录调用该过程前正在运行的过程的数据段基址。返回地址记录了调用该过程时程序运行的断点位置。对于主程序来说，SL、DL和RA的值均置为0。静态链的功能是在一个子过程要引用它的直接或间接父过程（这里的父过程是按定义过程时的嵌套情况来定的，而不是按执行时的调用顺序定的）的变量时，可以通过静态链，跳过个数为层差的数据段，找到包含要引用的变量所在的数据段基址，然后通过偏移地址访问它。

在过程返回时，解释程序通过返回地址恢复指令指针的值到调用前的地址，通过当前段基址恢复数据段分配指针，通过动态链恢复局部段基址指针。实现子过程的返回。对于主程序来说，解释程序会遇到返回地址为0的情况，这时就认为程序运行结束。

解释程序过程中的base函数的功能，就是用于沿着静态链，向前查找相差指定层数的局部数据段基址。这在使用sto、lod等访问局部变量的指令中会经常用到。

类PCODE代码解释执行的部分通过循环和简单的case判断不同的指令，做出相应的动作。当遇到主程序中的返回指令时，指令指针会指到0位置，把这样一个条件作为终至循环的条件，保证程序运行可以正常的结束。

### 3. 语法分析方法
语法分析子程序采用了自顶向下的递归子程序法，语法分析同时也根据程序的语意生成相应的代码，并提供了出错处理的机制。语法分析主要由分程序分析过程（block）、常量定义分析过程（constdeclaration）、变量定义分析过程（vardeclaration）、语句分析过程（statement）、表达式处理过程（expression）、项处理过程（term）、因子处理过程（factor）和条件处理过程（condition）构成。这些过程在结构上构成一个嵌套的层次结构。除此之外，还有出错报告过程（error）、代码生成过程（gen）、测试单词合法性及出错恢复过程（test）、登录名字表过程（enter）、查询名字表函数（position）以及列出类PCODE代码过程（listcode）作过语法分析的辅助过程。

由PL/0的语法图可知：一个完整的PL/0程序是由分程序和句号构成的。因此，本编译程序在运行的时候，通过主程序中调用分程序处理过程block来分析分程序部分（分程序分析过程中还可能会递归调用block过程），然后，判断最后读入的符号是否为句号。如果是句号且分程序分析中未出错，则是一个合法的PL/0程序，可以运行生成的代码，否则就说明源PL/0程序是不合法的，输出出错提示即可。

### 4. 中间代码表示
![中间代码表示](/pict/中间代码表示.png "中间代码表示")

## 五、题目分析与设计
### 1. 增加条件语句的ELSE子语句
要求将不等号# 改为 !=。

（1）相关文法规则

```G(S): S→if S else S | if S | a```

（2）语法描述图
![语法描述图](/pict/语法描述图.png "语法描述图")

（3）语义规则的实现

将`STATEMENT()`函数中的：


```cpp
case IDENT:
    i=POSITION(ID,TX);
    if (i==0) Error(11);
    else
    if (TABLE[i].KIND!=VARIABLE) { /*ASSIGNMENT TO NON-VARIABLE*/
        Error(12); i=0;
    }
    GetSym();
    if (SYM==BECOMES) GetSym();
    else Error(13);
    EXPRESSION(FSYS,LEV,TX);
    if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
    break;
```
修改为：
```cpp
case IDENT:
    i=POSITION(ID,TX);
    if (i==0) Error(11);
    else
    if (TABLE[i].KIND!=VARIABLE) { /*ASSIGNMENT TO NON-VARIABLE*/
        Error(12); i=0;
    }
    GetSym();
    // if (SYM==BECOMES) GetSym();
    // else Error(13);
    // EXPRESSION(FSYS,LEV,TX);
    // if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
    
    if (SYM==BECOMES) {
        GetSym();
        EXPRESSION(FSYS,LEV,TX);
        if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
    }
    else if(SYM==TIMESBECOMES) {    // *= 逻辑运算
        GEN(LOD,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
        GetSym();
        EXPRESSION(FSYS,LEV,TX);
        GEN(OPR,0,4);
        if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
    }
    else if(SYM==SLASHBECOMES) {    // /= 逻辑运算
        GEN(LOD,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
        GetSym();
        EXPRESSION(FSYS,LEV,TX);
        GEN(OPR,0,5);
        if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
    }
    else Error(13);
    
    break;
```


### 2. 扩充赋值运算
（1）运算符` *=，/=`

（2）在`GetSym()`函数增加如下内容：

```cpp
else if(CH=='*') {
        GetCh();
        if(CH=='=') {           // 运算符 '*='
            SYM=TIMESBECOMES; GetCh();
        }
        else SYM=TIMES;
    } else if(CH=='/') {
        GetCh();
        if(CH=='=') {           // 运算符 '/='
            SYM=SLASHBECOMES; GetCh();
        }
```

（3）添加`*=，/=`的逻辑运算，将`STATEMENT()`函数中的：
```cpp
case IDENT:
    i=POSITION(ID,TX);
    if (i==0) Error(11);
    else
    if (TABLE[i].KIND!=VARIABLE) { /*ASSIGNMENT TO NON-VARIABLE*/
        Error(12); i=0;
    }
    GetSym();
    if (SYM==BECOMES) GetSym();
    else Error(13);
    EXPRESSION(FSYS,LEV,TX);
    if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
    break;
```

修改为：
```cpp
case IDENT:
    i=POSITION(ID,TX);
    if (i==0) Error(11);
    else
    if (TABLE[i].KIND!=VARIABLE) { /*ASSIGNMENT TO NON-VARIABLE*/
        Error(12); i=0;
    }
    GetSym();
    // if (SYM==BECOMES) GetSym();
    // else Error(13);
    // EXPRESSION(FSYS,LEV,TX);
    // if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
    
    if (SYM==BECOMES) {
        GetSym();
        EXPRESSION(FSYS,LEV,TX);
        if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
    }
    else if(SYM==TIMESBECOMES) {    // *= 逻辑运算
        GEN(LOD,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
        GetSym();
        EXPRESSION(FSYS,LEV,TX);
        GEN(OPR,0,4);
        if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
    }
    else if(SYM==SLASHBECOMES) {    // /= 逻辑运算
        GEN(LOD,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
        GetSym();
        EXPRESSION(FSYS,LEV,TX);
        GEN(OPR,0,5);
        if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
    }
    else Error(13);
    
    break;
```

## 六、测试用例设计
### 1. 修改单词
PL0源码（!=）：

    PROGRAM EX01;
    VAR A,B,C;
    BEGIN
      B:=8;
      C:=2;
      IF B>5 THEN
        WRITE(B)
      ELSE
        WRITE(C);
      IF B<5 THEN
        WRITE(B)
      ELSE
        WRITE(C);
    END.
生成的COD文件：
    
    === COMPILE PL0 ===
      0 PROGRAM EX01; 
      0 VAR A,B,C; 
      1 BEGIN 
      2   B:=8; 
      4   C:=2; 
      6   IF B>5 THEN 
      9     WRITE(B) 
     12   ELSE 
     13     WRITE(C); 
     17   IF B<5 THEN 
     20     WRITE(B) 
     23   ELSE 
     24     WRITE(C); 
     28 END. 
      0  JMP   0   1
      1  INI   0   6
      2  LIT   0   8
      3  STO   0   4
      4  LIT   0   2
      5  STO   0   5
      6  LOD   0   4
      7  LIT   0   5
      8  OPR   0  12
      9  JPC   0  14
     10  LOD   0   4
     11  OPR   0  14
     12  OPR   0  15
     13  JMP   0  17
     14  LOD   0   5
     15  OPR   0  14
     16  OPR   0  15
     17  LOD   0   4
     18  LIT   0   5
     19  OPR   0  10
     20  JPC   0  25
     21  LOD   0   4
     22  OPR   0  14
     23  OPR   0  15
     24  JMP   0  28
     25  LOD   0   5
     26  OPR   0  14
     27  OPR   0  15
     28  OPR   0   0
    ~~~ RUN PL0 ~~~
    8
    2
    ~~~ END PL0 ~~~

### 2.扩充*=，/= 的赋值运算
PL0源码（运算符）：

    PROGRAM E01;
    VAR A,B,C,D;
    BEGIN
      A:=64;
      A/=2;
      WRITE(A);
      B:=5;
      B*=2;
      WRITE(B);
    END.
生成的COD文件：

    === COMPILE PL0 ===
      0 PROGRAM E01; 
      0 VAR A,B,C,D; 
      1 BEGIN 
      2   A:=64; 
      4   A/=2; 
      8   WRITE(A); 
     11   B:=5; 
     13   B*=2; 
     17   WRITE(B); 
     20 END. 
      0  JMP   0   1
      1  INI   0   7
      2  LIT   0  64
      3  STO   0   3
      4  LOD   0   3
      5  LIT   0   2
      6  OPR   0   5
      7  STO   0   3
      8  LOD   0   3
      9  OPR   0  14
     10  OPR   0  15
     11  LIT   0   5
     12  STO   0   4
     13  LOD   0   4
     14  LIT   0   2
     15  OPR   0   4
     16  STO   0   4
     17  LOD   0   4
     18  OPR   0  14
     19  OPR   0  15
     20  OPR   0   0
    ~~~ RUN PL0 ~~~
    32
    10
    ~~~ END PL0 ~~~
## 六、 总结与体会

通过这次实验，我获得了丰富的PL/0相关知识，并成功扩展了其功能，包括增加单词、修改单词。在实验过程中，我遇到了一些问题，但通过向老师请教和与同学交流，我一一解决了这些问题，这对我的学习带来了巨大的益处。我对编译原理这门课程的理解也因此更加深入。深入学习编译原理并理解其原理后，我发现对之前学过的C语言等多门课程有了全新的认识，这种感觉令人着迷。通过本次编译原理实验的经历，我深刻认识到这门课程的重要性，同时也衷心感谢老师对实验相关知识的耐心讲解。