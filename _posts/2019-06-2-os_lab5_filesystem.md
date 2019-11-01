---
layout:     post
title:      "操作系统_lab5_文件系统 "
subtitle:   " \"liunx 文件系统的实现\""
date:       2019-06-2 23:00:00
author:     "jack"
header-img: "img/post-bg-infinity.jpg"
catalog: true
tags:
    - 大学课程
    - 操作系统
---

## 操作系统实验五 文件系统  

> “16281052 杨涵晨 计科1601 ”

github 地址 https://github.com/jackyanghc/2019_BJTU_OS_16281052/tree/master/lab5

### 实验要求

  本实验要求在模拟的I/O系统之上开发一个简单的文件系统。

  用户通过create, open, read等命令与文件系统交互。

  文件系统把磁盘视为顺序编号的逻辑块序列，逻辑块的编号为0至*L* *−* 1。I/O系统利用内存中的数组模拟磁盘。

### TASK 1  实验概要设计

#### 1.1 实验理解

首先，进行文件系统的定义时，我们首先需要对存储文件的磁盘进行划分，主要分为两大部分：

1. 数据区： 存放文件的具体数据
2. 保留区： 用于实现位图区和文件描述区

我们使用文件记录区，存储当前文件的名字是用户可见的，可以利用其得到我们文件的位图0-1矩阵。

Inode table 主要存储文件的地址存放信息，可以对应到数据区中进行存储和读取。

这里必须明确，文件记录区和inode table也是磁盘的一部分

数据区也就是我们的磁盘最主要的部分，用于存储文件数据。

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190604224607.png)

#### 1.2 实现内容

主要有以下几点：

1. 三个虚拟分区对应的数据结构
2. 设计每个文件的数据结构
3. 文件存储和磁盘的存储关系函数
4. 对应的create函数等
5. 文件打开记录表的实现
6. 用户实现界面

### TASK 2 文件数据结构设计

首先是对block进行定义，对于磁盘主要定义用来存储数据的`data数组`，模拟扇区的512个字节的数据区

```c
typedef struct MyBlock
{
	char data[512];   //对应扇区的字节
	char flag;        //是否存储
	MyBlock* next;
}myblock;
```

对文件的基础信息进行定义，主要包含4项，其中`content`代表正文存储的块地址。

```c
typedef struct File         //文件结构体
{
	char* name;            //文件命名
	char* time;            //创建时间
	char* changeTime;      //修改时间
	myblock* content;
}file;
```

维持一个打开文件表

```c
typedef struct OpenFile
{
	int index;
	int pos;
	myblock* data;
}openFile;
```

### TASK 3 用户交互窗口设计

这部分主要对用户使用时的交互进行编写，根据用户输入的指令进行执行对于的命令，比较简单，采用if - else 进行判断，结果展示如下：

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190607201155.png)

### TASK 4 文件创建和删除

#### 4.1 位图的设计

在创建文件系统时，位图是十分重要的一环，为整个文件系统进行索引。

在设计时，我将位图信息存储在前2块磁盘上，利用“0“，”1“来区分。

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190607204159.png)

#### 4.2 文件的创建

主要实现在磁盘下进行文件存储区域的申请，编写思路如下：

1. 查看文件名是否存在非法字符
2. 查看位图，查看是否重名
3. 查看位图，寻找对应空的位置块
4. 申请磁盘空间，存储文件的data

对应的流程图，如下所示：

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/未命名文件.png)

```c
int create(char* filename)// 根据指定的文件名创建新文件。
{
	file newfile;
	newfile.name = filename;

	//命名是否含有非法字符
	for (int i = 0; i < 8; i++)
	{
		if (strchr(filename, error[i][0]))
		{
			cout << "error 命名含有非法字符，创建失败" << endl;
			break;
		}
	}
	int temp = 1;
	int pos = 0;
	// 是否命名重复
	for (int i = 0; i < B_MAX; i++)
	{
		if (ldisk[0][0][i].flag == '1')
		{
			if (strcmp(ldisk[0][0][i].data,filename)==0)
			{
				temp = 0;
				break;
			}
		}
		else {
			pos = i;
		}
	}
	//申请空间
	if (temp == 0)
	{
		cout << "error 命名重复，创建失败" << endl;
	}
	else {
		ldisk[0][0][pos].flag = '1';
		strcat(ldisk[0][0][pos].data, filename);
		for (int m = 1; m < C_MAX; m++)
		{
			for (int n = 1; n < H_MAX; n++)
			{
				for (int p =0 ; p < B_MAX; p++)
				{
					if (ldisk[m][n][p].flag != '1')
					{
						ldisk[m][n][p].flag = '1';
						ldisk[0][0][pos].next = &ldisk[m][n][p];
						break;
					}
				}
			}
		}
		cout << "ok  创建成功" << endl;
	}
	return 1;
}
```

结果展示：当出现

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190607200811.png)

#### 4.3 文件的删除

相对于文件的创建，这部分更为简单，直接将文件存储的磁盘进行清空。

```c
int destroy(char* filename)// 删除指定文件。
{
	int temp = 1;
	int pos = 0;
	// 找到文件
	for (int i = 0; i < B_MAX; i++)
	{
		if (ldisk[0][0][i].flag == '1')
		{
			if (strcmp(ldisk[0][0][i].data, filename)== 0)
			{
				pos = i;
				break;
			}
		}
	}
	if (pos == 0)
	{
		cout << "error 未找到文件，无法删除" << endl;
	}
	else {
		ldisk[0][0][pos].flag = '0';
		ldisk[0][0][pos].data[0] = '\0';
		ldisk[0][0][pos].next = NULL;
		cout << "ok  删除成功" << endl;
	}
	return 0;
}
```

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190607201653.png)

