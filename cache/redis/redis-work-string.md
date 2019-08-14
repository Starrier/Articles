---
title: redis-work-string
date: 2019-02-21 21:45:20
tags: redis
index_img: ../post_img/redis-work-string.jpeg
---

## Reis - Work - String

Redis 数据库里的每个键值对(KV) 都是有对象组成的，数据库键总是一个字符串对象，数据库的值可以是字符串对象，列表对象，哈希对象，集合对象，有序对象这五种对象的其中一种。Redis 底层数据结构的数据类型有：

 1. 简单字动态字符串（Simple Dynamic String）
 2. 链表
 3. 字典
 4. 跳跃表
 5. 整数集合
 6. 压缩列表
 7. 对象

String 操作：

``` redis
set x "redis"
get x;
redis
```

Redis 作为一种存储字符串的缓存结构，由 C 语言实现。在 C 语言中，字符串是通过字符数组实现的，即 char[]，而 Redis 对于字符串的实现是通过 SDS（Simple Dynamic String）实现的。

``` C
struct sdshdr{
    // SDS 所保存字符串的长度
    int length;
    // 记录 buf 数组未使用字节的数量
    int free;
    char[] buf;
}
```

示例：

```c
sdshdr
free 0  // 表示这个 SDS 没有分配任何未使用的空间
length 5 // 表示这个 SDS 保存了一个长度为 5 的字符串
buf --> |'R'|'e'|'d'|'i'|'s'|'\0'| // buf 数组中保存着 reids 的字符串，SDS 遵循 C 字符串以空字符串结尾的惯例，保存字符串的 1 字节空间不计算在 SDS 的 len 属性之中
```

free 不为 0 的情况：

```C
sdshdr
free 3 // 表示这个 SDS 分配了三个空闲的空间
length 5
buf -->|'R'|'e'|'d'|'i'|'s'| | | |
```

### SDS 与字符串的区别

#### 1.获取字符串长度

**C 字符串：**因为 C 语言并不记录自身的长度信息，所以获取一个 C 字符串的长度，程序必须遍历整个字符串，对遇到的，每个字符进行计数，直到遇到代表字符串结尾的空字符串为止，这个操作的复杂度为 O(n)。

**SDS:**与 C 语言不同的是，SDS 结构中的属性 length 记录了 SDS 本身的长度，所以获取一个 SDS 长度的复杂度为 O(1)。有人疑问那么 SDS 的 length 值是哪来的？这里的 length 值是 SDS API在设置和更新 SDS 时自动完成的。

**总结1：**通过使用 SDS 而不是 C 字符串，Redis 获取字符串长度的复杂度由 O(N) 降为 O(1)，这确保了字符串长度的获取的工作不会成为 Redis 的性能瓶颈。

#### 2. 缓存区溢出

**C字符串：**由于C自身不记录字符串的长度带来一个问题是容易造成缓冲区溢出（buffer overflow）。在<string.h>/strcat函数中，可以将一个字符串拼接到另外一个字符串的末尾。

```C
char *strcat(char *dest,const char *src)
```

理想状态下，用户在使用这个函数时，假定C为dest分配了足够多的内存，可以容纳src字符串中的所有内容，而一旦这个假定不成立，就会产生缓冲区溢出。举个例子，假定内存中有相邻的两个字符串s1，s2，如图：

    s1                        s2
     |                         |
...|'R'|'e'|'d'|'i'|'s'|'\0'||'g'|'o'|'o'|'d'|'\0'|...
1
2
3

如果执行strcat(s1," cluster");将Redis改为”Redis cluster“，但是粗心的却忘了在执行这句之前为s1分配足够的空间，那么在执行之后，s1的数据将会溢出到s2所在的空间，导致s2保存的内容意外的被修改。

**SDS:**与 C 语言不同的是，SDS 空间分配政策完全杜绝了发生缓冲区溢出的可能性：当 SDS API 需要对字符串进行修改时，首先会检查 SDS 的空间是否满足修改所需的要求，因为 SDS 自身有对字符串长度记录的属性 length 和空闲空间属性 free，可以借助这两个参数进行检查。SDS 会在执行动作之前判断 SDS 的空间大小，再去执行操作，如果空间不够的话，SDS API 会自动扩展空间。

#### 减少修改字符串时带来的内存重分配次数

**C 字符串：**因为 C 字符串不记录自身长度，每次增长或者缩短字符串长度时，程序都要对这个C字符串数组进行一次内存重新分配操作，不然容易造成内存益出。因为内存，分配设计复杂的算法，并且可能需要执行系统调用，所以它通常是一个比较耗时和耗能的操作。但是 Redis 作为缓存，追求速度，所以不能经常发生内存分配操作。

