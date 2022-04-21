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