#### 4.4 文件的重命名

针对存储文件名的数据块进行查找和修改，在进行重命名时也需要进行对应的是否命名冲突查找

```c
int rename(char *old, char *new1)
{
	int temp = 1;
	int pos = 0;
	// 找到文件
	for (int i = 0; i < B_MAX; i++)
	{
		if (ldisk[0][0][i].flag == '1')
		{
			if (strcmp(ldisk[0][0][i].data, old)==0)
			{
				pos = i;
				break;
			}
		}
	}
	if (pos == 0)
	{
		cout << "未找到文件" << endl;
	}
	else {
		strcat(ldisk[0][0][pos].data, new1);
		cout << "文件改名成功" << endl;
	}
	return 0;
}
```

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190607205118.png)

#### 4.4 文件的目录

编写directory函数通过位图进行文件的查找，将目录下的文件打印出来。

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190614210458.png)

### TASK 5 文件打开和关闭

#### 5.1 文件的打开

通过维护一个打开文件表，将打开文件的指针位置，index，块号指向存储进去。

```c
int open(char* filename)
// 打开文件。该函数返回的索引号可用于后续的read, write, lseek, 或close操作。
{
	int pos = 0;
	for (int i = 0; i < B_MAX; i++)
	{
		if (ldisk[0][0][i].flag == '1')
		{
			if (strcmp(ldisk[0][0][i].data, filename)==0)
			{
				pos = i;
				break;
			}
		}
	}
	if (pos == 0)
	{
		cout << "error 未找到文件，无法打开" << endl;
	}
	else {
		cout << "ok  打开成功" << endl;

		for (int j = 0; j < C_MAX; j++)
		{
			if (item[j].index == 0)
			{
				item[j].index = 19706500 + j;
				item[j].pos = 0;
				item[j].data = ldisk[0][0][pos].next;
				cout << "index: " << item[j].index << endl;
				return item[j].index;
			}

		}
	}
	return 0;
}
```

#### 5.2  文件的关闭

根据index，将维护的文件表项进行删除，清空即可

```c
int close(int index)// 关闭制定文件。
{
	for (int j = 0; j < C_MAX; j++)
	{
		if (item[j].index == index)
		{
			item[j].index == 0;
			item[j].pos = 0;
			item[j].data = NULL;
		}

	}
	cout << "关闭成功" << endl;
	return 0;
}
```

使用结果：

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190607210837.png)

### TASK6 文件的写入和读出

#### 6.1 文件指针的维护

每次再打开一个文件时，会通过文件表象进行文件指针的维护，文件指针指向默认对应文件数据的存储开始

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190614201020.png)

利用index和lseek函数进行查找，可以将其指向位置进行改变。

```c
int lseek(int index, int pos)
{
	for (int j = 0; j < C_MAX; j++)
	{
		if (item[j].index == index)
		{
			item[j].pos = pos;
			return 0;
		}
	}
	cout << "找不到文件,请重试" << endl;
	return 0;
}
```

#### 6.2 文件的写入

通过外部先维持一个较大的buff，进行数据的缓存，将要读入数据进行存放，等待I/O结束后将buff中数据写入

需要考虑如下几点：

1. 考虑buffer是否空
2. 考虑读多少次进行回写
3. 考虑磁盘不够进行申请的情况

主要流程图如下所示：

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/未命名文件 (3).png)

```c
int write(int index, string mem_area, int count)
// 把memarea指定的内存位置开始的count个字节顺序写入指定文件。写操作从文件的读写指针指示的位置开始。
{
	for (int j = 0; j < C_MAX; j++)
	{
		if (item[j].index == index)
		{
			myblock* p = item[j].data;
			for (int i = 0; i < count; i++)
			{
				if (i+1 % 512 == 0)
				{
					p->next = new myblock;
					p = p->next;
				}
                if(i % 10 == 0 )
                {
                    p->data[item[j].pos] = mem_area[i];
				    item[j].pos++;
                }
			}
			return 0;
		}
	}
	cout << "找不到文件,请重试" << endl;
	return 0;
}
```

当磁盘块满时，进行空间申请，并且通过拉链的方式将数据连接起来

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190614205913.png)

#### 6.3 文件的读出

跟文件的写入将对应将搜索文件指针指向的磁盘，进行搜索count个字符

如果磁盘被读取完毕，则将其指向指针指向的盘块

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/未命名文件 (4).png)

```c
int read(int index, string mem_area, int count)
// 从指定文件顺序读入count个字节memarea指定的内存位置。读操作从文件的读写指针指示的位置开始。
{
	for (int j = 0; j < C_MAX; j++)
	{
		if (item[j].index == index)
		{
            myblock* p = item[j].data;
			for (int i = 0; i < count; i++)
			{
                if (i+1 % 512 == 0)
				{
					p->next = new myblock;
					p = p->next;
				}
                if(i % 10 == 0 )
                {
                    mem_area[i] = p->data[i + item[j].pos];
                }
			   cout << mem_area;
			}
			cout << "读出成功" << endl;
			return 0;
		}
	}
	cout << "找不到文件,请重试" << endl;
	return 0;
}
```

#### 6.4 文件读取结果

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190614212536.png)

### 实验总结

通过本次文件系统的实验，我更加清楚的认识到**文件是如何在磁盘中存储的**，

通过对位图，文件描述区的定义以及数据结构的编写，熟悉了真实操作系统中的数据存储逻辑

临近考试周，虽然本次实验花费了我很多时间，但是也很有收获，尤其是对文件的读写，对磁盘的操作等等等等

最后，感谢助教和老师的一学期的帮助和指导。

