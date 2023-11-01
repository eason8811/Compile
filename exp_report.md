# 编译原理实验报告
## 一、课程实验要求
    对PL/0作以下修改扩充：
    （1）修改单词：不等号# 改为!=，只有！符号为非法单词，同时#成为非法符号。
    （2）增加单词(只实现词法分析部分)：
    保留字 ELSE，FOR，STEP，UNTILL，DO，RETURN
    运算符 *=，/=，&，||
    注释符 //
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
![中间代码表示](\pict\中间代码表示.png "中间代码表示")

## 五、题目分析与设计
### 1. 修改单词
要求将不等号# 改为 !=。

（1）删除源程序`Unit1.cpp`代码中`SSYM['#']=NEQ;`并增加`SSYM['!=']=NEQ;`。

修改前：

```cpp
SSYM['+']=PLUS;      SSYM['-']=MINUS;
SSYM['*']=TIMES;     SSYM['/']=SLASH;
SSYM['(']=LPAREN;    SSYM[')']=RPAREN;
SSYM['=']=EQL;       SSYM[',']=COMMA;
SSYM['.']=PERIOD;    SSYM['#']=NEQ;
SSYM[';']=SEMICOLON;
```
修改后：
```cpp
SSYM['+']=PLUS;      SSYM['-']=MINUS;
SSYM['*']=TIMES;     SSYM['/']=SLASH;
SSYM['(']=LPAREN;    SSYM[')']=RPAREN;
SSYM['=']=EQL;       SSYM[',']=COMMA;
SSYM['.']=PERIOD;    SSYM['!=']=NEQ;
SSYM[';']=SEMICOLON;
```
（2）在源程序`Unit1.cpp`代码`GetSym()`函数中加入关于`!=`的定义，且单独`!`为非法字符。
```cpp
else if(CH=='!') {
        GetCh();
        if (CH=='=') { SYM=NEQ; GetCh(); }
        else SYM=NUL;
    }
```

### 2. 增加单词
增加下列单词，只实现词法分析部分。
\
保留字 ELSE，FOR，STEP，UNTILL，DO，RETURN
\
运算符 *=，/=，&，||
\注释符 //

（1）由于保留字DO已经存在，因此新增5个保留字和4个运算符1个注释符，共计9个单词（注释符号不需要增加单词）。
原保留字个数是13，因为新增加保留字5个； 

修改前：
```cpp
const  AL    =  10;  /* LENGTH OF IDENTIFIERS */
const  NORW  =  14;  /* # OF RESERVED WORDS */
const  TXMAX = 100;  /* LENGTH OF IDENTIFIER TABLE */
const  NMAX  =  14;  /* MAX NUMBER OF DEGITS IN NUMBERS */
const  AMAX  =2047;  /* MAXIMUM ADDRESS */
const  LEVMAX=   3;  /* MAX DEPTH OF BLOCK NESTING */
const  CXMAX = 200;  /* SIZE OF CODE ARRAY */
```

修改后：
```cpp
const  AL    =  10;  /* LENGTH OF IDENTIFIERS */
const  NORW  =  19;  /* # OF RESERVED WORDS */
const  TXMAX = 100;  /* LENGTH OF IDENTIFIER TABLE */
const  NMAX  =  14;  /* MAX NUMBER OF DEGITS IN NUMBERS */
const  AMAX  =2047;  /* MAXIMUM ADDRESS */
const  LEVMAX=   3;  /* MAX DEPTH OF BLOCK NESTING */
const  CXMAX = 200;  /* SIZE OF CODE ARRAY */

const int SYMNUM = 42; //保留字个数
```
原单词总数是33，而新增加单词9个，则应将33都改为42
搜索并一共修改了以下9处函数的相关参数：
```cpp
SYMSET SymSetUnion(SYMSET S1,SYMSET S2)
SYMSET SymSetAdd(SYMBOL SY,SYMSET S)
SYMSET SymSetNew(SYMBOL a)
SYMSET SymSetNew(SYMBOL a,SYMBOL b)
SYMSET SymSetNew(SYMBOL a,SYMBOL b,SYMBOL c)
SYMSET SymSetNew(SYMBOL a,SYMBOL b,SYMBOL c,SYMBOL d)
SYMSET SymSetNew(SYMBOL a,SYMBOL b,SYMBOL c,SYMBOL d,SYMBOL e)
SYMSET SymSetNew(SYMBOL a,SYMBOL b,SYMBOL C,SYMBOL d,SYMBOL e,SYMBOL f)
SYMSET SymSetNULL()
```
故所有`i<33(i实际上是指单词总数)`因为新增加单词10个，故应改为`i<42`。
各个保留字和运算符的对应情况如下：

