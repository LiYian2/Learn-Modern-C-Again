## 1. 核心语法

### 1. Value Semantics（值语义）
+ Values are independent objects by default
+ Directly passing arguments creates a copy
+ ```cpp
  // x is a copy, modifying it doesn't affect the original
  void foo(int x) { x += 1; }  // Original remains unchanged
  ```

### 2. Reference（引用）
+ ```cpp
  // &T passes a reference (alias), allowing modification of the original
  void foo(int& x) { x += 1; }  // Modifies the original value
  
  // Use for iteration when modification is needed
  for (auto& val : myVec) { val *= 2; }
  ```

### 3. Constant（常量）
+ `const` is a promise not to modify, not that the value can't change elsewhere
+ ```cpp
  // Common pattern for read-only arguments (avoids copy)
  void foo(const int& x) { /* x cannot be modified here */ }
  // Note: x may still be modified outside this function's scope
  
  // Read-only iteration (efficient, no copies)
  for (const auto& val : myVec) { cout << val << " "; }
  ```

### 4. Pointer（指针）
+ Prefer references over pointers when possible (no null-check needed, cleaner syntax)
+ ```cpp
  void foo(int* x) { 
      if (x != nullptr) {  // Always check for nullptr
          *x += 1; 
      }
  }
  ```

### 5. Implementations in reality
+ ```cpp
  // 1. 输入参数（只读）
  void process_data(const std::vector<int>& data);  // ✓ 推荐
  
  // 2. 输出参数（需要修改）
  void get_results(std::vector<int>& results);      // ✓ 推荐
  
  // 3. 可选输出参数（可能为null）
  
  bool try_parse(const std::string& str, int* out); // 指针用于可选输出
  ```

### 6. Object Lifetime & RAII
+ The lifetime of an object ends with the scope. Destruction happens automatically in reverse order of construction (LIFO).
+ **RAII (Resource Acquisition Is Initialization)**: Resource management is tied to object lifetime.
  - Resources are acquired in constructors
  - Resources are released in destructors
  - Resources include: file handles, memory, locks, threads, GPU resources, etc.
+ **Why C++ doesn't need GC?** Object lifetimes are deterministic and predictable based on scope. Destructors are guaranteed to be called when leaving scope (even with exceptions, returns, or normal flow).

```cpp
class FileHandler {
private:
    FILE* file;
public:
    FileHandler(const char* filename) : file(fopen(filename, "r")) {
        if (!file) throw std::runtime_error("File open failed");
    }
    
    ~FileHandler() {
        if (file) fclose(file);  // Resource released automatically
    }
    
    // Disable copy to prevent double close
    FileHandler(const FileHandler&) = delete;
    FileHandler& operator=(const FileHandler&) = delete;
};
```

```cpp
// ❌ Avoid raw new/delete
int* p = new int(5);
int* arr = new int[10];
// ... risk of forgetting delete, or exception before delete

delete p;           // For single objects
delete[] arr;       // For arrays
```

```cpp
// ✅ Recommended: Smart pointers
#include <memory>

// Single object
auto p = std::make_unique<int>(5);  // unique_ptr

// Array
auto arr = std::make_unique<int[]>(10);  // C++20

// Member access
obj.foo();           // Direct object
ptr_obj->foo();      // Smart pointer (same as raw pointer)

// 'this' pointer
class MyClass {
    int value;
public:
    MyClass(int val) : value(val) {}  // Prefer initialization list
    
    MyClass& increment() {
        value++;
        return *this;  // Return reference for chaining
    }
    
    void print() const {
        std::cout << "Value: " << value << std::endl;
    }
};
```

### 7. Smart Pointers Overview

#### 1. `std::unique_ptr<T>` (独占所有权)
- **特点**: 唯一所有者，不可复制，可移动
- **用途**: 明确单一所有权，替代 `new/delete`
- **内存**: 零开销（与原始指针相同）
- **示例**:
    ```cpp
    #include <memory>

    // 创建
    auto ptr1 = std::make_unique<int>(42);
    auto ptr2 = std::make_unique<std::string>("Hello");

    // 移动所有权（不可复制）
    auto ptr3 = std::move(ptr1);  // ptr1 现在为 nullptr

    // 自定义删除器
    auto file_deleter = [](FILE* f) { if (f) fclose(f); };
    std::unique_ptr<FILE, decltype(file_deleter)> filePtr(fopen("test.txt", "r"), file_deleter);
    ```

#### 2. `std::shared_ptr<T>` (共享所有权)
+ **特点**: 引用计数，多个指针共享同一对象
+ **用途**: 需要共享所有权的场景
+ **开销**: 稍高（需要维护引用计数）
+ **注意**: 避免循环引用（使用 `weak_ptr` 打破）
+ **示例**:
    ```cpp
    auto sp1 = std::make_shared<int>(100);  // 引用计数 = 1
    {
        auto sp2 = sp1;  // 引用计数 = 2（共享所有权）
        std::cout << *sp2 << std::endl;
    }  // sp2 销毁，引用计数 = 1

    // 循环引用问题
    struct Node {
        std::shared_ptr<Node> next;
        // std::weak_ptr<Node> prev;  // 正确：使用 weak_ptr 避免循环引用
    };

    auto node1 = std::make_shared<Node>();
    auto node2 = std::make_shared<Node>();
    node1->next = node2;
    node2->next = node1;  // 循环引用！内存泄漏
    ```

#### 3. `std::weak_ptr<T>` (弱引用)
- **特点**: 不增加引用计数，观察 `shared_ptr` 但不拥有对象
- **用途**: 打破循环引用，缓存，观察者模式
- **示例**:
    ```cpp
    auto shared = std::make_shared<int>(50);
    std::weak_ptr<int> weak = shared;  // 不增加引用计数

    if (auto locked = weak.lock()) {  // 尝试获取 shared_ptr
        std::cout << *locked << std::endl;  // 对象还存在
    } else {
        std::cout << "Object expired" << std::endl;
    }
    ```

#### 4. `std::auto_ptr<T>` (已弃用)
- **注意**: C++98 引入，C++17 移除，存在缺陷，**不要使用**
- **替代**: 使用 `std::unique_ptr`

### 智能指针使用指南
1. **默认使用 `std::make_unique`** - 单一所有权
2. **需要共享时用 `std::make_shared`** - 共享所有权
3. **需要观察时用 `std::weak_ptr`** - 避免循环引用
4. **永远不用 `std::auto_ptr`** - 已废弃
5. **尽量不用 `new/delete`** - 使用智能指针

### 实际示例
```cpp
#include <iostream>
#include <memory>
#include <vector>

class Resource {
public:
    Resource() { std::cout << "Resource acquired\n"; }
    ~Resource() { std::cout << "Resource released\n"; }
    void use() { std::cout << "Using resource\n"; }
};

int main() {
    // 1. 独占资源
    auto res1 = std::make_unique<Resource>();
    res1->use();
    
    // 2. 共享资源
    auto shared_res = std::make_shared<Resource>();
    {
        auto another_owner = shared_res;  // 共享所有权
        another_owner->use();
    }  // another_owner 销毁，但资源还在
    
    // 3. 容器中的智能指针
    std::vector<std::unique_ptr<Resource>> resources;
    resources.push_back(std::make_unique<Resource>());
    resources.push_back(std::make_unique<Resource>());
    
    return 0;  // 所有资源自动释放
}
```