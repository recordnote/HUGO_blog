---
title: 剑指offer算法题
date: 2021-03-01T14:21:26+08:00
lastmod: 2021-03-01T14:21:26+08:00
author: Aaron
avatar: /me/yy.jpg
cover: /img/java.png
images:
  - /img/java.png
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

## 8、**JZ8** **跳台阶**

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
面试推荐型，自底向上型循环求解，时间复杂度为 O(n)。

```java
public class Solution {
    public int JumpFloor(int target) {
        // f[1] = 1, f[0] = 1 (f[0]是为了简便作答自己添加的)
        int a = 1, b = 1;
        for (int i = 2; i <= target; i++) {
            // 求f[i] = f[i - 1] + f[i - 2]
            a = a + b; // 这里求得的 f[i] 可以用于下次循环求 f[i+1]
            // f[i - 1] = f[i] - f[i - 2]
            b = a - b; // 这里求得的 f[i-1] 可以用于下次循环求 f[i+1]
        }
        return a;
    }
}
```

有小伙伴表示，方法二不太容易理解，这里做一下简单解释。其实就是自底向上求递推式的过程，这里再给出方法二原始的版本。

```java
public class Solution {
    public int JumpFloor(int target) {
        if (target <= 1) {
            return 1;
        }
        // a 表示第 f[i-2] 项，b 表示第 f[i-1] 项
        int a = 1, b = 1, c = 0;
        for (int i = 2; i <= target; i++) {
            c = a + b; // f[i] = f[i - 1] + f[i - 2];
            // 为下一次循环求 f[i + 1] 做准备
            a = b; // f[i - 2] = f[i - 1]
            b = c; // f[i - 1] = f[i]
        }
        return c;
    }
}
```

其实，方法二就是将这里的`if`条件判断和变量`c`优化掉了而已。