保留字SYM表示

|  ELSE   |  FOR   |  STEP   |  UNTIL   |  RETURN   |
|:-------:|:------:|:-------:|:--------:|:---------:|
| ELSESYM | FORSYM | STEPSYM | UNTILSYM | RETURNSYM |

运算符SYM表示

| *=     | /=     | &      | \|\|  |
|--------|--------|--------|-------|
| MULSYM | DIVSYM | ANDSYM | ORSYM |

因此首先先在保留字集合中按照格式添加新增的10个单词。

修改前：
```cpp
typedef enum { NUL,     IDENT,    NUMBER,    PLUS,     MINUS,   TIMES,
    SLASH,    ODDSYM,   EQL,       NEQ,      LSS,     LEQ,
    GTR,      GEQ,      LPAREN,    RPAREN,   COMMA,   SEMICOLON,
    PERIOD,   BECOMES,  BEGINSYM,  ENDSYM,  IFSYM,    THENSYM,
    WHILESYM, WRITESYM, READSYM,   DOSYM,   CALLSYM,	CONSTSYM,
    VARSYM,   PROCSYM,  PROGSYM
} SYMBOL;
char *SYMOUT[] = {"NUL", "IDENT", "NUMBER", "PLUS", "MINUS", "TIMES",
                  "SLASH", "ODDSYM", "EQL", "NEQ", "LSS", "LEQ", "GTR", "GEQ",
                  "LPAREN", "RPAREN", "COMMA", "SEMICOLON", "PERIOD",
                  "BECOMES", "BEGINSYM", "ENDSYM", "IFSYM", "THENSYM",
                  "WHILESYM", "WRITESYM", "READSYM", "DOSYM", "CALLSYM",
                  "CONSTSYM", "VARSYM", "PROCSYM", "PROGSYM"
                };
```

修改后：
```cpp
typedef enum { NUL,     IDENT,    NUMBER,    PLUS,     MINUS,   TIMES,
    SLASH,    ODDSYM,   EQL,       NEQ,      LSS,     LEQ,
    GTR,      GEQ,      LPAREN,    RPAREN,   COMMA,   SEMICOLON,
    PERIOD,   BECOMES,  BEGINSYM,  ENDSYM,  IFSYM,    THENSYM,
    WHILESYM, WRITESYM, READSYM,   DOSYM,   CALLSYM,	CONSTSYM,
    VARSYM,   PROCSYM,  PROGSYM,
    ELSESYM,  FORSYM,   STEPSYM, UNTILSYM, RETURNSYM,	// 共5个。ELSE，FOR，STEP，UNTIL，RETURN
    TIMESBECOMES, SLASHBECOMES, ANDSYM, ORSYM	// 共4个。*=，/=，&，||
} SYMBOL;
char *SYMOUT[] = {"NUL", "IDENT", "NUMBER", "PLUS", "MINUS", "TIMES",
                  "SLASH", "ODDSYM", "EQL", "NEQ", "LSS", "LEQ", "GTR", "GEQ",
                  "LPAREN", "RPAREN", "COMMA", "SEMICOLON", "PERIOD",
                  "BECOMES", "BEGINSYM", "ENDSYM", "IFSYM", "THENSYM",
                  "WHILESYM", "WRITESYM", "READSYM", "DOSYM", "CALLSYM",
                  "CONSTSYM", "VARSYM", "PROCSYM", "PROGSYM",
                  "ELSESYM", "FORSYM", "STEPSYM", "UNTILSYM", "RETURNSYM",
                  "TIMESBECOMES", "SLASHBECOMES", "ANDSYM", "ORSYM"};
```

