# GO语言的坑

### 1. 严格的``nil``值

复现：

笔者刷 [leetcode#206.反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)时，将指针定义成：

~~~go
pre, curr := nil, head
~~~

然而严格的go编译器**在非unsafe模式下**不允许有**未定义类型**的``nil``指针

于是写成

~~~go
var pre *ListNode = nil
    curr := head
~~~

当然可以在编程时，将指针定义为``unsafe.Pointer``类型

引以为戒。

