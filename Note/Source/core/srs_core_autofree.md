<link href="../../Style/note.css" rel="stylesheet"></link>
自动释放指针。

# srs_core_autofree
```cpp
/**
 * auto free the instance in the current scope, for instance, MyClass* ptr,
 * which is a ptr and this class will:
 *       1. free the ptr.
 *       2. set ptr to NULL.
 *
 * Usage:
 *       MyClass* po = new MyClass();
 *       // ...... use po
 *       SrsAutoFree(MyClass, po);
 *
 * Usage for array:
 *      MyClass** pa = new MyClass*[size];
 *      // ....... use pa
 *      SrsAutoFreeA(MyClass*, pa);
 *
 * @remark the MyClass can be basic type, for instance, SrsAutoFreeA(char, pstr),
 *      where the char* pstr = new char[size].
 */
#define SrsAutoFree(className, instance) \
impl__SrsAutoFree<className> _auto_free_##instance(&instance, false)
#define SrsAutoFreeA(className, instance) \
impl__SrsAutoFree<className> _auto_free_array_##instance(&instance, true)
template<class T>
class impl__SrsAutoFree
{
private:
    T** ptr;
    bool is_array;
public:
    /**
     * auto delete the ptr.
     */
    impl__SrsAutoFree(T** p, bool array) {
        ptr = p;
        is_array = array;
    }
    
    virtual ~impl__SrsAutoFree() {
        if (ptr == NULL || *ptr == NULL) {
            return;
        }
        
        if (is_array) {
            delete[] *ptr;
        } else {
            delete *ptr;
        }
        
        *ptr = NULL;
    }
};
```
通过宏定义，在使用`SrsAutoFree(className, instance)`时，使用类名和对象指针实例化一个模板类`impl__SrsAutoFree<className>`为`_auto_free_instance`，在离开作用域时通过析构函数自动释放该指针。  
其中##为宏内连接作用。
