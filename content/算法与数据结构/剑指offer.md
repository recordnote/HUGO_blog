---
title: 剑指offer算法题
date: 2021-04-01T14:21:26+08:00
lastmod: 2021-05-22T14:21:26+08:00
author: Aaron
avatar: /me/yy.jpg
cover: https://gitee.com/aaronlynn/picture/raw/master/img/image-20210728174624549.png
images:
  - https://gitee.com/aaronlynn/picture/raw/master/img/image-20210728174624549.png
categories:
  - 算法与数据结构
tags:
  - 算法与数据结构
weight: 1
---



## 1、JZ1  二维数组中的查找

描述  

在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

[

 [1,2,8,9],
 [2,4,9,12],
 [4,7,10,13],
 [6,8,11,15]

]

给定 target = 7，返回 true。

给定 target = 3，返回 false。

```java
/* 思路
* 矩阵是有序的，从左下角来看，向上数字递减，向右数字递增，
* 因此从左下角开始查找，当要查找数字比左下角数字大时。右移
* 要查找数字比左下角数字小时，上移
*/
public class Solution {
    public boolean Find(int target, int [][] array) {
    int rows = array.length;
        if(rows == 0){
            return false;
        }
        int cols = array[0].length;
        if(cols == 0){
            return false;
        }
        // 左下
        int row = rows-1;
        int col = 0;
        while(row>=0 && col<cols){
            if(array[row][col] < target){
                col++;
            }else if(array[row][col] > target){
                row--;
            }else{
                return true;
            }
        }
        return false;
    }
}
```

## 2、JZ2  替换空格

描述

请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

示例1

输入：

```
"We Are Happy"
```

复制

返回值：

```
"We%20Are%20Happy"
```
题解：

两种方法，思路一样

~~~ java
//解法一
public class Solution {
    public String replaceSpace (String s) {
        int length = s.length();
        char [] array = new char[length*3];
        int index = 0;
        for(int i= 0; i< length ;i++){
            char c= s.charAt(i);
            if(c == ' '){
                array[index++] = '%';
                array[index++] = '2';
                array[index++] = '0';
            }else{
                array[index++] = c;
            }
        }
        String newStr = new String (array,0,index);
        return newStr;
    }
}

//解法二
public class Solution {
    public String replaceSpace (String s) {
        StringBuilder stringBuilder = new StringBuilder();
        for(int i= 0 ; i < s.length(); i++){
            if(s.charAt(i) == ' '){
                stringBuilder.append("%20");
            }else{
                stringBuilder.append(s.charAt(i));
            }
        }
        return stringBuilder.toString();
    }
}
~~~

## 3、JZ3  从尾到头打印链表

描述

输入一个链表的头节点，按链表从尾到头的顺序返回每个节点的值（用数组返回）。

如输入{1,2,3}的链表如下图:

![image-20210721102102786](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210721102102786.png) 

返回一个数组为[3,2,1]

0 <= 链表长度 <= 1000

示例

输入：

```
{67,0,24,58}
```

复制

返回值：

```
[58,24,0,67]
```

题解

~~~ java
/**
三种方法： 前两种O（n），第三种O（2n）
复杂度
时间复杂度：
空间复杂度：
*/

/**第一种 非递归
listNode 是链表，只能从头遍历到尾，但是输出却要求从尾到头，这是典型的"先进后出"，我们可以想到栈！
ArrayList 中有个方法是 add(index,value)，可以指定 index 位置插入 value 值
所以我们在遍历 listNode 的同时将每个遇到的值插入到 list 的 0 位置，最后输出 listNode 即可得到逆序链表
*/
import java.util.*;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> list = new ArrayList<>();
        ListNode tmp = listNode;
        while(tmp!=null){
            list.add(0,tmp.val);
            tmp = tmp.next;
        }
        return list;
    }
}

/**第二种 递归
既然非递归都实现了，那么我们也可以利用递归，借助系统的"栈"帮忙打印
*/
import java.util.*;
public class Solution {
    ArrayList<Integer> list = new ArrayList();
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        if(listNode!=null){
            printListFromTailToHead(listNode.next);
            list.add(listNode.val);
        }
        return list;
    }
}

/**第三种
遍历值给数组，数组反转，返回值
*/
import java.util.*;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> arrays = new ArrayList<>();
         ArrayList<Integer> arrays2 = new ArrayList<>();
        for(ListNode p = listNode; p != null; p = p.next){
            arrays.add(p.val);
        }
        for(int i = arrays.size() - 1; i >= 0; i--){
            arrays2.add(arrays.get(i));
        }
        return arrays2;
        
    }
}
~~~



## 4、JZ4  重建二叉树

描述

给定某二叉树的前序遍历和中序遍历，请重建出该二叉树并返回它的头结点。

例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建出如下图所示。

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210721102138599.png" alt="image-20210721102138599" style="zoom:67%;" />

提示:

1.0 <= pre.length <= 2000

2.vin.length == pre.length

3.0 <= pre[i], vin[i] <= 10000

4.pre 和 vin 均无重复元素

5.vin出现的元素均出现在 pre里

6.只需要返回根结点，系统会自动输出整颗树做答案对比

示例1

输入：

```
[1,2,4,7,3,5,6,8],[4,7,2,1,5,3,8,6]
```

返回值：

```
{1,2,3,4,#,5,6,#,7,#,#,8}
```

说明：

```
返回根节点，系统会输出整颗二叉树对比结果     
```

示例2

输入：

```
[1],[1]
```

返回值：

```
{1}
```

示例3

输入：

```
[1,2,3,4,5,6,7],[3,2,4,1,6,5,7]
```