（3）将新增的保留字按照字母表升序的方式添加，运算符参照已有的运算符来进行添加并与SYM一一对应。对于新插入的单词造成的数组顺序乱序，需要根据升序方式对于数组下标进行重新整理排序。

修改前（保留字）：
```cpp
  strcpy(KWORD[ 1],"BEGIN");    strcpy(KWORD[ 2],"CALL");
  strcpy(KWORD[ 3],"CONST");    strcpy(KWORD[ 4],"DO");
  strcpy(KWORD[ 5],"END");      strcpy(KWORD[ 6],"IF");
  strcpy(KWORD[ 7],"ODD");      strcpy(KWORD[ 8],"PROCEDURE");
  strcpy(KWORD[ 9],"PROGRAM");  strcpy(KWORD[10],"READ");
  strcpy(KWORD[11],"THEN");     strcpy(KWORD[12],"VAR");
  strcpy(KWORD[13],"WHILE");    strcpy(KWORD[14],"WRITE");
  
  WSYM[ 1]=BEGINSYM;   WSYM[ 2]=CALLSYM;
  WSYM[ 3]=CONSTSYM;   WSYM[ 4]=DOSYM;
  WSYM[ 5]=ENDSYM;     WSYM[ 6]=IFSYM;
  WSYM[ 7]=ODDSYM;     WSYM[ 8]=PROCSYM;
  WSYM[ 9]=PROGSYM;    WSYM[10]=READSYM;
  WSYM[11]=THENSYM;    WSYM[12]=VARSYM;
  WSYM[13]=WHILESYM;   WSYM[14]=WRITESYM;
```

修改后（保留字）：
```cpp
// 新增5个保留字。ELSE, FOR, STEP, UNTIL, RETURN
    strcpy(KWORD[ 1],"BEGIN");  strcpy(KWORD[ 2],"CALL");
    strcpy(KWORD[ 3],"CONST");  strcpy(KWORD[ 4],"DO");
    strcpy(KWORD[ 5],"ELSE");   strcpy(KWORD[ 6],"END");    // 增加保留字1。 ELSE
    strcpy(KWORD[ 7],"FOR");    strcpy(KWORD[ 8],"IF");     // 增加保留字2。 FOR
    strcpy(KWORD[ 9],"ODD");    strcpy(KWORD[10],"PROCEDURE");
    strcpy(KWORD[11],"PROGRAM");strcpy(KWORD[12],"READ");
    strcpy(KWORD[13],"RETURN"); strcpy(KWORD[14],"STEP");	// 增加保留字5。 RETURN 增加保留字3。 STEP
    strcpy(KWORD[15],"THEN");   strcpy(KWORD[16],"UNTIL");	// 增加保留字4。 UNTIL
    strcpy(KWORD[17],"VAR");    strcpy(KWORD[18],"WHILE");
    strcpy(KWORD[19],"WRITE");

    // 新增5个保留字符号。ELSESYM, FORSYM, STEPSYM, UNTILSYM, RETURNSYM
    WSYM[ 1]=BEGINSYM;  WSYM[ 2]=CALLSYM;
    WSYM[ 3]=CONSTSYM;  WSYM[ 4]=DOSYM;
    WSYM[ 5]=ELSESYM;   WSYM[ 6]=ENDSYM;    // 增加保留字符号1。 ELSESYM
    WSYM[ 7]=FORSYM;    WSYM[ 8]=IFSYM;     // 增加保留字符号2。 FORSYM
    WSYM[ 9]=ODDSYM;    WSYM[10]=PROCSYM;
    WSYM[11]=PROGSYM;   WSYM[12]=READSYM;
    WSYM[13]=RETURNSYM; WSYM[14]=STEPSYM;   // 增加保留字符号5。 RETURNSYM 增加保留字符号3。 STEPSYM
    WSYM[15]=THENSYM;   WSYM[16]=UNTILSYM;  // 增加保留字符号4。 UNTILSYM
    WSYM[17]=VARSYM;    WSYM[18]=WHILESYM;
    WSYM[19]=WRITESYM;
```

对于运算符、注释符：

