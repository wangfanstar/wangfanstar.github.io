---
title: C语言中利用X Macro编程技巧简化代码架构
tags: [C,语言,架构]
category: 编程
date: 2018-05-06 08:49:59
---

## X Macro 编程技巧

本篇文章参考自知乎 [你所见过的最美的C语言代码](https://www.zhihu.com/question/57089104)，根据自己的项目经验举出应用场景。

这种技巧实际利用的是编译器的功能，通过编译不同的代码来维护功能函数与功能索引的对应关系。采用此方法最有用的地方是之后再进行增，删，改索引和功能函数时，只要在对应的头文件列表一处更改即可，无需搜索多处的引用逐个修改。还是举一个我工程中常用代码设计为例，因为设备不断升级，实现同样功能的模块芯片可能来自不同的厂家，或厂家不同的版本，每个版本的功能函数接口是一样的，内部实现都不一样。比如一个芯片实现的查表功能函数A，对应的接口返回值为int，参数一个是输入表项，另一个是输出结果。三种芯片内部实现各不相同，对外提供统一的接口函数类型。

## 表驱动法实现 

```c
/* funA 提供函数 */
chipA :  int funAchipA(int * input, int * output);
chipB :  int funAchipB(int * input, int * output);
chipC :  int funAchipC(int * input, int * output);

/* 芯片类型 */
enum CHIPTYPE{
    CHIPA,
    CHIPB,
    CHIPC,
    CHIP_BUTT
};

/* 选择跳转表 */
typedef (* int)pf_funA(int * input, int * output);
pf_funA jumptable[CHIP_BUTT] = {
    funAchipA,
    funAchipB,
    funAchipC,
}

/* 使用方法 */
pf_funA pFunA;
enum CHIPTYPE type;
type = getfromuser(); // 从上层获取芯片类型
pFunA = jumptable[type];

/*  pFunA 作为通用的功能函数接口提供给上层实现查表功能，从而屏蔽不同芯片的差异，上层不感知芯片变化 */
```

## 正常顺序代码实现

《代码大全》中推荐工程师第一篇最重要的要读的文章表驱动法即是以上代码。里面用了一个技巧是通过下标和向量的关联关系代替了if,switch的引用，否则正常的代码如下，一但芯片增加到几十种，就会非常冗长。

```C
/* 正常使用调用函数的代码 */
enum CHIPTYPE type;
type = getfromuser(); // 从上层获取芯片类型
if (type == CHIPA)
{
    pFunA = funAchipA;
}
else if(type == CHIPB)
{
    pFunA = funAchipB;
}
else if(type == CHIPC)
{
    pFunA = funAchipC;
}
...
    
/* 使用pFunA 来实现功能函数A */
```

## 表驱动法的限制

使用`pf_funA jumptable[CHIP_BUTT]`规避掉if，switch的判断代码后，还是会有一些问题，因为_jumptable_下标和内容的联系是我们人工写代码时维护的，但这两个值却在两个地方维护，一个在结构体`enum CHIPTYPE`，一个在`pf_funA jumptable[CHIP_BUTT] `，我们在写代码时，必须严格地保证CHIPTYPE的芯片类型的顺序与jumptable的对应下标的顺序一致，才能保证不出错，这两个结构体在文件中相距较远时，成员数量较多时，在增加删除和修改很可能漏删，少加，或顺序不匹配错误，如下所示，增加一个类型为CHIPCC的芯片，但在修改时，如果两个结构体不在同一处，可能会出现以下情况，导致CHIPC和CHIPCC的功能失配。

```c
/* 芯片类型 */
enum CHIPTYPE{
    CHIPA,
    CHIPB,
    CHIPCC,  
    CHIPC, 
    CHIP_BUTT
};

/* 选择跳转表 */
typedef (* int)pf_funA(int * input, int * output);
pf_funA jumptable[CHIP_BUTT] = {
    funAchipA,
    funAchipB,
    funAchipC,
    funAchipCC,
}
```

## 利用X Macro改进的表驱动法

针对以上情形，究其原因，是因为进行修改时我们要改2处地方，而且修改时有顺序的要求，维护代码时就必须花费更多的精力来进行比对和修改。有没有架构能实现每次修改我只改一处地方，而且不用关心我修改代码的顺序要求呢？X Macro模式就实现了这种功能，使用这种结构后，每次修改只需改一次地方，而且不用关心修改的地方顺序，可以插入任一位置。实现架构如下：

```c
/* 定义芯片类型和功能函数，修改时只改此表即可 */
#define CHIP_TABLE\
	ENTRY(CHIPA, funAchipA)\
	ENTRY(CHIPB, funAchipB)\
	ENTRY(CHIPC, funAchipC)\

/* 芯片类型 */
enum CHIPTYPE{
#define ENTRY(a,b) a,
    CHIP_TABLE
#undef ENTRY
    CHIP_BUTT
};

/* 选择跳转表 */
pf_funA jumptable[CHIP_BUTT] = {
#define ENTRY(a,b) b,
    CHIP_TABLE
#undef ENTRY
}

/* 头文件声明 */
#define ENTRY(a,b) static int b(int * input, int * output);
    CHIP_TABLE
#undef ENTRY
```

采用以上结构后，我们发现，如果要增加新的芯片类型CHIPCC，我们只需在头文件中的CHIP_TABLE中加入一行即可，而且没有顺序要求。

```c
/* 定义芯片类型和功能函数，修改时只改此表即可 */
#define CHIP_TABLE\
	ENTRY(CHIPA, funAchipA)\
	ENTRY(CHIPB, funAchipB)\
	ENTRY(CHIPCC, funAchipCC)\
	ENTRY(CHIPC, funAchipC)\
```

