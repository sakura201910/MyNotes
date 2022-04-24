### STL六大部件（components）

1. 容器(Containers)
2. 分配器(Allocators)
3. 算法(Algorithms)
4. 迭代器(Iterators)
5. 适配器(Adapters)
6. 仿函数(Functors)

![image-20220421094836857](https://cdn.konyue.site/image-20220421094836857.png)

### G4.9编译器的一个分配器

正常的分配器对于空间的申请，会默认添加附加的信息，而对于小空间，可能附加信息的比例甚至大于空间本身

对于容器中连续的小空间。这个分配器维护了一个内存池，只需要加一次附属信息

free_list上的每一个结点维护从8到128大小的内存空间链表

 ```cpp
 std::vector<int, __gnu_cxx::__pool_alloc<int> > vec;
 ```

![image-20220421161706954](https://cdn.konyue.site/image-20220421161706954.png)

### List

​    双向链表，带有prev，next指针，实际内存空间中是环形

#### 迭代器：

是一个类

![image-20220421232131581](https://cdn.konyue.site/image-20220421232131581.png)

### Vector

start、finish、end_of_storage三个指针,capacity(容量),size(使用的大小)

#### 2倍增长

push_back会判断是否还有空间，没有空间就会去扩容

空间为0时，会分配1

#### 拷贝情况：

1. 把原本的内容拷贝到新的内存中
2. 释放原vector
3. 调整迭代器指向新的vector

#### 迭代器

是个指针









































