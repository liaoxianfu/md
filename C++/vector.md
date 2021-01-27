

## vector常用api

```c++
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

void printVector(vector<int> &v)
{
    for_each(v.begin(), v.end(), [](int tmp) {
        cout << tmp << " ";
    });
    cout << endl;
}

/******************************************************************************
 * 功能描述
 *     构造vector
 * 
 * 函数原型
 *       vector<T> vl                        默认构造函数
 *       vector<T>(v.begin(),v.end())        将v[begin(), end())区间中的元素
 *                                           拷贝给本身。
 *       vector<T>(n,elem)                   构造函数将n个elem拷贝给本身。
 *       vector<T>(const vector &vec)        拷贝构造函数
 ******************************************************************************/

void test01()
{
    vector<int> v;

    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }

    // 将v[begin(), end())区间中的元素
    vector<int> v2 = vector<int>(v.begin(), v.end());
    printVector(v2);
    // vector<T>(n,elem)            构造函数将n个elem拷贝给本身。
    vector<int> v3 = vector<int>(10, 1);
    printVector(v3);
    vector<int> v4 = vector<int>(v3);
    printVector(v4);
}

/******************************************************************************
 * 功能描述
 *     给vector容器进行赋值 
 * 
 * 函数原型
 *     vector& operator=(const vector &vec)    重载'='运算符
 *     assign(beg,end)                         将[beg,end)区间（左闭右开）中的数
 *                                             据拷贝赋值给本身
 *     assign(n,elem)                          将n个elem赋值给本身 
 ******************************************************************************/

void test02()
{
    vector<int> v;

    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    // vector& operator=(const vector &vec)    重载'='运算符
    vector<int> v2 = v;
    printVector(v2);
    // assign(beg,end)     将[beg,end)区间（左闭右开）中的数据拷贝赋值给本身
    vector<int> v3(v2.begin(), v2.end());
    printVector(v3);
    // assign(n,elem)                          将n个elem赋值给本身
    vector<int> v4 = vector<int>(10, 2);
    printVector(v4);
}

/******************************************************************************
 * 功能描述
 *     对vector容器的容量和大小操作
 * 
 * 函数原型
 *      empty()                 判断是否为空
 *      capacity()              容器的容量
 *      size()                  容器元素的数量
 *      resize(int num)         重新指定容器的长度，若容器变长就以默认值填充新位置
 *                              容器长度变短就删除超出的元素
 *      resize(int num,elem)    重新指定容器的长度，若容器变长就以elem值填充新位置
 *                              容器长度变短就删除超出的元素
 ******************************************************************************/

void test03()
{
    vector<int> v;
    cout << "v为是否空" << v.empty() << endl;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    cout << "v的容器大小为" << v.capacity() << endl;
    cout << "v的元素数量为" << v.size() << endl;
    //resize(int num) 超出容量
    v.resize(20);
    printVector(v);
    v.resize(30, 5);
    printVector(v);
    // 缩减容量
    v.reserve(2);
    printVector(v);
}

/******************************************************************************
 * 功能描述
 *     对vector容器进行插入删除操作
 * 
 * 函数原型
 *     push_back(elem)                              尾部插入元素elem
 *     pop_back()                                   删除最后一个元素
 *     insert(const_iterator pos, ele)              迭代器指向位置pos插入元素ele
 *     insert(const_iterator pos, int count,ele)    迭代器指向位置pos插入count个
 *                                                  元素ele
 *     erase(const_iterator pos)                    删除迭代器指向的元素
 *     erase(const_iterator start, 
 *                     const_iterator end);         删除迭代器从start到end之间
 *                                                  的元素
 *     clear()                                      删除容器中所有元素
 ******************************************************************************/
void test04()
{
    vector<int> v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    //  push_back(elem)  尾部插入元素elem
    v.push_back(10);
    printVector(v);
    //  pop_back()   删除最后一个元素
    v.pop_back();
    printVector(v);
    // insert(const_iterator pos, ele)  迭代器指向位置pos插入元素ele
    v.insert(v.begin(), 100);
    printVector(v);
    v.insert(v.begin(), 2, 99);
    printVector(v);
    v.erase(v.begin());
    printVector(v);
    v.erase(v.begin(), v.end());
    printVector(v);
    v.clear();
}

/******************************************************************************
 * 功能描述
 *     对vector中的数据进行存取操作
 * 函数原型
 *     at(int idx)      返回索引idx指向位置的数据
 *     operator[]       返回索引idx指向位置的数据
 *     front()          返回容器的第一个元素
 *     back()           返回容器的最后一个元素
 ******************************************************************************/
void test05()
{
    vector<int> v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    cout << v.at(0) << endl;
    cout << v[1] << endl;
    cout << v.front() << endl;
    cout << v.back() << endl;
}

/******************************************************************************
 * 功能描述
 *     实现两个容器内元素进行互换
 * 意义
 *     通过互换元素实现收缩内存
 * 函数原型
 *     swap(vector v)   将v与本身进行互换
 ******************************************************************************/
void test06()
{
    vector<int> v1;
    for (int i = 0; i < 10; i++)
    {
        v1.push_back(i);
    }

    vector<int> v2;
    for (int i = 10; i > 0; i--)
    {
        v2.push_back(i);
    }
    cout << "互换之前" << endl;
    printVector(v1);
    printVector(v2);
    cout << "互换之后" << endl;
    v1.swap(v2);
    printVector(v1);
    printVector(v2);

    int num = 0;
    int *p = NULL;
    vector<int> v3;
    for (int i = 0; i < 100000; i++)
    {
        v3.push_back(i);
    }
    cout << "v的容量为：" << v3.capacity() << endl;
    cout << "v的大小为：" << v3.size() << endl;

    v3.resize(3);

    cout << "v的容量为：" << v3.capacity() << endl;
    cout << "v的大小为：" << v3.size() << endl;

    // 收缩内存
    vector<int>(v3).swap(v3); //匿名对象

    cout << "v的容量为：" << v3.capacity() << endl;
    cout << "v的大小为：" << v3.size() << endl;
}

/******************************************************************************
 * 功能描述
 *     减少vector在动态扩展容量时的扩展次数
 * 
 * 函数原型
 *      reserve(int len)    容器预留len个元素长度，预留位置不初始化，元素不可访问。
 ******************************************************************************/

void num_test(vector<int> &v3)
{
    int num = 0;
    int *p = NULL;
    for (int i = 0; i < 100000; i++)
    {
        v3.push_back(i);
        if (p != &v3[0])
        {
            p = &v3[0];
            num++;
        }
    }
    printf("开辟了%d次内存空间\n", num);
}

void test07()
{

    vector<int> v3;
    num_test(v3);
    vector<int> v;
    v.reserve(100000);
    num_test(v);
}

/******************************************************************************
 * 函数测试
 ******************************************************************************/
int main()
{
    // test01();
    // test02();
    // test03();
    // test04();
    // test05();
    // test06();
    test07();
}
```

