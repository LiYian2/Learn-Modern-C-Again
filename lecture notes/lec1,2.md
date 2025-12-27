# Lecture 1 & 2: Introduction to C++
## Lec 1: Nothing important, just setup
## Lec 2: Basics of C++
### 1. Number types
- `int`, `float`, `double`, `char`, etc.
- Type modifiers: `short`, `long`, `unsigned`, `signed`
- Type aliases: `using` or `typedef`
- Acutually, the size of these types can **vary by platform**, but generally:
  - `int`: at least 16 bits
  - `float`: typically 32 bits (single precision)
  - `double`: typically 64 bits (double precision)
- `char`: typically 8 bits (1 byte)
- `bool`: typically 1 byte (true/false)
## a. Overflow & Underflow
- Integer overflow: when a calculation exceeds the maximum value of the type, it wraps around (undefined behavior for signed types). 
```cpp
// Example of integer overflow in addition
int a = INT_MAX;
int b = a + 1; // Undefined behavior for signed int
unsigned int c = UINT_MAX;
unsigned int d = c + 1; // Wraps around to 0

// Example of integer overflow in multiplication
int x = 100000;
int y = 30000;
int z = x * y; // May overflow and result in undefined behavior
long long z = x*y; // Still may overflow if x and y are too large as the type of intermediate result seleted is still int (the larger type of two operands, at least int)

int a = 1000000;
int b = 1000000;
long long c = a * b; // WRONG: Math is done as 'int' first, then assigned.

// Correct way:
long long c = static_cast<long long>(a) * b; // Cast one operand to long long to ensure the multiplication is done in long long
long long c = a * 1LL * b; // Using a long long literal to promote the operation
```
- Floating-point overflow: results in `infinity`.
- Underflow: when a number is too small to be represented, it may become zero or denormalized.

|Scenario | Rule | Result Type|
| ----------- | ----------- | ----------- |
|int * int| Both are the same.|int (Overflows if result > 2.1 billion)|
|int * long long|long long is higher rank.|long long
|short * short|"Both are ""smaller"" than int."|int (This is called Integral Promotion)
|float * int|Floating point ranks higher.|float
### 2. Variables & Constants
- Variables: Skip.
- Constants: Use `const` or `constexpr` for values that should not change.
#### 1. 在变量上的差别
    - `const` indicates that a variable's value cannot be changed after initialization.
    ```cpp
    int x;
    std::cin >> x;           // 运行期输入
    const int y = x;         // 正确：y 是只读的，但它的值在编译时是未知的
    // y = 10;               // 错误：不可修改
    ```
    - `constexpr` indicates that a value can be evaluated at compile time.
    ```cpp
    constexpr int size = 10; // 正确：编译期已知
    int arr[size];           // 正确：数组长度需要编译期常量

    int x;
    std::cin >> x;
    // constexpr int z = x;  // 错误：x 的值在编译期未知，无法初始化 constexpr
    ``` 
#### 2. 在函数上的差别
- `const` function parameters indicate that the function will not modify the argument.通常指类成员函数，承诺不修改对象的状态

    ```cpp
    void foo(const int& x) {
        // x = 10;           // 错误：不可修改
    }
    ```
- `constexpr` functions can be evaluated at compile time if given constant expressions.这种函数如果传入的参数是编译期常量，那么它本身可以在编译期计算出结果。
    ```cpp
    constexpr int square(int x) {
        return x * x;
    }
    int main() {
    // 1. 在编译期计算
    constexpr int res = square(10); 

    // 2. 也可以像普通函数一样在运行期使用
    int x;
    std::cin >> x;
    int res2 = square(x); 
    }
    ```
## Lec 3: Control Flow (Skipped)
## Lec 4: Vector, Array, Normal Pointer, References
### 1. Vector:
- Declare: `std::vector<type> vecName;` or `std::vector<type> vecName(size);` or `std::vector<type> vecName = {val1, val2, ...};` or
`std::vector<type> vecName(size, initialValue);` or
`std::vector<type> vecName(OriginalVector);` (copy constructor)
- Access: `vecName[index]` or `vecName.at(index)` (with bounds checking)
- Size: `vecName.size()`
- Add element at end: `vecName.push_back(value);`
- Remove last element: `vecName.pop_back();`
- Empty check: `vecName.empty();`
- Clear all elements: `vecName.clear();`
- Iteration: using range-based for loop or iterators

