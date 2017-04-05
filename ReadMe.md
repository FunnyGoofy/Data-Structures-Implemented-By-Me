# 常用数据结构与算法的实现

[TOC]

## KMP字符串匹配算法

有时候我们需要在一大段字符串中得到找到我们想要的位置，这个时候KMP字符串匹配算法是一种比较好的选择。

首先我们需要进行**匹配字符串的预处理**，即我们要在一个字符串中找另外一个字符串，我们需要预处理这个字符串。主要是查看这个字符串上有没有“最长公共子序列”。最长公共子序列是是KMP算法的核心概念。这决定了我们一位位向前移动匹配的时候，如果遇到匹配失败的我们应该如何处理，是整个匹配字符串向前挪动一位吗，其实不是挪动一位，只要我们可以知道最长公共子序列，那么我们就可以直接在比较发现不匹配之后，移动一个更大的距离，而不是一格，继续比较。而我们所说的那个“更大的距离”，就是由“最长公共子序列”决定的，我们需要知道，在已经匹配到的子字符串两头最大的公共部分是多少，下次比较的时候我们直接在匹配失败的时候把最大公共部分的前面那部分平移过来，前面那一部分最长公共子序列的下一位开始比较。

预处理还是很简单的：

```c++
//我们首先先进行预处理，需要delete这个堆区数组
    int *pre_handle_arr = pre_handle(patch_string);
    
    
    //然后进行比较
    int compare_continue = -1;
    
    int patch_string_index = 0;
    int goal_string_index = 0;
    
    //退出这个循环主要是两种情况，首先就是目标字符串走完了，还有就是要匹配别人的小字符串走完了，前者代表失败，后者代表成功
    while(patch_string_index < patch_string.length() && goal_string_index < goal_string.length()){
        
        if(patch_string[patch_string_index] == goal_string[goal_string_index]){
            compare_continue = pre_handle_arr[patch_string_index];
            patch_string_index++;
            goal_string_index++;
        }else{
            //如果没有匹配成功，并且这个时候compare_continue是-1，说明在已匹配子串中没有最大公共子序列
            //那么我们直接匹配目标字符串的下一位
            if(compare_continue == -1){
                goal_string_index++;
                patch_string_index = 0;
            }else{
                //如果在某一位匹配失败，并且有公共子序列，那么就利用公共子序列，将公共子序列的前半部分向前挪到后半部分的位置
                patch_string_index = compare_continue + 1;
                
                compare_continue = pre_handle_arr[compare_continue];
//                cout << patch_string_index << "," << compare_continue << endl;
            }
        }
        
//        cout << patch_string_index << "," << goal_string_index << endl;
    }

```

道理很简单，除了第一位，每一位的字符都要和tmp+1这个位置的字符比对一下。如果相等，那么就可以说明最大公共子字符串又扩充了一位。

在逻辑上比较繁琐的就是进行字符串比较的那个过程，有点绕：

```c++
//我们首先先进行预处理，需要delete这个堆区数组
    int *pre_handle_arr = pre_handle(patch_string);
    
    
    //然后进行比较
    int compare_continue = -1;
    
    int patch_string_index = 0;
    int goal_string_index = 0;
    
    //退出这个循环主要是两种情况，首先就是目标字符串走完了，还有就是要匹配别人的小字符串走完了，前者代表失败，后者代表成功
    while(patch_string_index < patch_string.length() && goal_string_index < goal_string.length()){
        
        if(patch_string[patch_string_index] == goal_string[goal_string_index]){
            compare_continue = pre_handle_arr[patch_string_index];
            patch_string_index++;
            goal_string_index++;
        }else{
            //如果没有匹配成功，并且这个时候compare_continue是-1，说明在已匹配子串中没有最大公共子序列
            //那么我们直接匹配目标字符串的下一位
            if(compare_continue == -1){
                goal_string_index++;
                patch_string_index = 1;
            }else{
                //如果在某一位匹配失败，并且有公共子序列，那么就利用公共子序列，将公共子序列的前半部分向前挪到后半部分的位置
                patch_string_index = compare_continue + 1;
                
                compare_continue = pre_handle_arr[compare_continue];
                //cout << patch_string_index << "," << compare_continue << endl;
            }
        }
        
//        cout << patch_string_index << "," << goal_string_index << endl;
    }

```