返回值：

```
{1,2,5,3,4,6,7}
```

### **<font color='red'>补充知识：</font>**

<font color='red'>**二叉树的前序、中序、后序遍历**</font>

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210720100922944.png" alt="image-20210720100922944" style="zoom:80%;" /> 

~~~ xml
前序遍历A-B-D-F-G-H-I-E-C

中序遍历F-D-H-G-I-B-E-A-C

后序遍历F-H-I-G-D-E-B-C-A

前序(根左右)，中序(左根右)，后序(左右根)
~~~

### 题解：递归构建二叉树

#### 1. 分析

根据中序遍历和前序遍历可以确定二叉树，具体过程为：

1. 根据前序序列第一个结点确定根结点
2. 根据根结点在中序序列中的位置分割出左右两个子序列
3. 对左子树和右子树分别递归使用同样的方法继续分解

例如：
前序序列{1,2,4,7,3,5,6,8} = pre
中序序列{4,7,2,1,5,3,8,6} = in

1. 根据当前前序序列的第一个结点确定根结点，为 1

2. 找到 1 在中序遍历序列中的位置，为 in[3]

3. 切割左右子树，则 in[3] 前面的为左子树， in[3] 后面的为右子树

4. 则切割后的**左子树前序序列**为：{2,4,7}，切割后的**左子树中序序列**为：{4,7,2}；切割后的**右子树前序序列**为：{3,5,6,8}，切割后的**右子树中序序列**为：{5,3,8,6}

5. 对子树分别使用同样的方法分解

#### 2. 代码

复杂度

时间复杂度：O(n)
空间复杂度：O(n)

~~~ java
/**
 * Definition for binary tree
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
import java.util.Arrays;
public class Solution {
    public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
        if (pre.length == 0 || in.length == 0) {
            return null;
        }
        TreeNode root = new TreeNode(pre[0]);
        // 在中序中找到前序的根
        for (int i = 0; i < in.length; i++) {
            if (in[i] == pre[0]) {
                // 左子树，注意 copyOfRange 函数，左闭右开
                root.left = reConstructBinaryTree(Arrays.copyOfRange(pre, 1, i + 1), Arrays.copyOfRange(in, 0, i));
                // 右子树，注意 copyOfRange 函数，左闭右开
                root.right = reConstructBinaryTree(Arrays.copyOfRange(pre, i + 1, pre.length), Arrays.copyOfRange(in, i + 1, in.length));
                break;
            }
        }
        return root;
    }
}
~~~

## 5、JZ5 用两个栈实现队列

描述

用两个栈来实现一个队列，分别完成在队列尾部插入整数(push)和在队列头部删除整数(pop)的功能。 队列中的元素为int类型。保证操作合法，即保证pop操作时队列内已有元素。

示例:

输入:

["PSH1","PSH2","POP","POP"]

返回:

1,2

解析:

"PSH1":代表将1插入队列尾部

"PSH2":代表将2插入队列尾部

"POP“:代表删除一个元素，先进先出=>返回1

"POP“:代表删除一个元素，先进先出=>返回2

示例1

输入：

```
["PSH1","PSH2","POP","POP"]
```

返回值：

```
1,2
```

题解：

~~~ java
/**
思路：两个栈做相反操作 等于 队列的先进先出
*/

import java.util.Stack;

public class Solution {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();
    
    public void push(int node) {
        stack1.push(node);
    }
    
    public int pop() {
        if(stack2.empty()){
            while(!stack1.empty()){
               stack2.push(stack1.pop());
            }
        }
  
        return stack2.pop();
    }
}
~~~

## 6、**JZ6** **旋转数组的最小数字**

描述

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

示例1

输入：

```
[3,4,5,1,2]
```

复制

返回值：

```
1
```

### 题解 ：采用二分法

#### 分析：

采用二分法解答这个问题，

mid = low + (high - low)/2

需要考虑三种情况：

(1)array[mid] > array[high]:

出现这种情况的array类似[3,4,5,6,0,1,2]，此时最小数字一定在mid的右边。

low = mid + 1

(2)array[mid] == array[high]:

出现这种情况的array类似 [1,0,1,1,1] 或者[1,1,1,0,1]，此时最小数字不好判断在mid左边

还是右边,这时只好一个一个试 ，

high = high - 1

(3)array[mid] < array[high]:

出现这种情况的array类似[2,2,3,4,5,6,6],此时最小数字一定就是array[mid]或者在mid的左

边。因为右边必然都是递增的。

high = mid

**注意这里有个坑：如果待查询的范围最后只剩两个数，那么mid** **一定会指向下标靠前的数字**

比如 array = [4,6]

array[low] = 4 ;array[mid] = 4 ; array[high] = 6 ;

如果high = mid - 1，就会产生错误， 因此high = mid

但情形(1)中low = mid + 1就不会错误

#### 代码：

~~~ java
import java.util.ArrayList;
public class Solution {
    public int minNumberInRotateArray(int [] array) {
        int low = 0 ; int high = array.length - 1;   
        while(low < high){
            int mid = low + (high - low) / 2;        
            if(array[mid] > array[high]){
                low = mid + 1;
            }else if(array[mid] == array[high]){
                high = high - 1;
            }else{
                high = mid;
            }   
        }
        return array[low];
    }
}
~~~

## 7、**JZ7** **斐波那契数列**

描述

大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0，第1项是1）。

*n*≤39

示例1

输入：

```
4
```

返回值：

```
3
```

题解：**f(n) = f(n-1) + f(n-2)  **     <font color='red'> **0 1 1 2 3 5**</font>