对`GetSym()`进行修改
\
修改前：
```cpp
void GetSym() {
  int i,J,K;   ALFA  A;
  while (CH<=' ') GetCh();
  if (CH>='A' && CH<='Z') { /*ID OR RESERVED WORD*/
    K=0;
	do {
	  if (K<AL) A[K++]=CH;
	  GetCh();
	}while((CH>='A' && CH<='Z')||(CH>='0' && CH<='9'));
	A[K]='\0';
	strcpy(ID,A); i=1; J=NORW;
	do {
	  K=(i+J) / 2;
	  if (strcmp(ID,KWORD[K])<=0) J=K-1;
	  if (strcmp(ID,KWORD[K])>=0) i=K+1;
	}while(i<=J);
	if (i-1 > J) SYM=WSYM[K];
	else SYM=IDENT;
  }
  else
    if (CH>='0' && CH<='9') { /*NUMBER*/
      K=0; NUM=0; SYM=NUMBER;
	  do {
	    NUM=10*NUM+(CH-'0');
		K++; GetCh();
      }while(CH>='0' && CH<='9');
	  if (K>NMAX) Error(30);
    }
    else
      if (CH==':') {
	    GetCh();
		if (CH=='=') { SYM=BECOMES; GetCh(); }
		else SYM=NUL;
      }
	  else /* THE FOLLOWING TWO CHECK WERE ADDED
	         BECAUSE ASCII DOES NOT HAVE A SINGLE CHARACTER FOR <= OR >= */
	    if (CH=='<') {
		  GetCh();
		  if (CH=='=') { SYM=LEQ; GetCh(); }
		  else SYM=LSS;
		}
		else
		  if (CH=='>') {
		    GetCh();
			if (CH=='=') { SYM=GEQ; GetCh(); }
			else SYM=GTR;
          }
		  else { SYM=SSYM[CH]; GetCh(); }
} /*GetSym()*/
```

修改后：
```cpp
void GetSym() {
    int i,J,K;   ALFA  A;
    while (CH<=' ') GetCh();
    if (CH>='A' && CH<='Z') { /*ID OR RESERVED WORD*/
        K=0;
        do {
            if (K<AL) A[K++]=CH;
            GetCh();
        }while((CH>='A' && CH<='Z')||(CH>='0' && CH<='9'));
        A[K]='\0';
        strcpy(ID,A); i=1; J=NORW;
        do {
            K=(i+J) / 2;
            if (strcmp(ID,KWORD[K])<=0) J=K-1;
            if (strcmp(ID,KWORD[K])>=0) i=K+1;
        }while(i<=J);
        if (i-1 > J) SYM=WSYM[K];
        else SYM=IDENT;
    }
    else
    if (CH>='0' && CH<='9') { /*NUMBER*/
        K=0; NUM=0; SYM=NUMBER;
        do {
            NUM=10*NUM+(CH-'0');
            K++; GetCh();
        }while(CH>='0' && CH<='9');
        if (K>NMAX) Error(30);
    }
    else
    if (CH==':') {
        GetCh();
        if (CH=='=') { SYM=BECOMES; GetCh(); }
        else SYM=NUL;
    }
    else /* THE FOLLOWING TWO CHECK WERE ADDED
             BECAUSE ASCII DOES NOT HAVE A SINGLE CHARACTER FOR <= OR >= */
    if (CH=='<') {
        GetCh();
        if (CH=='=') { SYM=LEQ; GetCh(); }
        else SYM=LSS;
    }
    else
    if (CH=='>') {
        GetCh();
        if (CH=='=') { SYM=GEQ; GetCh(); }
        else SYM=GTR;
    }

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
        else if(CH=='*'){       // '/* */' 多行注释
            GetCh();
            i=CH;
            while (i!='*' || CH!='/') {
                i=CH;
                GetCh();
            }
            if(i!='*' && CH!='/') Error(19);
            else{
                GetCh();
                GetSym();
            }
        }
        else if(CH=='/'){       // '//' 单行注释
            i=CX;
            while (CC!=LL) {
                GetCh();
            }
            GetSym();
        }
        else SYM=SLASH;
    } else if(CH=='&') {        // 运算符 '&'
        GetCh();
        SYM=ANDSYM;
    } else if(CH=='|') {
        GetCh();
        if(CH=='|') {           // 运算符 '||'
            SYM=ORSYM; GetCh();
        }
        else Error(19);
    } else if(CH=='!') {
        GetCh();
        if (CH=='=') { SYM=NEQ; GetCh(); }
        else SYM=NUL;
    }
    else { SYM=SSYM[CH]; GetCh(); }
} /*GetSym()*/
```

