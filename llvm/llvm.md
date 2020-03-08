#### 环境准备 ---- 下载源码，编译属于自己的编译器
1. 下载 llvm :`$ git clone https://git.llvm.org/git/llvm.git`
2. 在 `llvm/tools/` 目录下载 clang，

	```
	$ cd llvm/tools
	$ git clone https://git.llvm.org/git/clang.git`
	```
3. 安装 cmake 和 ninja（可以加快编译 clang 的速度）

	```
	$ brew install cmake
	$ brew install ninja
	```
	如果用 brew 直接安装失败的话，可以 `/usr/local/bin`
4. 在 llvm 源码的同级目录下新建一个 `llvm_build` 和 `llvm_release` 目录，用于存放 Ninja 模板和编译后的文件
5. 在 `llvm_build` 目录将 llvm 源码变成 Ninja 的模板
	
	```
	$ cd llvm_build
	$ cmake -G Ninja ../llvm -DCMAKE_INSTALL_PREFIX=../llvm_release
	```
	成功的标志是该目录了下有一个 `build.ninja` 文件
6. 在 `llvm_build` 目录下执行指令

	```
	$ ninja
	```
	开始编译 llvm 源码
7. 编译完成之后还是有很多东西混在一起的。执行以下命令将二进制工具提取到 `llvm_release` 目录中：

	```·
	$ ninja install
	```
	
#### 应用与实践
1. 语法树分析、语言转换等  
	官方参考：https://clang.llvm.org/docs/Tooling.htlm
2. Clang 插件开发
	- https://clang.llvm.org/docs/ClangPlugins.html
	- https://clang.llvm.org/docs/ExternalClangExamples.html
	- https://clang.llvm.org/docs/RAVFrontendAction.html

#### 编写插件
1. 在路径 `llvm/tools/clang/tools` 下新建一个目录，比如 `my-plugin`，用以开发插件
2. 打开 `llvm/tools/clang/tools/CMakeLists.txt`， 在最下边添加 `add_clang_subdirectory(my-plugin)`
3. 创建 .cpp 文件

	```
	$ cd llvm/tools/clang/tools/my-plugin
	$ touch MYPlugin.cpp
	```
4. 创建 `llvm/tools/clang/tools/my-plugin/CMakeLists.txt`，用于说明我们需要加载哪些 c++ 代码。在里面写入：

	```
	add_llvm_loadable_module(MyPlugin MyPlugin.cpp)
	```
5. 创建 Xcode 模板，方便编写 c++ 代码
	
	```
	$ mkdir llvm_xcode #在 llvm 源码的同级目下创建 llvm_xcode 目录
	$ cmake -G Xcode ../llvm
	```