~~~ java
/**
*第一种用递归
*时间复杂度：O(2^n)
*空间复杂度：递归栈的空间
*/
public class Solution {
    public int Fibonacci(int n) {
       if(n == 0 || n == 1){
            return n;
        }
        return Fibonacci(n-1) + Fibonacci(n-2);
    }
}

/**
*第二种 动态规划
*时间复杂度：O(n)
*空间复杂度：O(1)
*/
public class Solution {
    public int Fibonacci(int n) {
        if(n == 0 || n == 1){
            return n;
        }
        int a = 0, b = 1, c = 0;
        for (int i = 2; i <= n; i++) {
            c = a + b;
            a = b;
            b = c;
        }
        return c;
    }
}
~~~

## 8、**JZ8** **青蛙跳台阶**

描述

一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。

示例1

输入：

```
2
```

返回值：

```
2
```

示例2

输入：

```
7
```

返回值：

```
21
```

题解: 

分析：

![image-20210720204449750](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210720204449750.png)

[斐波那契数列（四种解法）](https://blog.csdn.net/weixin_42292229/article/details/104505402)

~~~
一阶：口
1
有1种跳法

二阶：口口
1  1
2
有2种跳法

三阶：口口口
 1 1 1
 1 2
 2 1
有3种跳法

四阶口口口口
1 1 1 1
1 1 2
1 2 1
2 1
2 2
有5种跳法

跳法规律  ： 1 2 3 5 .......
f(n) = f(n-1) + f(n-2)
~~~

**方法一：**
面试别写型递推版实现，时间复杂度 *O*(2^n)

```java
public class Solution {
    public int JumpFloor(int n) {
        if (n == 1) return 1; 
        if (n == 2) return 2;
        return JumpFloor(n - 1) + JumpFloor(n - 2);
    }
}
```

**方法二：**
自底向上型循环求解，时间复杂度为 O(n)。

```java
public class Solution {
    public int jumpFloor(int target) {
        if(target == 0 || target == 1 || target == 2){
            return target;
        }
        int a = 1, b = 2, c = 0;
        for (int i = 3; i <= target; i++) {
            c = a + b;
            a = b;
            b = c;
        }
        return c;
    }
}
```

## 9、**JZ9** **青蛙跳台阶扩展问题（变态版）**

描述

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶(n为正整数)总共有多少种跳法。

示例1

输入：

```
3
```

返回值：

```
4
```

分析：

```xml
4阶 口口口口
1 1 1 1
1 1 2
1 2 1
1 3
2 1 1
2 2
3 1
4
总共8种跳法

3阶 口口口
1 1 1
1 2
2 1
3
总共4种跳法

2阶 口口口
1 1
2
总共2种跳法

1阶 口
1
总共1种

跳法： 1 2 4 8 ... 2^(n-1)

```

**【分析】**
每个台阶可以看作一块木板，让青蛙跳上去，n个台阶就有n块木板，最后一块木板是青蛙到达的位子，
必须存在，其他 (n-1) 块木板可以任意选择是否存在，则每个木板有存在和不存在两种选择，(n-1) 块木板
就有 [2^(n-1)] 种跳法，可以直接得到结果。



其实我们所要求的序列为：0,1,2,4,8,16,……

**<font color='red'>所以除了第一位外，其他位的数都是前一位的数去乘以2所得到的积。</font>**



题解：

~~~ java
/**
*第一种写法
*/
public class Solution {
    public int jumpFloorII(int target) {
        if(target == 0){
            return 0;
        }
        int sum = 1;
        for (int i = 1; i <= target - 1; i++) {
            sum *= 2;
        }
        return sum;
        
    }
}

/**
*第二种写法 递归
*/
public class Solution {
    public int jumpFloorII(int target) {
        if(target == 0){
            return 0;
        }else if(target == 1){
            return 1;
        }else{
            return 2*jumpFloorII(target - 1);
        }   
    }
}

~~~

## 10、**JZ10** **矩形覆盖**

描述

我们可以用2x1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2X1的小矩形无重叠地覆盖一个2xn的大矩形，从同一个方向看总共有多少种不同的方法？

比如n=3时，2*3的矩形块有3种不同的覆盖方法(从同一个方向看)：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210721141607160.png" alt="image-20210721141607160" style="zoom: 50%;" /> 

输入描述：

2*1的小矩形的总个数n

返回值描述：

覆盖一个2*n的大矩形总共有多少种不同的方法(从同一个方向看)

示例1

输入：

```
0
```

返回值：

```
0
```

示例2

输入：

```
1
```

返回值：

```
1
```

示例3

输入：

```
4
```

返回值：

```
5
```

### 分析：

递推

对于这种题没有思路怎么办？

那就对n 从小到大，一步步分析：

![image-20210721160942885](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210721160942885.png) 

n=1时，显然只有一种方法

![image-20210721161015941](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210721161015941.png) 

n=2时，如图有2种方法

![image-20210721161028733](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210721161028733.png) 

n=3，如图有3中方法

 <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210721161053281.png" style="zoom: 80%;" />

n=4,如图有5种方法。

如果到这里，还没有发现规律怎么办呢？

那我们就再分析以下，从n=3到n=4，怎么来的呢？

这里有2种情况：

- 直接在n=3的情况下，再后面中添加一个竖着的。这个很显然成立，有3种情况
- 然后横着的显然能添加到n-2的情况上，也就是在n=2后面，添加2个横着的。有2种情况

通过以上分析，发现刚好和图中的个数一样。

所以总结：f [n]表示2*n大矩阵 的方法数。

可以得出：f[n] = f[n-1] + f[n-2]，初始条件f[1] = 1, f[2] =2

所以代码可用递归，记忆递归，和动态规划和递推


### 题解：

```java
public class Solution {
    public int rectCover(int target) {
        if(target <= 2){
            return target;
        }
        int a = 1, b = 2, c = 0;
        for(int i = 3;i<= target;i++){
            c = a + b;
            a = b;
            b = c;
        }
        return b;
    }
}
```

## 11、 **JZ11** **二进制中1的个数**

描述

输入一个整数，输出该数32位二进制表示中1的个数。其中负数用补码表示。

示例1

输入：

```
10
```

返回值：

```
2
```

分析：

如果一个整数不为0，那么这个整数至少有一位是1。如果我们把这个整数减1，那么原来处在整数最右边的1就会变为0，原来在1后面的所有的0都会变成1(如果最右边的1后面还有0的话)。其余所有位将不会受到影响。
举个例子：一个二进制数1100，从右边数起第三位是处于最右边的一个1。减去1后，第三位变成0，它后面的两位0变成了1，而前面的1保持不变，因此得到的结果是1011.我们发现减1的结果是把最右边的一个1开始的所有位都取反了。这个时候如果我们再把原来的整数和减去1之后的结果做与运算，从原来整数最右边一个1那一位开始所有位都会变成0。如1100&1011=1000.也就是说，把一个整数减去1，再和原整数做与运算，会把该整数最右边一个1变成0.那么一个整数的二进制有多少个1，就可以进行多少次这样的操作

**<font color='red'>补充知识：</font>**

1）与运算符（&）

运算规则：

0&0=0；0&1=0；1&0=0；1&1=1

即：<font color='red'>**两个同时为1，结果为1**</font>，否则为0

例如：3&5

十进制3转为二进制的3：0000 0011

十进制5转为二进制的5：0000 0101

------------------------结果：0000 0001 ->转为十进制：1

即：3&5 = 1

2）或运算（|）

运算规则：

0|0=0； 0|1=1； 1|0=1；  1|1=1；

即 ：参加运算的两个对象，<font color='red'>**一个为1，其值为1**</font>

例如：3|5　即 00000011 | 0000 0101 = 00000111，因此，3|5=7。　

3）异或运算符（^）

运算规则：0^0=0； 0^1=1； 1^0=1；  1^1=0；

即：参加运算的两个对象，如果两个位为<font color='red'>**“异”（值不同），则该位结果为1**</font>，否则为0。

例如：3^5 = 0000 0011 | 0000 0101 =0000 0110，因此，3^5 = 6

[**负数补码表示**](https://blog.csdn.net/baidu_35679960/article/details/80364315)

题解：

```java
public class Solution {
    public int NumberOf1(int n) {
        
        int count=0;
        while(n!=0){
            count++;
            n=n&(n-1);
        }
        return count;
    }
}
```

## 12、**JZ12** **数值的整数次方**

描述

给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。

保证base和exponent不同时为0。不得使用库函数，同时不需要考虑大数问题，也不用考虑小数点后面0的位数。

示例1

输入：

```
2.00000,3
```

返回值：

```
8.00000
```

示例2

输入：

```
2.10000,3
```

返回值：

```
9.26100
```

示例3

输入：

```
2.00000,-2
```

返回值：

```
0.25000
```

说明：

```
2的-2次方等于1/4=0.25
```

### 分析：

第一种 暴力法

第二种 递归法（快速幂）

 <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210723105148744.png" alt="image-20210723105148744" style="zoom:80%;" />

第三种 非递归的快速幂

![image-20210723111649577](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210723111649577.png)

### 题解：

~~~ java
//第一种 暴力法 
//时间复杂度：O(n)
//空间复杂度：O(1)
public class Solution {
    public double Power(double base, int exponent) {
        if(exponent == 0){
            return 1;
        }
        if(exponent == 1){
            return base;
        }
        int e = exponent > 0 ? exponent : -exponent;
        Double result = 1.0d;
        for (int i = 1; i<=e; i++){
            result *= base;
        }
        return exponent > 0 ? result : 1/result;
  }
}

/** 
*第二种 递归法（快速幂）
*时间复杂度：O(logn)，每次规模减少一半
*空间复杂度：O(logn)，递归栈，因为要记住logn个变量
*/
public class Solution {
    public double Power(double base, int exponent) {
        if(exponent<0){
            exponent = -exponent;
            base = 1/base;
        }
        if(exponent == 0){
            return 1.0;
        }

        double ret = Power(base, exponent/2);
        if((exponent&1) == 1){
            //指数为奇数
            return ret*ret*base;
        }else{
            return ret*ret;
        }
  }
}

/**
*第三种 非递归的快速幂
*时间复杂度：O(logn)，因为n的二进制位个数为logn
*空间复杂度：O(1)
*/
public class Solution {
    public double Power(double base, int exponent) {
        if (exponent < 0) {
            base = 1 / base;
            exponent = -exponent;
        }
        double x = base; // 记录x^0, x^1, x^2 ...
        double ret = 1.0;
        while (exponent > 0) {
            if ((exponent&1) == 1) {
                ret *= x; // 二进制位数是1的，乘进答案。
            }
            x *= x;
            //指数位数右移一位
            exponent >>= 1;
        }
        return ret;
  }
}


~~~

## 13、 **JZ13** **调整数组顺序使奇数位于偶数前面**

描述

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

示例1

输入：

```
[1,2,3,4]
```

返回值：

```
[1,3,2,4]
```

示例2

输入：

```
[2,4,6,5,7]
```

返回值：

```
[5,7,2,4,6]
```

题解：
~~~ java
import java.util.*;
/**
*时间复杂度O(n)
*空间复杂度O(n)
*/
public class Solution {
    public int[] reOrderArray (int[] array) {
        Queue<Integer> oddQueue = new LinkedList<>();
        Queue<Integer> evenQueue = new LinkedList<>();
        for (int i = 0; i < array.length; i++) {
            if((array[i]&1) == 1){
                //为奇数
                oddQueue.add(array[i]);
            }else{
                evenQueue.add(array[i]);
            }
        }

        for (int i = 0; i < array.length; i++) {
            if(!oddQueue.isEmpty()){
                array[i] = oddQueue.poll();
            }else {
                array[i] = evenQueue.poll();
            }
        }
        return array;
    }
}
~~~

## 14、**JZ14** **链表中倒数最后k个结点**

描述

输入一个链表，输出一个链表，该输出链表包含原链表中从倒数第k个结点至尾节点的全部节点。

如果该链表长度小于k，请返回一个长度为 0 的链表。

示例1

输入：

```
{1,2,3,4,5},1 
```

返回值：

```
{5}
```

分析：

在链表中：倒数的+顺数的长度等于链表总长度，所以可以设置两个指针，一个先走K步，剩下的到链表的末尾要走的步数就是倒数第k个节点，需要从头开始走的步数

~~~ java
import java.util.*;

/*
 * public class ListNode {
 *   int val;
 *   ListNode next = null;
 *   public ListNode(int val) {
 *     this.val = val;
 *   }
 * }
 */

public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 
     * @param pHead ListNode类 
     * @param k int整型 
     * @return ListNode类
     */
    public ListNode FindKthToTail (ListNode pHead, int k) {
        if(pHead == null){
            return null;
        }
        ListNode first = pHead;
        ListNode second = pHead;

        for (int i = 0; i < k; i++) {
            if(first == null){
                //判断k可能大于链表节点数，
                return null;
            }
            //第一个指针走k个位置
            first = first.next;
        }
        while (first != null){
            //第一个指针走完最后一步，第二个指针的位置就是倒数k
            first = first.next;
            second = second.next;
        }
        return second;
    }
}
~~~

## 15、**JZ15** **反转链表**

描述

输入一个链表，反转链表后，输出新链表的表头。

示例1

输入：

```
{1,2,3}
```

返回值：

```
{3,2,1}
```

分析：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210727153028017.png" alt="image-20210727153028017" style="zoom:80%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210727152952167.png" alt="image-20210727152952167" style="zoom:80%;" /> 

题解：

~~~ java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
import java.util.*;
public class Solution {
    public ListNode ReverseList(ListNode head) {
       if(head == null){
            return null;
        }
        ListNode pre = null;
        ListNode cur = head;
        while (cur != null){
            //反转 ==》指针反转
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur;	
            cur = next;
        }
        return pre;
    }
}
~~~

## 16、**JZ16** **合并两个排序的链表**

描述

输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

示例1

输入：

```
{1,3,5},{2,4,6}
```

返回值：

```
{1,2,3,4,5,6}
```

题解：

```java
/**
*第一种
*
*
*
*
*
*/
public class Solution {
    public ListNode Merge(ListNode list1,ListNode list2) {
		if(list1 == null && list2 == null){
            return null;
        }
        if(list1 == null){
            return list2;
        }
        if(list2 == null){
            return list1;
        }
        ListNode head = new ListNode(0); //用来做合并的链表
        ListNode root = head; //保存头节点引用
        
        while(list1 != null && list2 != null){
            if(list1.val < list2.val){
                head.next = list1;
                list1 = list1.next;
            }else {
                head.next = list2;
                list2 = list2.next;
            }
            head = head.next;
        }
        //考虑可能会有一个链表走完的情况，将未走完链表接在合并链表后面
        if(list1 == null){
            head.next = list2;
        }
        if(list2 == null){
            head.next = list1;
        }
        return  root.next;
    }
}

/**
*第二种
*递归版本，可以练习递归代码。
*写递归代码，最重要的要明白递归函数的功能。可以不必关心递归函数的具体实现。
*比如这个ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
*函数功能：合并两个单链表，返回两个单链表头结点值小的那个节点。
*时间复杂度：O(m+n)
*空间复杂度：O(m+n),每一次递归，递归栈都会保存一个变量，最差情况会保存(m+n)个变量
*/
public class Solution {
    public ListNode Merge(ListNode list1,ListNode list2) {
        if(list1 == null){
            return list2;
        }
        if(list2 == null){
            return list1;
        }
       if(list1.val < list2.val){
            list1.next = Merge(list1.next, list2);
            return list1;
       }else {
           list2.next = Merge(list1, list2.next);
           return list2;
       }
    }
}

```

## 17、**JZ17** **树的子结构**

描述

输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）

示例1

输入：

```
{8,8,#,9,#,2,#,5},{8,9,#,2}
```

返回值：

```
true
```

题解：

~~~ java
/**
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    public boolean HasSubtree(TreeNode root1,TreeNode root2) {
      boolean result = false;
        //当Tree1和Tree2都不为零的时候，才进行比较。否则直接返回false
        if (root2 != null && root1 != null) {
            //如果找到了对应Tree2的根节点的点
            if(root1.val == root2.val){
                //以这个根节点为为起点判断是否包含Tree2
                result = doesTree1HaveTree2(root1,root2);
            }
            //如果找不到，那么就再去root的左儿子当作起点，去判断时候包含Tree2
            if (!result) {
                result = HasSubtree(root1.left,root2);
            }
             
            //如果还找不到，那么就再去root的右儿子当作起点，去判断时候包含Tree2
            if (!result) {
                result = HasSubtree(root1.right,root2);
               }
            }
            //返回结果
        return result;
    }
    public static boolean doesTree1HaveTree2(TreeNode node1, TreeNode node2) {
        //如果Tree2已经遍历完了都能对应的上，返回true
        if (node2 == null) {
            return true;
        }
        //如果Tree2还没有遍历完，Tree1却遍历完了。返回false
        if (node1 == null) {
            return false;
        }
        //如果其中有一个点没有对应上，返回false
        if (node1.val != node2.val) {  
            return false;
        }
         
        //如果根节点对应的上，那么就分别去子节点里面匹配
        return doesTree1HaveTree2(node1.left,node2.left) && doesTree1HaveTree2(node1.right,node2.right);
    }
}
~~~

## 18、**JZ18** **二叉树的镜像**

描述

操作给定的二叉树，将其变换为源二叉树的镜像。

```
比如：    源二叉树 
            8
           /  \
          6   10
         / \  / \
        5  7 9 11
        镜像二叉树
            8
           /  \
          10   6
         / \  / \
        11 9 7  5
```

示例1

输入：

```
{8,6,10,5,7,9,11}
```

返回值：

```
{8,10,6,11,9,7,5}
```

分析：

题解：

~~~ java
import java.util.*;

/*
 * public class TreeNode {
 *   int val = 0;
 *   TreeNode left = null;
 *   TreeNode right = null;
 *   public TreeNode(int val) {
 *     this.val = val;
 *   }
 * }
 */

public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 
     * @param pRoot TreeNode类 
     * @return TreeNode类
     */
    public TreeNode Mirror (TreeNode pRoot) {
        if (pRoot == null){
            return null;
        }
        TreeNode left = Mirror(pRoot.left);
        TreeNode right = Mirror(pRoot.right);
        pRoot.left = right;
        pRoot.right = left;
        return pRoot;
    }

}
~~~

