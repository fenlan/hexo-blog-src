---
title: 计算两个大数乘积巧算法
date: 2016-05-25
---
**最近遇到了一个编程上的小问题，编程题目是这样的，计算一个数的阶乘，我当时就在想，这不是很简单吗，然后我随手写了串代码，如下**

<!--more-->

``` bash
double Jie_Cheng(int i)
{
	double the_output_number = 1;
	int j = 1;

	for (j=1;j<=i;j++)    the_output_number *= j;
	return the_output_number;
}
```

**但是有人提醒我这串代码在有些编译器中只能算1-13的阶乘，我就意思到这样的代码会有溢出数据范围，然后自己想了些法子去实现，然后又去网上找别人写的代码，看了一些不太好的代码，代码多又复杂，所以没兴趣看，然后突然找到了如下代码，我认真地看着**

``` bash
const int max = 3000;
int f[max] = {0};
int main ()
{
	int i, j, number;
    scanf("%d",&number);
    
    f[0] = 1;
    for (i=2;i<=n;i++)
    {
    	int left = 0;                   //下面要用的余数
        for (j=0;j<max;j++)
        {
        	int sum = f[j] * i + left;
            //一个数乘以另一个数的每一项
            f[j] = sum % 10;
            left = sum / 10;
        }
    }
}
```

**大概代码就是这些，代码本身不难看懂，只是这个实现的方法是我没有想过的。这个算法设计很巧，所以有必要记录一下**

**代码翻译成计算过程如下，举个乘积例子：1213 * 325，自己动手计算一下，我的老实计算步骤是这样的：**

``` bash
              1213
           *   325
          ------------
              6065
             2426
            3639
          ------------
            394225
```

**那么代码怎么实现这两个数的计算的呢？首先用5 * 1213 然后将最后一位的5落下来，将剩下的 606 加在第二次用 2 * 1213 的结果上，再将6落下来，同样的道理一直到计算完。这是从计算本身的角度去设计代码，简单易懂，也便于设计代码，那么再大的数也可以实现阶乘计算了，所以最后完成的代码如下：**
``` bash
//本程序有待完善
# include <stdio.h>
# include <stdlib.h>

int * JieCheng (int );
/*计算大数阶乘
 *输出为数组（指针）
 *断点为f[i] = 10,使用该函数时可定义一个指针指向该函数返回值，
 *如函数返回值：5370923 10，最后10为断点
 *注：该函数灵活度很差，有待改善
 */
int main ()
{
	int number = 0;
	int * g;

	while (1)
	{
		printf("Please input a integer no more than 1000\n");
		scanf("%d", &number);

		printf("The result is:");
		g = JieCheng(number);

		for (;(*g)!=10;g++)      printf("%d",*g);
		printf("\n");
	}

	system("pause");
	return 0;
}

int * JieCheng (int number)
{
	int f[3000] = {0};       //待完善
	int i, j;

	f[0] = 1;
	for (i=1;i<=number;i++)            //迭代求阶乘，从1到所求数
	{
		int left = 0;

		for (j=0;j<3000;j++)           //数组表示所求阶乘大数的每一分数位，如个位是f[0],十位是f[1]
		{
			int sum = f[j] * i + left;        //具体求法在博客上
			f[j] = sum % 10;
			left = sum / 10;
		}
	}

	for (i=3000-1;;i--)    if (f[i]!=0)   break;
	for (j=0;j<i/2+1;j++)
	{
		int a = 0;
		a = f[j];               //将数组倒置
		f[j] = f[i-j];
		f[i-j] = a;
	}

	f[i+1] = 10;        //设置断点，使用时以此终止输出 
	return f;
}

```