首先我们要注意循环的条件，必须两个string都没有遍历完才可以进行循环，两个之中有其中一个遍历完了就不可以循环了。

我们主要使用patch_string_index和goal_string_index两个索引分别遍历匹配小字符串和目标字符串。如果相等，当然是这两个索引都会自增，然后比较目标字符串和匹配小字符串的下一位。如果发现不匹配了，首先就要看看以匹配的子串里面有没有最大公共子字符串（使用compare_continue==-1）。如果没有，那么直接使用匹配字符串的第0位去匹配，目标字符串的下一位就好了（patch_string_index=0赋值，goal_string_index自增）。如果发现有最大公共子字符串那就让最大公共子字符串的后一位去匹配目标字符串的当前位置。所以对于比较的结果，一共是三类情况。



## 环状队列

因为队列只能从尾部加入，从顶部移出，所以我们使用传统数组的方式会有问题，因为随着头部内容的不断移出以及尾部的不断移进，整个队列会逐渐往线性表的后部靠拢，这样子在线性表的前半部分就会有很大的空间浪费。如果要把已经贴近后半部分的内容挪到靠近线性表头部的位置，这就需要大量的开销，所以我们就需要换装队列。

环状队列本质上还是线性表，通过取模的操作获得环状的效果。

环状队列比较什么时候满的时候我们，除了可以维护一个存储了环状队列当前容量的变量之外还可以预留一个空位置。为了统一我们的逻辑，我们必须设定一套在所有的时期都适用的判定环状队列空或者满的办法。我们设定头指针与尾指针重合代表空，尾索引取模如果比头索引取模之后小1，或者比头索引大maxsize-1那么就说明队列已经满了。rear永远指向没有值的下一个要插入的位置，头索引永远指向下一个要推出的值，除非队列是空的。

这次实现中使用了模板类，因为模板类不是类，所以模板类是不能拆的，应该放在一个头文件中，要不就会报错。此外我们就是要注意取模和边界条件。这样子就可以了。总体的实现还是比较简单的。

## 走迷宫算法

走迷宫算法是一个对栈结构的非常不错的利用，我们我们通过一个栈来获得我们已经走过的路径，如果栈顶的节点在所有方向上都走不通，那就进行弹栈操作。在倒数第二个节点上进行各个方向的尝试。任何曾经进过栈的节点都会被记录下来，防止走重复的路。

这里在编程里面碰到了一些问题，比如在C++11中，我们发现枚举和结构体实例化的时候我们不再需要在前面再加struct和enum这两个关键词。直接当做一个类使用就好了。

此外我们发现int a\[\]\[\]，实际上a变量这个时候并不是一个二维指针，而是一个数组对象，所以说我们没有办法把他当做一个二维指针使用。其实使用栈空间来申请二维数组，获取其二维指针的方式并不是没有，但是比较困难，并且语法比较容易混淆，所以我们使用new的方式产生线性数组，以及指针数组来产生二维数组。

在if和while条件上我也出现了不止一次的错误，注意非（！）与或等运算符组合起来的关系。

还要注意的是，枚举是一种常量，不能进行自增操作。

## 中缀表达式转后缀表达式

这也是一个使用栈的经典案例，我们使用栈来保存符号，而变量字符会被直接输出。符号一共有6种，加、减、左右小括号，乘除。而这个“符号栈”的入栈和出栈规则如下：

- 符号是有优先级的，正常情况下，符号会一位一位进入栈中，如果碰到了一个运算符，不比栈顶运算符的优先级小，那么这个运算符就要正常入栈。
- 出栈的策略非常简单，如果整个表达式走完了就把所有的符号弹出并且输出，如果表达式没有走完，但是有一个比栈顶运算符优先级小的运算符要入栈，那就先要将栈中的所有元素推出。
- 括号的处理也很重要，一般情况下，左括号我们直接入栈就好，遇到右括号，那么我们就需要我们将右括号与左括号之间的弹栈就好。左括号了右括号不会出现在后缀表达式中。




## 环装链表

环装链表。环装链表主要就是链表的最后一个元素的next指针会指向链表的头部。