## 19、 **JZ19** **顺时针打印矩阵**

描述

输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下4 X 4矩阵：

```
[[1,2,3,4],
[5,6,7,8],
[9,10,11,12],
[13,14,15,16]]
```

则依次打印出数字

```
[1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10]
```

示例1

输入：

```
[[1,2],[3,4]]
```

返回值：

```
[1,2,4,3]
```

分析：

简单来说，就是不断地收缩矩阵的边界
定义四个变量代表范围，up、down、left、right

1. 向右走存入整行的值，当存入后，该行再也不会被遍历，代表上边界的 up 加一，同时判断是否和代表下边界的 down 交错
2. 向下走存入整列的值，当存入后，该列再也不会被遍历，代表右边界的 right 减一，同时判断是否和代表左边界的 left 交错
3. 向左走存入整行的值，当存入后，该行再也不会被遍历，代表下边界的 down 减一，同时判断是否和代表上边界的 up 交错
4. 向上走存入整列的值，当存入后，该列再也不会被遍历，代表左边界的 left 加一，同时判断是否和代表右边界的 right 交错

题解：

~~~ java
import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> printMatrix(int [][] matrix) {
       ArrayList<Integer> list = new ArrayList<>();
        if(matrix == null || matrix.length == 0 || matrix[0].length == 0){
            return list;
        }
        int up = 0; //第一行
        int down = matrix.length - 1; //最后一行
        int left = 0; //最左边
        int right = matrix[0].length - 1; //最右边

        while(true){
            for (int col = left; col <= right; col++) {
                list.add(matrix[up][col]);
            }
            up++;
            if(up > down){
                break;
            }
            for (int row = up; row <= down; row++) {
                list.add(matrix[row][right]);
            }
            right--;
            if(right < left){
                break;
            }
            for (int col = right; left <= col ; col--) {
                list.add(matrix[down][col]);
            }
            down--;
            if(down < up){
                break;
            }
            for (int row = down; row >= up ; row--) {
                list.add(matrix[row][left]);
            }
            left++;
            if(left > right){
                break;
            }
        }
        return list;
    }
}
~~~