（4）在`STATEMENT()`函数中添加所有添加的单词
\
修改前：
```cpp
oid STATEMENT(SYMSET FSYS,int LEV,int &TX) {   /*STATEMENT*/
  int i,CX1,CX2;
  switch (SYM) {
	case IDENT:
		i=POSITION(ID,TX);
		if (i==0) Error(11);
		else
		  if (TABLE[i].KIND!=VARIABLE) { /*ASSIGNMENT TO NON-VARIABLE*/
			Error(12); i=0;
		  }
        GetSym();
		if (SYM==BECOMES){ GetSym();
			EXPRESSION(FSYS,LEV,TX);
			if (i!=0) GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
		}
		else if (SYM==TIMESBK){ //codes for *=

		}
		else if(SYM==SLASHBK){ //codes for /=

		}
		else Error(13);
		break;
	case READSYM:
		GetSym();
		if (SYM!=LPAREN) Error(34);
		else
		  do {
			GetSym();
			if (SYM==IDENT) i=POSITION(ID,TX);
			else i=0;
			if (i==0) Error(35);
			else {
			  GEN(OPR,0,16);
			  GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
			}
			GetSym();
		  }while(SYM==COMMA);
		if (SYM!=RPAREN) {
		  Error(33);
		  while (!SymIn(SYM,FSYS)) GetSym();
		}
		else GetSym();
		break; /* READSYM */
	case WRITESYM:
		GetSym();
		if (SYM==LPAREN) {
		  do {
			GetSym();
			EXPRESSION(SymSetUnion(SymSetNew(RPAREN,COMMA),FSYS),LEV,TX);
			GEN(OPR,0,14);
		  }while(SYM==COMMA);
		  if (SYM!=RPAREN) Error(33);
		  else GetSym();
		}
		GEN(OPR,0,15);
		break; /*WRITESYM*/
	case CALLSYM:
		GetSym();
		if (SYM!=IDENT) Error(14);
		else {
		  i=POSITION(ID,TX);
		  if (i==0) Error(11);
		  else
			if (TABLE[i].KIND==PROCEDUR)
			  GEN(CAL,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
			else Error(15);
		  GetSym();
		}
		break;
	case IFSYM:
		GetSym();
		CONDITION(SymSetUnion(SymSetNew(THENSYM,DOSYM),FSYS),LEV,TX);
		if (SYM==THENSYM) GetSym();
		else Error(16);
		CX1=CX;  GEN(JPC,0,0);
		STATEMENT(FSYS,LEV,TX);  CODE[CX1].A=CX;
		break;
	case BEGINSYM:
		GetSym();
		STATEMENT(SymSetUnion(SymSetNew(SEMICOLON,ENDSYM),FSYS),LEV,TX);
		while (SymIn(SYM, SymSetAdd(SEMICOLON,STATBEGSYS))) {
		  if (SYM==SEMICOLON) GetSym();
		  else Error(10);
		  STATEMENT(SymSetUnion(SymSetNew(SEMICOLON,ENDSYM),FSYS),LEV,TX);
		}
		if (SYM==ENDSYM) GetSym();
		else Error(17);
		break;
	case WHILESYM:
		CX1=CX; GetSym(); CONDITION(SymSetAdd(DOSYM,FSYS),LEV,TX);
		CX2=CX; GEN(JPC,0,0);
		if (SYM==DOSYM) GetSym();
		else Error(18);
		STATEMENT(FSYS,LEV,TX);
		GEN(JMP,0,CX1);
		CODE[CX2].A=CX;
		break;
  }
  TEST(FSYS,SymSetNULL(),19);
} /*STATEMENT*/
```