首先是数据成员的设计，在实践过程中发现，环装链表最好不要使用first指针，使用last指针才是比较正确的。因为使用first指针的时候如果我们需要在链表的头部增加节点，那么最后一个节点的next指针要指向新的头部。那这就涉及到一个问题，我们怎么自能让最后一个节点的指针指向新的头部，如果我们只有first指针，那么我们只能遍历整个链表，然后去修改最后一个节点的next指针，这样子开销很大。

所以在环形链表中应该使用last指针，last指针指向链表的最后一个节点，这样子无论从链表的头部开始尾部添加元素都会变得比较简单。

另外我们发现在模板中，我们不能为形参设定缺省值。另外加在头部和尾部的节点也要在逻辑上单独处理。

在删除节点的过程中，我们对于最后一个节点的删除要尤其注意，因为删除最后一个节点之后，我们要重新定位last，所以我们需要单独处理，需要单独记录一下倒数第二个节点的指针，以便让last指针定位。

最后在说一下析构函数，析构函数就是不断执行链表第一个元素的删除操作就好了。



## 双向链表

当我们需要向头尾两个方向遍历的时候我们就需要双向链表了，为了体现双向链表的优势，我们打算实现goleft、goright、addleft、addright、deleteleft、deleteright，这几个函数，主要的工作就是让一个指针不断在链表中移动，可以理解为是双向链表这个对象和迭代器的结合体。

其实在实现链表的时候书上一直都建议使用空头部节点的设计，这种方式有很多的好处，可以统一逻辑，减少边界条件的判断。在双向链表的实现中我们也使用这种被推荐的设计。此外我们依旧要实现一个换装双向链表。

这里唯一要注意的问题就是这个空白的头结点的左右指针怎么指，为了保证在遍历链表的过程中，让左边最边上一个元素的在往左边走就是空白头结点，而空白头结点再往左边走是右边第一个节点，这样子循环起来。我们需要精心设计空白头结点的通往左节点的指针和右节点的指针。

有关于双向链表的初始化问题比较有一起，比较精妙的设计是初始化一个空白的头结点，然后让这个空白头结点的左右节点指针全部指向自己，这样子做的好处就是可以统一所有的关于插入和删除节点的逻辑。并且上文所说的头结点左右指针的“精心设计”也可以和一般节点的逻辑统一起来。

意外因为头结点比较重要最好不要删除，所以我们在删除的时候会弹出警告。

这样子双向链表就完成了。

## 树

### 二叉树

树是一种非常重要的数据结构，二叉树更是重中之重。我们可以知道。使用数组来表示一个二叉树是一个非常容易的事情。我们从数组的1号位置开始记录，我们在1号位置存储的是树的根。对于一个深度为k的数组来说，我们最多可以容纳2^k-1个元素。

我们使用数组对二叉树进行了实现，对于i号元素来说，他的左节点就在i\*2处，右节点在i\*2+1处。我们知道在此基础之上注意数组溢出的问题就可以了。

子树的删除使用的是递归程序，在删除子树的根节点之后我们知道再去删除子树的左右节点就可以了。

二叉树的数组实现对于完全二叉树来说是一种比较简单合适的解法，但是如果是不太完全的二叉树，那么就会出现空间的浪费，所以使用二叉树的链表表示更为合适。



# 常用算法整理与总结

这一部分我们主要介绍一些我们无暇实现但是依旧是耳熟能详的算法。我们会尽可能加入一些思路甚至伪代码的实现。

## 插入排序

插入排序的思路就像我们按顺序整理一套扑克牌一样。我们以从小到大排序为例，我们从第一位开始将一个牌从当前位置抽出，然后让这张牌与之前的牌作比较。比这张牌大的牌都依次往前移一位，直到之前的牌比抽出的牌小。这样子就会空出来一个位置，将抽出的牌放到这个位置就好了。插入排序的特点就是我们抽出的牌的前面的牌都是已经排好序的。我们要做的实际上就是不断让一个新的值插入到一个已经排好序的序列中。

伪代码：