## 20、**JZ20** **包含min函数的栈**

描述

定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数，并且调用 min函数、push函数 及 pop函数 的时间复杂度都是 O(1)

push(value):将value压入栈中

pop():弹出栈顶元素

top():获取栈顶元素

min():获取栈中最小元素

示例:

输入:  ["PSH-1","PSH2","MIN","TOP","POP","PSH1","TOP","MIN"]

输出:  -1,2,1,-1

解析:

"PSH-1"表示将-1压入栈中，栈中元素为-1

"PSH2"表示将2压入栈中，栈中元素为2，-1

“MIN”表示获取此时栈中最小元素==>返回-1

"TOP"表示获取栈顶元素==>返回2

"POP"表示弹出栈顶元素，弹出2，栈中元素为-1

"PSH-1"表示将1压入栈中，栈中元素为1，-1

"TOP"表示获取栈顶元素==>返回1

“MIN”表示获取此时栈中最小元素==>返回-1

示例1

输入：

```
 ["PSH-1","PSH2","MIN","TOP","POP","PSH1","TOP","MIN"]
```

返回值：

```
-1,2,1,-1
```

分析：借助辅助栈

首先需要一个正常栈normal,用于栈的正常操作，然后需要一个辅助栈minval，专门用于获取最小值

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/284295_1587290406796_0EDB8C9599BA026855B6DCCC1D5EDAE5" alt=" " style="zoom:67%;" /> 