修改后：
```cpp
void STATEMENT(SYMSET FSYS,int LEV,int &TX) {   /*STATEMENT*/
    int i,CX1,CX2;
    switch (SYM) {
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
        case READSYM:
            GetSym();
            if (SYM!=LPAREN) Error(34);
            else
                do {
                    GetSym();
                    if (SYM==IDENT) i=POSITION(ID,TX);
                    else i=0;
                    if (i==0) Error(35);
                    else {
                        GEN(OPR,0,16);
                        GEN(STO,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
                    }
                    GetSym();
                }while(SYM==COMMA);
            if (SYM!=RPAREN) {
                Error(33);
                while (!SymIn(SYM,FSYS)) GetSym();
            }
            else GetSym();
            break; /* READSYM */
        case WRITESYM:
            GetSym();
            if (SYM==LPAREN) {
                do {
                    GetSym();
                    EXPRESSION(SymSetUnion(SymSetNew(RPAREN,COMMA),FSYS),LEV,TX);
                    GEN(OPR,0,14);
                }while(SYM==COMMA);
                if (SYM!=RPAREN) Error(33);
                else GetSym();
            }
            GEN(OPR,0,15);
            break; /*WRITESYM*/
        case CALLSYM:
            GetSym();
            if (SYM!=IDENT) Error(14);
            else {
                i=POSITION(ID,TX);
                if (i==0) Error(11);
                else
                if (TABLE[i].KIND==PROCEDUR)
                    GEN(CAL,LEV-TABLE[i].vp.LEVEL,TABLE[i].vp.ADR);
                else Error(15);
                GetSym();
            }
            break;
        case IFSYM:
            GetSym();
            CONDITION(SymSetUnion(SymSetNew(THENSYM,DOSYM),FSYS),LEV,TX);
            if (SYM==THENSYM) GetSym();
            else Error(16);
            CX1=CX;  GEN(JPC,0,0);
            // STATEMENT(FSYS,LEV,TX);  CODE[CX1].A=CX; 注释掉

            // ↓↓↓ 新增部分 ↓↓↓
            STATEMENT(SymSetUnion(SymSetNew(ELSESYM),FSYS),LEV,TX);
            if(SYM!=ELSESYM)
                CODE[CX1].A=CX;
            else {
                GetSym();
                CX2=CX;
                GEN(JMP,0,0);		//直接跳转，执行完Then后面的则跳转到条件语句最后面
                CODE[CX1].A=CX;		//回填条件跳转，填回else语句块中第一句
                STATEMENT(FSYS,LEV,TX);
                CODE[CX2].A=CX;  	//回填直接跳转地址
            }
            // ↑↑↑ 新增部分 ↑↑↑

            break;
        case BEGINSYM:
            GetSym();
            STATEMENT(SymSetUnion(SymSetNew(SEMICOLON,ENDSYM),FSYS),LEV,TX);
            while (SymIn(SYM, SymSetAdd(SEMICOLON,STATBEGSYS))) {
                if (SYM==SEMICOLON) GetSym();
                else Error(10);
                STATEMENT(SymSetUnion(SymSetNew(SEMICOLON,ENDSYM),FSYS),LEV,TX);
            }
            if (SYM==ENDSYM) GetSym();
            else Error(17);
            break;
        case WHILESYM:
            CX1=CX; GetSym(); CONDITION(SymSetAdd(DOSYM,FSYS),LEV,TX);
            CX2=CX; GEN(JPC,0,0);
            if (SYM==DOSYM) GetSym();
            else Error(18);
            STATEMENT(FSYS,LEV,TX);
            GEN(JMP,0,CX1);
            CODE[CX2].A=CX;
            break;

// 用来检验保留字是否添加成功的标志
        case FORSYM:
            GetSym();
            Form1->printfs("~~~~ FORSYM~~~~");
            break;
        case STEPSYM:
            GetSym();
            Form1->printfs("~~~~ STEPSYM~~~~");
            break;
        case UNTILSYM:
            GetSym();
            Form1->printfs("~~~~ UNTILSYM~~~~");
            break;
        case RETURNSYM:
            GetSym();
            Form1->printfs("~~~~ RETURNSYM~~~~");
            break;
        case DOSYM:
            GetSym();
            Form1->printfs("~~~~ DOSYM~~~~");
            break;
        case ELSESYM:
            GetSym();
            Form1->printfs("~~~~ ELSESYM~~~~");
            break;

// 用来检验运算符是否添加成功的标志。
        case TIMESBECOMES:
            GetSym();
            Form1->printfs("~~~~ *= ~~~~");
            break;
        case SLASHBECOMES:
            GetSym();
            Form1->printfs("~~~~ /= ~~~~");
            break;
        case ANDSYM:
            GetSym();
            Form1->printfs("~~~~ &  ~~~~");
            break;
        case ORSYM:
            GetSym();
            Form1->printfs("~~~~ || ~~~~");
            break;

    }
    TEST(FSYS,SymSetNULL(),19);
} /*STATEMENT*/
```

