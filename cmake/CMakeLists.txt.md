```cpp
# 指定CMake的最低版本要求为3.20
cmake_minimum_required(VERSION 3.20)

# 定义项目名称和使用的编程语言
# hello_cpp: 项目名称
# LANGUAGES CXX: 使用C++语言
project(hello_cpp LANGUAGES CXX)

# 设置C++标准为C++20
set(CMAKE_CXX_STANDARD 20)

# 要求必须支持指定的C++标准，如果不支持会报错
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 添加一个可执行目标
# hello: 生成的可执行文件名称
# main.cpp: 源文件
add_executable(hello main.cpp)
```
更多的扩展
```cpp
# 设置默认构建类型（如果命令行未指定）
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Debug")
endif()

# 添加编译器警告选项
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# 查找依赖包
find_package(Boost 1.70 REQUIRED COMPONENTS filesystem system)

# 包含子目录
add_subdirectory(src)
add_subdirectory(tests)

# 添加头文件目录
target_include_directories(hello PUBLIC include)

# 链接库
target_link_libraries(hello PUBLIC some_library)
```