时间复杂度：O(1)
空间复杂度：O(n), 开辟了一个辅助栈。

题解：

~~~ java
import java.util.Stack;

public class Solution {

    Stack<Integer> stack1 = new Stack<>(); 
    Stack<Integer> stack2 = new Stack<>(); 
    public void push(int node) {
        stack1.push(node);
        if(stack2.empty()){
            stack2.push(node);
        }else{
            stack2.push(Math.min(node,stack2.peek()));
        }
    }
    
    public void pop() {
        stack1.pop();
        stack2.pop();
    }
    
    public int top() {
        return stack1.peek();
    }
    
    public int min() {
        return stack2.peek();
    }
}
~~~

## 21、**JZ21** **栈的压入、弹出序列**

描述

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

示例1

输入：

```
[1,2,3,4,5],[4,3,5,1,2]
```

返回值：

```
false
```

分析：新建一个栈，将数组A压入栈中，当栈顶元素等于数组B时，就将其出栈，当循环结束时，判断栈是否为空，若为空则返回true.

题解：

~~~ java
/**
时间复杂度：O(n)
空间复杂度：O(n), 用了一个辅助栈，最坏情况下会全部入栈
*/

public class Solution {
    public boolean IsPopOrder(int [] pushA,int [] popA) {
        if(pushA.length == 0 || popA.length == 0){
            return false;
        }
        int j = 0;
        Stack<Integer> stack = new Stack<>();
        for (int i = 0; i < pushA.length; i++){
            stack.push(pushA[i]);
            while (!stack.isEmpty() && stack.peek() == popA[j] ){
                stack.pop();
                j++;
            }
        }
        return stack.isEmpty();
    }
}
~~~