```
//插入排序，我们假设为从小到大排序
//形参是输入和数组的长度
insert_sort(A[],len)
	for i <- 0 to len
		do 
		//首先我们获取当前位置的元素
		tmp <- A[i]
		//这里激活一个嵌套循环，让获取的元素和之前的元素做比较
		for j <- i to 0
			do
			if A[j] >= tmp
				then
				A[j+1] = A[j]
			else
				A[j] = tmp
				break;
```

这个是一个时间复杂度为n^2的算法。

## 选择排序

选择排序是一种看起来更笨拙的排序方式，时间复杂度也是n^2。他的总体原理是一开始遍历整个数组，将最小的放在数组的第一个位置。然后遍历数组的剩下部分，将最小的放在数组的第二个位置，直到最后所有的元素都各得其所。

## 合并（归并）排序

### 合并排序算法思路

这是一种比较快的排序算法，时间复杂度达到了nlogn。他的思路就是将整个要排序的集合不断拆分，然后再两两合并，在合并的过程中进行排序操作。归并排序是一个递归算法，要合并的两个子列都是已经经过合并排好序的。所以这个算法的核心就是对已经排好序的两个数组进行合并，合并之后依旧是排好序的数组

数组合并伪代码：

```
//输入为两个数组和两个数组的大小，从小到大的排序
merge(A[],B[],m,n){
	i <- 0
	j <- 0
	q <- 0
	result[m+n]
	while i < m && j < n
		do
		if A[i] > B[j]
			then
			result[q] = B[j]
			j++
			q++
		else
			result[q] = A[i]
			i++
			q++
	//两个数组中有其中一个数组还会剩余元素，我们将其拷到要返回的目标数组中
	for i1 <- i to m
		do
		result[q] = A[i1]
		q++
	
	for j1 <- j to n
		do
		result[q] = A[j1]
		q++
}
```

以上就是归并排序的核心部分，但是实际上外部还需要一个函数来进行驱动

```
//归并排序，形参为要排序的乱序数组
merge_sort(A[],len){
	//将数组进行拆分
	mid = len/2
	A1[mid]
	A2[len-mid-1]
	for i <- 0 to mid
		do
		A1[i] = A[i]
	for j <- 0 to len-mid-1
		do 
		A2[j] = A[mid+1+j]
	merge_sort(A1[] , mid)
	merge_sort(A2[] , len-mid-1)
	A[] = merge(A1[] , A2[] , mid , len-mid-1)
}
```

### 改造归并排序求逆序对个数

这个是一个到腾讯的笔试中出的一个考题，可以通过修改归并排序来获得一个数组中逆序对的个数。我们的主要的思路如下，依旧是一个递归的思路，但是我们需要在递归的过程中加入一个计数的步骤。

假设A[]数组，前后分为A1[m]和A2[n]两个部分。A1中的元素我们在逻辑上不需要特殊处理，然后A2中的元素我们在进行合并的时候要特殊处理。比如，A2中的第x个元素合并完之后就在A中的y位置。这个时候我们就知道这个元素对应的逆序就是，m-(y-x)。y-x算出的是A1中比这个元素小的元素，而m-(y-x)就是A1中比这个元素大的元素，所以这样子就可以算出与这个元素相关的逆序数。通过不断的遍历和向上递归，整个数组的逆序数就算出来了。

## 冒泡排序

这是一种乍一看和插入排序很像的排序方式，但是实际上有着本质上的不同。我们以从小到大排列为例，我们首先要注意冒泡的方向。将从小到大排列中我们可以从右边开始，将小的元素向左冒泡，也可以从左边开始将打的元素向右冒泡。每一轮冒泡都可以讲当前子列最大或者最小值挪到数组的两端。

```
//冒泡即是将打的值从左侧
bubble_sort(A[] , len)
	//冒泡的轮数
	for i <- 0 to len
		do
		//每一轮要冒泡的元素
		for j <- 0 to len - i -1
			do
			if(A[j]>A[j+1])
				tmp = A[j+1]
				A[j+1] = A[j]
				A[j] = tmp
```



## 霍纳规则的多项式运算

这个就是秦九昭算法，将多项式的计算变成了一个每一步都逻辑相同的递归。对于多项式来说我们不断提出前n项共有的X来进行化简。直到最后在每一个都可以进行ax+b的递归。