## 六、测试用例设计
### 1. 修改单词
PL0源码（!=）：

    PROGRAM E02;
    VAR A;
    BEGIN
      A:=989;
      IF A!=985 THEN
            WRITE(A)
    END.
生成的COD文件：
    
    === COMPILE PL0 ===
      0 PROGRAM E02; 
      0 VAR A; 
      1 BEGIN 
      2   A:=989; 
      4   IF A!=985 THEN 
      7         WRITE(A) 
     10 END. 
      0  JMP   0   1
      1  INI   0   4
      2  LIT   0 989
      3  STO   0   3
      4  LOD   0   3
      5  LIT   0 985
      6  OPR   0   9
      7  JPC   0  11
      8  LOD   0   3
      9  OPR   0  14
     10  OPR   0  15
     11  OPR   0   0
    ~~~ RUN PL0 ~~~
    989
    ~~~ END PL0 ~~~
PL0源码（单独 !）：

    PROGRAM E02;
    VAR A;
    BEGIN
      A:=989;
      IF A!985 THEN
          WRITE(A)
    END.
生成的COD文件：

    === COMPILE PL0 ===
      0 PROGRAM E02; 
      0 VAR A; 
      1 BEGIN 
      2   A:=989; 
      4   IF A!985 THEN 
    ***       ^23
    ***               ^20
      6       WRITE(A) 
      9 END. 
      0  JMP   0   1
      1  INI   0   4
      2  LIT   0 989
      3  STO   0   3
      4  LOD   0   3
      5  LIT   0 985
      6  JPC   0  10
      7  LOD   0   3
      8  OPR   0  14
      9  OPR   0  15
     10  OPR   0   0
    ERROR IN PL/0 PROGRAM
### 2.增加单词
PL0源码（运算符）：

    PROGRAM E03
    VAR A;
    BEGIN
      *=;
      /=;
      &;
      ||;
      //  123456
      /* 666
          777
          888*/
    END.
生成的COD文件：

    === COMPILE PL0 ===
      0 PROGRAM E03 
      0 VAR A; 
    ***   ^5
      1 BEGIN 
      2   *=; 
      2   /=; 
      2   &; 
      2   ||; 
      2   //  123456 
      2   /* 666 
      2       777 
      2       888*/ 
      2 END. 
      0  JMP   0   1
      1  INI   0   4
      2  OPR   0   0
    ERROR IN PL/0 PROGRAM
PL0源码（关键字）：

    PROGRAM E04;
    VAR A,B,C;
    BEGIN
      ELSE;
      FOR;
      DO;
      UNTIL;
      RETURN;
    END.
生成的COD文件：

    === COMPILE PL0 ===
      0 PROGRAM E04; 
      0 VAR A,B,C; 
      1 BEGIN 
      2   ELSE; 
      2   FOR; 
      2   DO; 
      2   UNTIL; 
      2   RETURN; 
      2 END. 
      0  JMP   0   1
      1  INI   0   6
      2  OPR   0   0
    ~~~ RUN PL0 ~~~
    ~~~ END PL0 ~~~
## 六、 总结与体会

通过这次实验，我获得了丰富的PL/0相关知识，并成功扩展了其功能，包括增加单词、修改单词。在实验过程中，我遇到了一些问题，但通过向老师请教和与同学交流，我一一解决了这些问题，这对我的学习带来了巨大的益处。我对编译原理这门课程的理解也因此更加深入。深入学习编译原理并理解其原理后，我发现对之前学过的C语言等多门课程有了全新的认识，这种感觉令人着迷。通过本次编译原理实验的经历，我深刻认识到这门课程的重要性，同时也衷心感谢老师对实验相关知识的耐心讲解。