## 22、**JZ22** **从上往下打印二叉树**

描述

从上往下打印出二叉树的每个节点，同层节点从左至右打印。

示例1

输入：

```
{5,4,#,3,#,2,#,1}
```

返回值：

```
[5,4,3,2,1]
```

分析：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210728161409544.png" alt="image-20210728161409544" style="zoom: 80%;" /> 

题解：

~~~ java
import java.util.*;
/**
*思路是用arraylist模拟一个队列来存储相应的TreeNode
*/
public class Solution {
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        if(root == null){
            return new ArrayList<>();
        }
        Deque<TreeNode> deque = new LinkedList<>(); //将节点放入队列
        ArrayList<Integer> list = new ArrayList<>();
        deque.addLast(root);
        while(!deque.isEmpty()){
            TreeNode node = deque.pollFirst();
            list.add(node.val);

            if(node.left != null){
                deque.addLast(node.left);
            }
            if(node.right != null){
                deque.addLast(node.right);
            }
        }
        return list;
    }
}
~~~

## 23、**JZ23** **二叉搜索树的后序遍历序列**

描述

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则返回true,否则返回false。假设输入的数组的任意两个数字都互不相同。（ps：我们约定空树不是二叉搜索树）

示例1

输入：

```
[4,8,6,12,16,14,10]
```

返回值：

```
true
```

分析：
		思路通过根节点，将数组划分为两部分，假设根节点为x，则左子树 < x , 右子树 > x。通过递归即可，递归的每一层都涉及到对序列的遍历，虽然层数越深节点越少（少了子树的根节点），但是这种减少是微不足道的，即使是到了最底层，依旧有n/2的节点（完美二叉树第i层节点数是其上所有节点数之和+1），因此递归方法在每一层的遍历开销是O(n)，而对于二叉树而言，递归的层数平均是O(logn)，因此，递归方法的最终复杂度是O(nlogn).

[另一种大神解法上限约束法](https://blog.nowcoder.net/n/8fe97e67996249ccbe71328d3a49c4af?f=comment)

 <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210728213700344.png" alt="image-20210728213700344" style="zoom:80%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210728213620840.png" alt="image-20210728213620840" style="zoom:80%;" /> 

题解：

~~~ java

import java.util.*;
/**
*时间复杂度：O(nlogn）
*
*
*/
public class Solution {
    public boolean VerifySquenceOfBST(int [] sequence) {
         if(sequence.length < 1){
            return false;
        }
        ArrayList<Integer> list = new ArrayList<>();
        for(int num : sequence){
            list.add(num);
        }
        return VerifyBST(list);
    }
    
    public  boolean VerifyBST(ArrayList<Integer> list) {
        if(list.size() < 1){
            //递归结束条件
            return true;
        }
        int mid = list.get(list.size() - 1);
        ArrayList<Integer> leftList = new ArrayList<>();
        ArrayList<Integer> rightList = new ArrayList<>();

        int i = 0;
        while(list.get(i) < mid){
            leftList.add(list.get(i++));
        }
        while(list.get(i) > mid){
            rightList.add(list.get(i++));
        }

        if(i < list.size() - 1){
            //判断是否走完数组列表
            //正常走完数组 i = list.size() - 1
            return false;
        }

        return VerifyBST(leftList) && VerifyBST(rightList);
    }
}
~~~



## 24、**JZ24** **二叉树中和为某一值的路径**

描述

输入一颗二叉树的根节点和一个整数，按字典序打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。

示例1

输入：

```
{10,5,12,4,7},22
```

返回值：

```
[[10,5,7],[10,12]]
```

示例2

输入：

```
{10,5,12,4,7},15
```

返回值：

```
[]
```

分析：使用深度优先遍历（DFS）

题解：

~~~ java

public class Solution {
    private ArrayList<ArrayList<Integer>> resultList = new ArrayList<>();
    private ArrayList<Integer> tempList = new ArrayList<>();
    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root,int target) {
        
        if(root == null){
            //叶子节点走完返回列表
            return resultList;
        }
        
        tempList.add(root.val);
        
        if (root.left == null && root.right == null && target - root.val == 0){
            resultList.add(new ArrayList<Integer>(tempList));
        }

        FindPath(root.left, target - root.val);
        FindPath(root.right, target - root.val);
        
        //移除最后一个元素，深度遍历完一条路径后要回退
        tempList.remove(tempList.size()-1);
        
        return resultList;
    }
}

