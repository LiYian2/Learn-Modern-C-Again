# 0. 基础命令
```bash
# 配置阶段：生成构建系统文件
# -S . : 指定源目录为当前目录（CMakeLists.txt所在目录）
# -B build : 指定构建目录为build（会在当前目录下创建build文件夹）
cmake -S . -B build

# 构建阶段：编译和链接
# --build build : 在build目录中执行构建
cmake --build build

# 运行生成的可执行文件
./build/hello
```

# 1. 配置常用参数
	```bash
	# 指定生成器（Generator）
	cmake -S . -B build -G "Unix Makefiles"  # Unix/Linux默认
	cmake -S . -B build -G "Ninja"           # 使用Ninja（更快）
	cmake -S . -B build -G "Visual Studio 17 2022"  # Windows
	
	# 指定构建类型
	cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug    # 调试版本
	cmake -S . -B build -DCMAKE_BUILD_TYPE=Release  # 发布版本
	cmake -S . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo  # 带调试信息的发布版
	
	# 设置编译器
	cmake -S . -B build -DCMAKE_CXX_COMPILER=g++-12
	cmake -S . -B build -DCMAKE_C_COMPILER=gcc-12
	
	# 设置安装前缀
	cmake -S . -B build -DCMAKE_INSTALL_PREFIX=/usr/local
	
	# 传递自定义变量
	cmake -S . -B build -DMY_CUSTOM_VAR=ON
	```
# 2. 构建常用参数
	```bash
	# 指定并行编译（利用多核CPU）
	cmake --build build --parallel 4  # 使用4个线程
	cmake --build build -j 4          # 简写形式
	
	# 指定构建目标
	cmake --build build --target hello     # 只构建hello目标
	cmake --build build --target clean     # 清理构建
	cmake --build build --target install   # 安装
	
	# 显示详细输出
	cmake --build build --verbose     # 显示详细编译命令
	
	# 指定配置（多配置生成器如Visual Studio）
	cmake --build build --config Debug
	cmake --build build --config Release
	```
# 3. 其他常用命令
	```bash
	# 查看帮助
	cmake --help
	cmake --help-command add_executable  # 查看特定命令帮助
	
	# 生成编译数据库（供工具如clangd使用）
	cmake -S . -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
	
	# 清除构建目录（替代rm -rf build）
	cmake --build build --target clean
	```