# 算法
## 树
### 寻找最近公共父节点
#### 二叉搜索树 / 二叉排序树
二叉搜索树：在二叉树的基础上，节点排序存在限制，左子树节点的值小于根节点的值，右子树节点的值大于根节点的值
##### 存储路径，二次遍历
时间复杂度：O(n)

空间复杂度：O(n)，需要申请额外的数组空间

算法思路：

1、定义getPath函数，得到节点路径，将路径存储在数组中

```allykeynamelanguage
function getPath(root, target) {
    let result = [];
    while(root !== target) {
        result.push(root);
        if (target.val >= root.val) {
            root = root.right;
        } else {
            root = root.left;
        }
    }
    result.push(root);
    return result;
}
```

2、分别得到目标节点的路径，遍历路径找到分叉节点，该分叉节点就是最近公共节点

```allykeynamelanguage
var lowestCommonAncestor = function(root, p, q) {
    let pPath = getPath(root, p);
    let qPath = getPath(root, q);
    let result;
    for(let i = 0; i < qPath.length && i < pPath.length;i++) {
        if(pPath[i] === qPath[i]) {
            result = pPath[i];
        } else {
            break;
        }
    }
    return result;
};
```
##### 一次遍历
时间复杂度：O(n)

空间复杂度：O(1)

算法思路：寻找目标节点的位置，如果p，q节点大于跟节点，说明p，q节点在右子树；如果p，q节点小于跟节点，说明p，q节点在左子树。
最后根节点就是最近公共父节点

```allykeynamelanguage
var lowestCommonAncestor = function(root, p, q) {
    while(root !== null) {
        if (root.val < p.val && root.val < q.val) {
            root = root.right;
        } else if (root.val > p.val && root.val > q.val) {
            root = root.left;
        } else break;
    }
    return root;
}
```
##### 递归
时间复杂度：O(n)

空间复杂度：O(n)

算法思路：与一次遍历的思路类似

```allykeynamelanguage
var lowestCommonAncestor = function(root, p, q) {
    while(root !== null) {
        if (root.val < p.val && root.val < q.val) {
            return lowestCommonAncestor(root.right, p, q);
        } else if (root.val > p.val && root.val > q.val) {
            return lowestCommonAncestor(root.left, p, q);
        } else {
            return root;
        }
    }
}
```
#### 二叉树
二叉树：每个节点最多有两个子节点，节点排序没有限制

### 判断二叉搜索树是否存在两个节点值之和等于目标值k
遍历 + 哈希表

遍历的同时，将节点值塞入哈希表中并判断哈希表中是否存在k - val的值

## 四则运算
### 二进制加法
双指针模拟法

算法思路：将二进制逐位相加，满2进位

1、将两个字符串使用双指针从尾部开始计算，res保存每次的计算结果，flag表示是否进一。

2、长度不一致时我们可以补全，比如"111"与"10"相加时我们可以看成"111"与"010"相加，具体做法可以在指针小于0时取0来进行计算。

3、当双指针都小于0时，计算完毕，如果flag还存在，说明首位还需要进一，因此需要在res结果前面再补一个1。

```allykeynamelanguage
var addBinary = function(a, b) {
    let lenA = a.length - 1,lenB = b.length - 1;
    let res = '', carry = 0;
    while(lenA >= 0 || lenB >= 0) {
        let itemA = 1 * a[lenA--] || 0;
        let itemB = 1 * b[lenB--] || 0;
        const sum = itemA + itemB + carry;
        res = sum % 2 + res;
        carry = sum >> 1;
    }
    if (carry === 1) {
        return '1' + res;
    }
    return res;
};
```

    sum % 2：表示取sum的二进制表示的最低位，用于得到当前位想家的结果，取值为0或1
    sum >> 1：表示两个位的和超过1时，需要进位到高位，用于得到进位的值
### 不用+、-、*、/进行四则运算
#### 不用+进行整数加法运算
算法思路：将十进制的两位整数转成二进制，加法需要考虑两个方面，一个是逐位相加，一个是进位。

- 逐位相加：异或操作符^
- 进位：且运算符& 和 位运算符<<

```allykeynamelanguage
var add = function(a, b) {
    while (b != 0) {
        const carry = (a & b) << 1;
        a = a ^ b;
        b = carry;
    }
    return a;
};
```
#### 不用/进行整数除法运算

## 数组
### 递增数组求两湖之和等于目标数
#### 二分查找

#### 双指针

### 查找插入位置，要求时间复杂度log n
看到有序的数组，就要想到二分查找法，时间复杂度也对得上

### 左右子数组和相等，求中心下标记
#### 数学等式

### 求目标整数是否在递增有序的二维数组中
思路：抓住有序的前提，进行逐行逐列消除

时间复杂度：O(n + m)

### 数组中重复的数字
#### 哈希表
思路：利用哈希表的特点，将数组元素存储到哈希表中，可以判断是否存在重复的元素

空间复杂度：O(n) 时间复杂度：O(n)

#### 原地交换
思路：数组元素的索引和值是`一对一`的关系，当一个索引值对应两个元素，则说明这两个元素是重复的

如何将数组元素和索引对应上？

通过遍历数组进行交换操作，就能通过索引映射对应的值，起到与哈希表相同的效果

空间复杂度：O(1) 时间复杂度：O(n)

## 单链表
### 求两个链表的第一个公共节点
#### 双指针

### 反转链表
#### 双指针迭代

#### 整体法递归

### 求是否为回文链表
#### 转成求是否为回文数组

## 字符串
### 判断回文字符串
#### 双指针

### 最多删除一个元素，判断是否为回文字符串
#### 双指针