~~~

## 25、**JZ25** **复杂链表的复制**

描述

输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针random指向一个随机节点），请对此链表进行深拷贝，并返回拷贝后的头结点。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）。 下图是一个含有5个结点的复杂链表。图中实线箭头表示next指针，虚线箭头表示random指针。为简单起见，指向null的指针没有画出。

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210729154454708.png" alt="image-20210729154454708" style="zoom:50%;" />  

示例:

输入:{1,2,3,4,5,3,5,#,2,#}

输出:{1,2,3,4,5,3,5,#,2,#}

解析:我们将链表分为两段，前半部分{1,2,3,4,5}为ListNode，后半部分{3,5,#,2,#}是随机指针域表示。

以上示例前半部分可以表示链表为的ListNode:1->2->3->4->5

后半部分，3，5，#，2，#分别的表示为

1的位置指向3，2的位置指向5，3的位置指向null，4的位置指向2，5的位置指向null

如下图:

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210729154430590.png" alt="image-20210729154430590" style="zoom:67%;" /> 

示例1

输入：

```
{1,2,3,4,5,3,5,#,2,#}
```

返回值：

```
{1,2,3,4,5,3,5,#,2,#}
```

分析：

题解：

~~~ java
/*
public class RandomListNode {
    int label;
    RandomListNode next = null;
    RandomListNode random = null;

    RandomListNode(int label) {
        this.label = label;
    }
}
*/
import java.util.*;
public class Solution {
    HashMap<RandomListNode,RandomListNode> map = new HashMap<>();
    public RandomListNode Clone(RandomListNode pHead) {
        if(pHead == null)
            return null;

        if(map.containsKey(pHead)){
            return map.get(pHead);
        }

        RandomListNode tempNode = new RandomListNode(pHead.label);
        map.put(pHead, tempNode);
        tempNode.next = Clone(pHead.next);
//         tempNode.random = Clone(pHead.random); // 这一步目的：防止cannot used original node of list
        tempNode.random = map.get(pHead.random);
        
        return tempNode;
    }
}
~~~

## 26、**JZ26** **二叉搜索树与双向链表**

描述

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。如下图所示

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/E1F1270919D292C9F48F51975FD07CE2" alt="img" style="zoom: 33%;" /> 

注意:

1.要求不能创建任何新的结点，只能调整树中结点指针的指向。当转化完成以后，树中节点的左指针需要指向前驱，树中节点的右指针需要指向后继
2.返回链表中的第一个节点的指针
3.函数返回的TreeNode，有左右指针，其实可以看成一个双向链表的数据结构
4.你不用输出或者处理，示例中输出里面的英文，比如"From left to right are:"这样的，程序会根据你的返回值自动打印输出

示例:

输入: {10,6,14,4,8,12,16}

输出:From left to right are:4,6,8,10,12,14,16;From right to left are:16,14,12,10,8,6,4;

解析:

输入就是一棵二叉树，如上图，输出的时候会将这个双向链表从左到右输出，以及从右到左输出，确保答案的正确

示例1

输入：

```
{10,6,14,4,8,12,16}
```

返回值：

```
From left to right are:4,6,8,10,12,14,16;From right to left are:16,14,12,10,8,6,4;
```

示例2

输入：

```
{5,4,#,3,#,2,#,1}
```

返回值：

```
From left to right are:1,2,3,4,5;From right to left are:5,4,3,2,1;
```

说明：

```
                    5
                  /
                4
              /
            3
          /
        2
      /
    1
树的形状如上图  
```

分析：



题解：

~~~ java
/**
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    public TreeNode Convert(TreeNode pRootOfTree) {
        
    }
}
~~~

## 27、**JZ27** **字符串的排列**

描述

输入一个字符串，打印出该字符串中字符的所有排列，你可以以任意顺序返回这个字符串数组。例如输入字符串abc,则按字典序打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。

输入描述：

输入一个字符串,长度不超过9(可能有字符重复),字符只包括大小写字母。

示例1

输入：

```
"ab"
```

返回值：

```
["ab","ba"]
```

说明：

```
返回["ba","ab"]也是正确的  
```

示例2

输入：

```
"aab"
```

返回值：

```
["aab","aba","baa"]
```

示例3

输入：

```
"abc"
```

返回值：

```
["abc","acb","bac","bca","cab","cba"]
```

分析：

题解：

~~~ java
import java.util.ArrayList;
public class Solution {
    public ArrayList<String> Permutation(String str) {
       
    }
}
~~~

## 28、 **JZ28** **数组中出现次数超过一半的数字**

描述

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组[1,2,3,2,2,2,5,4,2]。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。你可以假设数组是非空的，并且给定的数组总是存在多数元素。1<=数组长度<=50000，0<=数组元素<=10000

示例1

输入：

```
[1,2,3,2,2,2,5,4,2]
```

返回值：

```
2
```

示例2

输入：

```
[3,3,3,3,2,2,2]
```

返回值：

```
3
```

示例3

输入：

```
[1]
```

返回值：

```
1
```

分析：

题解：

~~~ java
public class Solution {
    public int MoreThanHalfNum_Solution(int [] array) {
        
    }
}
~~~