**SDS:**SDS 数组中的未使用空间字节数量由 SDS 的属性 free 记录，通过 free 记录，SDS 实现了空间预分配和惰性释放两种优化策略。

 1. 空间预分配

空间预分配用于优化 SDS 的字符串增长操作：当 SDS 的 API 对一个 SDS 进行修改，并且需要对 SDS 的空间进行扩展时，程序不仅会为 SDS 分配修改所需要的空间，而且还会为 SDS 分配额外的空间。额外的空间分配规则如下：

   1. 如果修改 SDS 之后，SDS 的长度小于 1MB，那么程序会给 SDS 分配和 length 一样大的额外空间，这是 SDS length 和 free 的值相等。举个例子，如果修改后的字符串长度为 13k，那么 SDS 的空间将会占据 13+13+1=27k（额外的一个字节用于保存空字符串）。
   2. 如果修改 SDS 之后，SDS 的长度大于 1MB，那么程序会给 SDS 分配额外的 1MB 空间，举个例子，比如修改后的 SDS 有30MB 的大小，那么程序会分配 1MB 的未使用空间，SDS 的 buf 数组实际大小将是 30MB+1MB+1byte。

 2. 惰性释放

惰性释放用于优化 SDS 的字符串缩短操作：当 SDS 的 API 要缩短 SDS 保存的字符串时，程序并不需要立即使用内存重分配策略来回收缩短后多出来的字节，而是使用 free 属性将这些字节记录起来，并等待使用。

C 语言字符串在进行字符串的扩充和收缩的时候，都会面临着内存空间的重新分配问题。

 1. 字符串拼接会产生字符串的内存空间的扩充，在拼接的过程中，原来的字符串的大小很可能小于拼接后的字符串的大小，那么这样的话，就会导致一旦忘记申请分配空间，就会导致内存的溢出。
 2. 字符串在进行收缩的时候，内存空间会相应的收缩，而如果在进行字符串的切割的时候，没有对内存的空间进行一个重新分配，那么这部分多出来的空间就成为了内存泄露。

　　举个例子：我们需要对下面的SDS进行拓展，则需要进行空间的拓展，这时候redis 会将SDS的长度修改为13字节，并且将未使用空间同样修改为1字节
　　　因为在上一次修改字符串的时候已经拓展了空间，再次进行修改字符串的时候会发现空间足够使用，因此无须进行空间拓展　　　通过这种预分配策略，SDS将连续增长N次字符串所需的内存重分配次数从必定N次降低为最多N次。

#### 惰性空间释放

我们在观察 SDS 的结构的时候可以看到里面的 free 属性，是用于记录空余空间的。我们除了在拓展字符串的时候会使用到 free 来进行记录空余空间以外，在对字符串进行收缩的时候，我们也可以使用 free 属性来进行记录剩余空间，这样做的好处就是避免下次对字符串进行再次修改的时候，需要对字符串的空间进行拓展。

然而，我们并不是说不能释放 SDS 中空余的空间，SDS 提供了相应的 API，让我们可以在有需要的时候，自行释放 SDS 的空余空间。

通过惰性空间释放，SDS 避免了缩短字符串时所需的内存重分配操作，并未将来可能有的增长操作提供了优化。

##### 二进制安全

C 字符串中的字符必须符合某种编码（比如ASCII），并且除了字符串末尾之外，字符串里面不能包含空字符串，否则最先被程序读入的空字符串将被误认为是字符串结尾。SDS API 都是二进制安全的，所有 SDS API 都会以处理二进制的方式来处理存放在 SDS buf 中的数据，数据写什么样，它被读取时就是什么样子。

##### 兼容部分C字符串函数

SDS 的 API 总会以 SDS 保存的数据的末尾设置为空字符串，并且在分配 SDS 空间时会多分配一个字节的空间来容纳空字符串，这是为了那些保存的数据可以重用一部分 `<string.h>` 库中的函数。

##### **总结**

| C 字符串| SDS |
| ---|---|
| C 字符串获取长度复杂度O(n) |SDS 获取字符串长度复杂度 O(1)|
| API 是不安全的，会出现缓存区溢出| API 是安全的的，不会出现缓存区溢出|
| 修改字符串长度 N 次，必然执行 N 次内存重分配| 修改字符串 N 次，执行最多 N 次内存重分配|
| 只保存文本数据| 可以保存文本数据或二进制数据|
| 可以使用 `<string.h>` 库中的函数| 可以使用一部分|