### 2. Reference (Alias):
- Declaration: `type& refName = originalVariable;`
- Usage: Use `refName` just like `originalVariable`.
- Characteristics:
    - Must be initialized when declared.
    - Cannot be null.
    - Acts as an alias to the original variable.
- Example:
    ```cpp
    int a = 10;
    int& refA = a; // refA is a reference to a
    refA += 5;     // Modifies a
    std::cout << a; // Outputs 15
    ```
- **Reference as function parameters**:
    - Allows functions to modify the original argument.
    - Avoids copying large objects for efficiency.
    ```cpp
    void modify(int& x) {
        x += 10; // Modifies the original variable
    }
    ```

### 3. Pointer:
- Declaration: `type* ptrName = &variable;` or `type* ptrName = nullptr;`
- Usage: Dereference using `*ptrName` to access or modify the value.
- Characteristics:
    - Can be null.
    - Can be reassigned to point to different variables.
    - Can be **arithmetic** (increment/decrement to point to next/previous memory locations).
    - Can be **dangerous** if not handled properly (dangling pointers, memory leaks).
- Example:
    ```cpp
    int a = 10;
    int* ptrA = &a; // ptrA points to a
    *ptrA += 5;     // Modifies a
    std::cout << a; // Outputs 15
    ```
- **Pointer as function parameters**:
    - Allows functions to modify the original argument.
    - Can be null, so always check before dereferencing.
    ```cpp
    void modify(int* x) {
        if (x != nullptr) {
            *x += 10; // Modifies the original variable
        }
    }
    ```
### 4. Array:
- Declaration: `type arrName[size];` or `type arrName[] = {val1, val2, ...};`
- Access: `arrName[index]`
- Size: Use `sizeof(arrName) / sizeof(arrName[0])` for static arrays.
- Characteristics:
    - Fixed size (cannot be resized).
    - Decays to pointer when passed to functions.
    - The arrayName itself acts like a pointer to the first element and therefore loses size information when passed to functions.
- Example:
    ```cpp
    int arr[5] = {1, 2, 3, 4, 5};

    for (int i = 0; i < 5; ++i) {
        std::cout << arr[i] << " "; // Outputs: 1 2 3 4 5
    }

    // Or use pointer arithmetic
    for (int* p = arr; p < arr + 5; ++p) { // p points to each element. ++p moves to the next element by adding sizeof(int)
        std::cout << *p << " "; // Outputs: 1 2 3 4 5  
    }

    // Function that takes array as parameter
    void printArray(int* arr, int size) { // arr decays to pointer and size must be passed separately
        for (int i = 0; i < size; ++i) {
            std::cout << arr[i] << " ";
        }

    // Array as function Return Value

    // 1. Using static array with new keyword (not recommended for modern C++) 需要后期手动释放内存，也有可能引起内存泄漏的问题
    int* createArray(int size) {
        int* arr = new int[size]; // Dynamically allocate array
        // Initialize array...
        return arr; // Return pointer to array
    }

    // 2. 将返回array作为参数传入，实际上不可用。因为数组在传参时会退化为指针，无法传递大小信息。
    void copy(const int source[], int destination[], int size);

    // 3. Using std::vector (recommended)
    std::vector<int> createVector(int size) {
        std::vector<int> vec(size); // Create vector of given size
        // Initialize vector...
        return vec; // 安全返回，利用移动语义（Move Semantics）效率极高，当然也可以用拷贝
    }
    ```
    - Character arrays (C-style strings):
    ```cpp
    char str[] = "Hello"; // Automatically adds null terminator '\0'
    std::cout << str; // Outputs: Hello
    ```
    - Important usage note:
      - Always ensure character arrays are null-terminated when used as strings. It’s important to not forget the space for the zero terminator. It’s helpful to declare character arrays with an “extra space” for the zero terminator
    - Note: Prefer `std::string` over character arrays for string manipulation in modern C++.

- When passing a **multi-dimensional array** to a function, you must have bounds for all dimensions except the first.
    ```cpp
    void processMatrix(int matrix[][COLS], int rows); // COLS must be known
    ```