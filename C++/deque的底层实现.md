# deque的底层实现

## 存储结构

deque 容器存储数据的空间是由一段一段等长的连续空间构成，各段空间之间并不一定是连续的，可以位于在内存的不同区域。

为了管理这些连续空间，deque 容器用数组（数组名假设为 map）存储着各个连续空间的首地址。也就是说，map 数组中存储的都是**指针**，指向那些真正用来存储数据的各个连续空间

![image-20220523154844067](https://cdn.konyue.site/image-20220523154844067.png)

当 deque 容器需要在头部或尾部增加存储空间时，它会申请一段新的连续空间，同时在 map 数组的开头或结尾添加指向该空间的指针，由此该空间就串接到了 deque 容器的头部或尾部。

> 如果 map 数组满了怎么办？很简单，再申请一块更大的连续空间供 map 数组使用，将原有数据（很多指针）拷贝到新的 map 数组中，然后释放旧的空间。

## 迭代器底层实现

由于 deque 容器底层将序列中的元素分别存储到了不同段的连续空间中，因此要想实现迭代器的功能，必须先解决如下 2 个问题：

1. 迭代器在遍历 deque 容器时，必须能够确认各个连续空间在 map 数组中的位置；
2. 迭代器在遍历某个具体的连续空间时，必须能够判断自己是否已经处于空间的边缘位置。如果是，则一旦前进或者后退，就需要跳跃到上一个或者下一个连续空间中。

为了实现遍历 deque 容器的功能，deque 迭代器定义了如下的结构：

```cpp
template<class T,...>
struct __deque_iterator{
    ...
    T* cur;
    T* first;
    T* last;
    map_pointer node;//map_pointer 等价于 T**
}
```

可以看到，迭代器内部包含 4 个指针，它们各自的作用为：

- cur：指向当前正在遍历的元素；
- first：指向当前连续空间的首地址；
- last：指向当前连续空间的末尾地址；
- node：它是一个二级指针，用于指向 map 数组中存储的指向当前连续空间的指针。


借助这 4 个指针，deque 迭代器对随机访问迭代器支持的各种运算符进行了重载，能够对 deque 分段连续空间中存储的元素进行遍历。例如：

```cpp
//当迭代器处于当前连续空间边缘的位置时，如果继续遍历，就需要跳跃到其它的连续空间中，该函数可用来实现此功能
void set_node(map_pointer new_node){
    node = new_node;//记录新的连续空间在 map 数组中的位置
    first = *new_node; //更新 first 指针
    //更新 last 指针，difference_type(buffer_size())表示每段连续空间的长度
    last = first + difference_type(buffer_size());
}
//重载 * 运算符
reference operator*() const{return *cur;}
pointer operator->() const{return &(operator *());}
//重载前置 ++ 运算符
self & operator++(){
    ++cur;
    //处理 cur 处于连续空间边缘的特殊情况
    if(cur == last){
        //调用该函数，将迭代器跳跃到下一个连续空间中
        set_node(node+1);
        //对 cur 重新赋值
        cur = first;
    }
    return *this;
}
//重置前置 -- 运算符
self& operator--(){
    //如果 cur 位于连续空间边缘，则先将迭代器跳跃到前一个连续空间中
    if(cur == first){
        set_node(node-1);
        cur == last;
    }
    --cur;
    return *this;
}
```

## deque容器的底层实现

deque 容器除了维护先前讲过的 map 数组，还需要维护 start、finish 这 2 个 deque 迭代器。以下为 deque 容器的定义：

```cpp
//_Alloc为内存分配器
template<class _Ty,
    class _Alloc = allocator<_Ty>>
class deque{
    ...
protected:
    iterator start;
    iterator finish;
    map_pointer map;
...
}
```

其中，start 迭代器记录着 map 数组中首个连续空间的信息，finish 迭代器记录着 map 数组中最后一个连续空间的信息。另外需要注意的是，和普通 deque 迭代器不同，start 迭代器中的 cur 指针指向的是连续空间中首个元素；而 finish 迭代器中的 cur 指针指向的是连续空间最后一个元素的下一个位置。



借助 start 和 finish，以及 deque 迭代器中重载的诸多运算符，就可以实现 deque 容器提供的大部分成员函数，比如

```cpp
//begin() 成员函数
iterator begin() {return start;}
//end() 成员函数
iterator end() { return finish;}
//front() 成员函数
reference front(){return *start;}
//back() 成员函数
reference back(){
    iterator tmp = finish;
    --tmp;
    return *tmp;
}
//size() 成员函数
size_type size() const{return finish - start;}//deque迭代器重载了 - 运算符
//enpty() 成员函数
bool empty() const{return finish == start;}
```

[来源]([C++ STL deque容器底层实现原理（深度剖析） (biancheng.net)](http://c.biancheng.net/view/6908.html))





















