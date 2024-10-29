# 1. 什么是CMake？为何要用CMake？编译器是怎么处理我们写好的程序的？

- 首先是为什么需要CMake或者Makefile这些东西？

  如果我们只写了一个或两个源文件，当然我们可以通过命令一个一个进行编译运行，但是大型项目中，可能存在上百上千个源文件，这时再通过这种一个一个编译运行的方式肯定不太现实！

  于是有一种解决方式，就是通过编写Makefile，指定一系列指令告诉编译器应该如何去编译这些文件，然后通过一个批处理指令make来进行项目构建

- CMake是一个项目构建工具，简单来说就是构建项目，实现编译；而比较常见的有Makefile（通过make命令来进行项目构建），大多数编译器是集成了make；而Makefile是很依赖当前的编译平台的，这就导致如何呢？你用不同的编译平台，就必须重新编写一套Makefile；而且当你去拉取别人的项目时，如果别人的编译平台与你不同，你也无法直接编译；再者，Makefile编写的工作量很大，依赖关系非常复杂！

- CMake干的就是一个省心，逻辑是：

  ​	编写CMakeLists.txt ---运行cmake命令---生成本地化的Makefile和工程文件---用户通过make命令项目构建---生成可执行文件

- 优点明显可知：

  - 可以跨平台，方便写
  - 能管理大型项目，能够用树状结构管理大型项目
  - 简化编译构建过程和编译过程
  - 可扩展

- 编译器 toolchain 工具链 分为四部分：

  - 预处理器：对源文件进行头文件展开、对应宏替换、注释去掉等操作 --- 得到的源文件
  - 些文件，然后通过一个批处理指令make来进行项目构建编译器：gcc或g++ --- 处理得到 汇编文件
  - 汇编处理 --- 成为二进制文件
  - 链接器 将这些二进制文件进行链接 --- 成为一个二进制文件 --- 成为可执行文件

# 2. 语法特性介绍

- 基本语法格式：指令（参数1 参数2 …）

  - 参数使用括弧括起
  - 参数之间使用空格或分号分开

- ##### 指令是大小写无关的，参数和变量是大小写相关的

- 变量使用`${}`方式取值，但是在`IF`控制语句中是直接使用变量名

  ```cmake
  if(HELLO) #正确
  if(${HELLO}) #错误
  ```

# 3.重要指令和CMake常用变量

- **指定`CMake`的最小版本要求:**

  语法：`cmake_minimum_required(VERSION versionNumber[FATAL_ERROR])`

  ```cmake
  # 我的Cmake的版本为3.16.3,最小版本要求可以设定为2.8.3
  cmake_minmum_required(VERSION 2.8.3)
  ```

  如果项目对`CMake`版本的设定比实际需要的版本高，那么可能会限制项目在其他环境中的兼容性或移植性。在实际的开源项目中，降低环境要求以增强项目的兼容性和可用性，是很常见的一个做法。

- **定义工程名称，并可指定工程支持的语言**

  语法：`project(projectname[CXX]\[C]\[JAVA])`

  ```cmake
  # 指定工程名为HELLOWORLD
  # 自定义
  project(HELLOWORLD)
  ```

- **定义变量**

  语法：`set(VAR[VALUE]\[CACHE TYPE DOCSTRING[FORCE]])`

  ```cmake
  # 定义SRC变量，其值为sayhello.cpp hello.cpp
  # 方式1：各个源文件之间使用空格间隔
  set(SRC sayhello.cpp hello.cpp)
  # 方式2：各个源文件使用分好间隔
  set(SRC sayhello.cpp;hello.cpp)
  ```

- **指定使用的C++标准**			

  语法：`set(CMAKE_CXX_STANDARD 11)`

  ```cmake
  # 指定使用c++ 11
  set(CMAKE_CXX_STANDARD 11)
  ```

- 指定输出的路径

  在CMake中指定可执行程序输出的路径，也对应一个宏 ： EXECUTABLE_OUTPUT_PATH ，它的值还是通过set命令来指定

  ```cmake
  # 定义一个变量用于存储一个绝对路径
  set(HOME /home/robin/Linux/Sort)
  # 将拼接好的路径值设置给EXECUTABLE_OUTPUT_PATH宏
  set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin)
  ```

  由于可执行程序是基于 cmake 命令生成的 makefile 文件然后再执行 make 命令得到的，所以如果此处指定可执行程序生成路径的时候使用的是相对路径 ./xxx/xxx，那么这个路径中的 ./ 对应的就是 makefile 文件所在的那个目录。

> 插入点逻辑让整个流程串起来！
>
> 前面有set指令可以实现将多个源文件指定为一个变量SRC，这样在这些操作中会有便利：
>
> ```cmake
> add_executable(main add.cpp div.cpp mult.cpp get_max.cpp get_min.cpp)
> ```
>
> ```cmake
> set(SRC add.cpp div.cpp mult.cpp get_max.cpp get_min.cpp)
> add_executable(main ${SRC})
> ```
>
> 但这其实没有解決本质的问题，因为你会发现set干的事不过只是将几个.c或.cpp文件用一个变量去替代，本质上你在add_executable()中所填的，还是一个文件名，而真正要实现的应该是：不指定文件的名字，而是使用一个指令，这个指令直接帮我们去搜索某个目录下的文件，cmake中这样的指令有两个：

- **搜索文件**

  如果一个项目里边的源文件很多，在编写CMakeLists.txt文件的时候不可能将项目目录的各个文件一一罗列出来，这样太麻烦也不现实。所以，在CMake中为我们提供了搜索文件的命令，可以使用``aux_source_directory``命令或者`file`命令。

  ```cmake
  # dir：要搜索的目录
  # variable：将从dir目录下搜索到的源文件列表存储到该变量中
  aux_source_directory(< dir > < variable >)
  ```

  ```cmake
  file(GLOB/GLOB_RECURSE 变量名 要搜索的文件路径和文件类型)
  ```

  ```cmake
  # 搜索当前目录下的src目录下所有的源文件，并存储到变量中
  file(GLOB MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
  file(GLOB MAIN_HEAD ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
  ```

  ``CMAKE_CURRENT_SOURCE_DIR ``宏表示当前访问的 CMakeLists.txt 文件所在的路径。

- **向工程添加多个特定的头文件搜索路径 —>相当于指定g++编译器的-I参数**

  语法：`include_directories([AFTER|BEFORE]\[SYSTEM] dir1 dir2 …)`

  ```cmake
  # 将/usr/include/myincludefolder和./include添加到头文件搜索路径
  include_directories(/usr/include/myincludefolder ./include)
  ```

  ```cmake
  include_directories(${PROJECT_SOURCE_DIR}/include)
  ```

  PROJECT_SOURCE_DIR宏对应的值就是我们在使用cmake命令时，后面紧跟的目录，一般是工程的根目录。
  
  `\#include "for_max.h"`
  
  `\#include "for_min.h"` 这样子写相当于`for_max.h` 和 `for_min.h `都是在 `"./ "`目录下的，即文件当前目录

> 同样加入一点逻辑，串起来
>
> 有些时候我们编写的源代码并不需要将他们编译生成可执行程序，而是生成一些静态库或动态库提供给第三方使用
>
> 在Linux中，静态库名字分为三部分：lib+库名字+.a，此处只需要指定出库的名字就可以了，另外两部分在生成该文件的时候会自动填充。
>
> 在Linux中，动态库名字分为三部分：lib+库名字+.so，此处只需要指定出库的名字就可以了，另外两部分在生成该文件的时候会自动填充。
>
> 那么也就可以谈到动态库和静态库的区别了：
>
> 动态库的链接和静态库是完全不同的：
>
> 静态库会在生成可执行程序的链接阶段被打包到可执行程序中，所以可执行程序启动，静态库就被加载到内存中了。
> 动态库在生成可执行程序的链接阶段不会被打包到可执行程序中，当可执行程序被启动并且调用了动态库中的函数的时候，动态库才会被加载到内存

- **生成库文件**（动态库或静态库）

  如果要生成静态库：语法为`add_library(calc STATIC ${SRC_LIST})`,注意关键字STATIC

  下面有一个目录，需要将src目录中的源文件编译成静态库

  ```shell
  .
  ├── build
  ├── CMakeLists.txt
  ├── include           # 头文件目录
  │   └── head.h
  ├── main.cpp          # 用于测试的源文件
  └── src               # 源文件目录
      ├── add.cpp
      ├── div.cpp
      ├── mult.cpp
      └── sub.cpp
  ```

  ```cmake
  cmake_minimum_required(VERSION 3.0)
  project(CALC)
  include_directories(${PROJECT_SOURCE_DIR}/include)
  file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
  add_library(calc STATIC ${SRC_LIST})
  ```

  这样最终就会生成对应的静态库文件libcalc.a。

  如果要生成动态库，语法为`add_library(库名称 SHARED 源文件1 [源文件2] ...) `,注意关键字`SHARED`

  ```cmake
  cmake_minimum_required(VERSION 3.0)
  project(CALC)
  include_directories(${PROJECT_SOURCE_DIR}/include)
  file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
  add_library(calc SHARED ${SRC_LIST})
  ```

  这样最终就会生成对应的动态库文件libcalc.so。

同样咯，生成库文件，同样可以指定库的输出地址

- 指定库文件输出地址

  对于生成的库文件来说和可执行程序一样都可以指定输出路径。由于在Linux下生成的动态库默认是有执行权限的，所以可以按照生成可执行程序的方式去指定它生成的目录：

  ```cmake
  # 设置动态库生成路径
  set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
  add_library(calc SHARED ${SRC_LIST})
  ```

  由于在Linux下生成的静态库默认不具有可执行权限，所以在指定静态库生成的路径的时候就不能使用`EXECUTABLE_OUTPUT_PATH`宏了，而应该使用`LIBRARY_OUTPUT_PATH`，这个宏对应静态库文件和动态库文件都适用。

  ```cmake
  cmake_minimum_required(VERSION 3.0)
  project(CALC)
  include_directories(${PROJECT_SOURCE_DIR}/include)
  file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
  # 设置动态库/静态库生成路径
  set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
  # 生成动态库
  # add_library(calc SHARED ${SRC_LIST})
  # 生成静态库
  add_library(calc STATIC ${SRC_LIST})
  ```

> 当制作出库文件之后，该如何在自己的项目中去应用它们呢？
>
> 需要在CMakeLists.txt中去包含库文件
>
> 制作并且指定路径产出之后，接着就是来验证我们所生成的库文件是否能使用；
>
> 要使用，需要先将库文件发布出去，在发布的时候，需要提供库文件（要么是静态库要么是动态库）及头文件，
>
> 为什么需要头文件，因为头文件里都是什么？都是我们在源文件（src）中定义的函数变量啥之类的声明；
>
> 如果没有声明，给到你一个库，你知道库里有什么东西吗？如果不知道，我们如何调用库里面的东西呢？库文件其实就是我们将若个源文件进行了打包操作，成为的一个二进制文件；如果将其转回文本模式，就是之前我们写的源文件了（src）
>
> 当我们已经将src中的文件生成了库文件，那么现在src中的源文件，对我们来说也就没有什么作用了

- **包含库文件**

  其实就是链接库文件

  前面所讲动态库和静态库文件的区别，也在链接这一步有所区别！

  在cmake中，链接静态库的命令如下：

  ```cmake
  link_libraries(<static lib> [<static lib>...])
  ```

  参数1：指定出要链接的静态库的名字

​					可以是全名 libxxx.a
​					也可以是掐头（lib）去尾（.a）之后的名字 xxx

​	   参数2-N：要链接的其它静态库的名字

​		<u>如果该静态库不是系统提供的（自己制作或者使用第三方提供的静态库）可能出现静态库找不到的情况，此时可以将静态库的路径也指定出来</u>

```cmake
link_directories(<lib path>)
```

完整的示例：

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
# 搜索指定目录下源文件
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
# 包含头文件路径
include_directories(${PROJECT_SOURCE_DIR}/include)
# 包含静态库路径
link_directories(${PROJECT_SOURCE_DIR}/lib)
# 链接静态库
link_libraries(calc)
add_executable(app ${SRC_LIST})
```

​		在cmake中链接动态库的命令如下:

```cmake
target_link_libraries(
    <target> 
    <PRIVATE|PUBLIC|INTERFACE> <item>... 
    [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
```

`target`：指定要加载动态库的文件的名字

该文件可能是一个源文件
该文件可能是一个动态库文件
该文件可能是一个可执行文件

**`PRIVATE|PUBLIC|INTERFACE`**：动态库的访问权限，默认为`PUBLIC`

如果各个动态库之间没有依赖关系，无需做任何设置，三者没有没有区别，一般无需指定，使用默认的 `PUBLIC `即可。
    
动态库的链接具有传递性，如果动态库 A 链接了动态库B、C，动态库D链接了动态库A，此时动态库D相当于也链接了动态库B、C，并可以使用动态库B、C中定义的方法。

`target_link_libraries(A B C)
target_link_libraries(D A)`

**`PUBLIC`**：在`public`后面的库会被Link到前面的`target`中，并且里面的符号也会被导出，提供给第三方使用。
**`PRIVATE`**：在`private`后面的库仅被link到前面的`target`中，并且终结掉，第三方不能感知你调了啥库
**`INTERFACE`**：在`interface`后面引入的库不会被链接到前面的`target`中，只会导出符号。

且由于动态库的特性，在cmake中指定要链接的动态库的时候，==应该将命令写到生成了可执行文件之后==

```cmake
cmake_minimum_required(VERSION 3.0)
project(TEST)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 添加并指定最终生成的可执行程序名
add_executable(app ${SRC_LIST})
# 指定可执行程序要链接的动态库名字
target_link_libraries(app pthread)
```

但是别忘了，可执行程序启动之后，去加载calc这个动态库，是不知道这个动态库被放到了什么位置的，所以会加载失败；

因此应该在 CMake 生成可执行程序之前，通过命令指定出要链接的动态库的位置！指定静态库位置使用的也是这个命令

```cmake
link_directories(path)
```

所以修改之后的CMakeLists.txt文件应该是这样的：

```cmake
cmake_minimum_required(VERSION 3.0)
project(TEST)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 指定源文件或者动态库对应的头文件路径
include_directories(${PROJECT_SOURCE_DIR}/include)
# 指定要链接的动态库的路径
link_directories(${PROJECT_SOURCE_DIR}/lib)
# 添加并生成一个可执行程序
add_executable(app ${SRC_LIST})
# 指定要链接的动态库
target_link_libraries(app pthread calc)
```

- **生成可执行文件**

  语法：`add_executable`

  ```cmake
  # 编译main.cpp生成可执行文件main
  add_executable(main main.cpp)
  ```

# 4. CMake中使用宏

- **常用宏定义**

​				**宏 **																					**功能**
**PROJECT_SOURCE_DIR **								使用cmake命令后紧跟的目录，一般是工程的根目录
PROJECT_BINARY_DIR 								执行cmake命令的目录
**CMAKE_CURRENT_SOURCE_DIR **				当前处理的CMakeLists.txt所在的路径
CMAKE_CURRENT_BINARY_DIR 				target 编译目录
EXECUTABLE_OUTPUT_PATH 					重新定义目标二进制可执行文件的存放位置
LIBRARY_OUTPUT_PATH 							重新定义目标链接库文件的存放位置
PROJECT_NAME 											返回通过PROJECT指令定义的项目名称
CMAKE_BINARY_DIR 									项目实际构建路径，假设在build目录进行的构建，那																			么得到的就是这个目录的路径

- **宏定义**

  > 在进行程序测试的时候，可以在代码中添加一些宏定义，通过这些宏来控制这些代码是否生效
  >
  > 应用背景：在项目开发中，前期测试肯定少不了大量的调试语句来判断程序是否正确，那么当程序真正落地，可以投入使用时，这些大量调试语句的存在肯定会影响程序运行的效率，此时，如果是一条条回溯去注释这些语句的话，是比较笨的做法，通常我们会使用这样一些宏开关，来直接控制这些调试语句是否生效

  ```cmake
  #include <stdio.h>
  #define NUMBER  3
  
  int main()
  {
      int a = 10;
  #ifdef DEBUG
      printf("我是一个程序猿, 我不会爬树...\n");
  #endif
      for(int i=0; i<NUMBER; ++i)
      {
          printf("hello, GCC!!!\n");
      }
      return 0;
  }
  ```

  在程序的第七行对DEBUG宏进行了判断，如果该宏被定义了，那么第八行就会进行日志输出，如果没有定义这个宏，第八行就相当于被注释掉了，因此最终无法看到日志输入出（上述代码中并没有定义这个宏）。

  为了让测试更灵活，我们可以不在代码中定义这个宏，而是在测试的时候去把它定义出来

  在CMake中我们也可以做类似的事情，对应的命令叫做`add_definitions`

  ```cmake
  cmake_minimum_required(VERSION 3.0)
  project(TEST)
  # 自定义 DEBUG 宏
  add_definitions(-DDEBUG)
  add_executable(app ./test.c)
  ```

  通过这种方式，上述代码中的第八行日志就能够被输出出来了。

# 5. CMake编译工程(必掌握)

### 5.1 CMake目录结构：项目主目录存在一个CMakeLists.txt文件

- 两种方式设置编译规则：

  - 包含源文件的子文件夹包含CMakeLists.txt文件，主目录的CMakeLists.txt通过add_subdirectory添加子目录即可；
  - 包含源文件的子文件夹未包含CMakeLists.txt文件，子目录编译规则体现在主目录的CMakeLists.txt中；

- 编译流程：

  在linux平台下使用CMake构建C/C++工程的流程如下：

  1. 手动编写CMakeLists.txt
  2. 执行命令`cmake PATH`生成Makefile（PATH是顶层CMakeLists.txt所在的目录）
  3. 执行命令`make`进行编译

- 外部构建方式框架：

  ```cmake
  ## 外部构建
  
  # 1.在当前目录下，创建build文件夹
  mkdir build
  # 2.进入到build文件夹
  cd build
  # 3.编译上级目录的CMakeLists.txt，生成Makefile和其他文件
  cmake ..
  # 4.执行make指令，生成target
  make
  ```

- 具体展示：

  将==编译输出文件==与==源文件==放到不同目录中:

  ```bash
  # 进入当前工作区
  # 创建include目录，放置头文件
  mkdir include
  # 分别创建两个文件夹src（放置源文件）和build（编译输出文件）
  mkdir src
  mkdir build
  # 在最外部创建main.c
  touch main.c
  # 同样在最外部创建CMakeLists.txt文件
  touch CMakeLists.txt
  ```

  编辑好CMakeLists.txt

  ```cmake
  # 指定所需要的最小CMake版本
  cmake_minimum_required(VERSION 3.0) 
  # 定义工程名字，可自定义
  project(SWAP)
  # 添加头文件搜索路径，记得之前include是用来放头文件的
  include_directories(include)
  # 生成指定编译文件
  add_executable(main_cmake main.cpp src/swap.cpp)
  # src/swap.cpp 是指进入到src目录下找到swap.cpp文件
  # main_cmake 是所生成的编译所需的二进制文件，同样也是可自定义，注意不要同名即可
  ```

  开始用终端搞事：

  ```bash
  # 进入到build目录下
  cmake .. # 在命令行中调用 CMake 构建系统的命令。
  # . 和 .. 分别表示当前目录和上级目录\#include "for_max.h"
  
  \#include "for_min.h" 这样子写相当于for_max.h 和 for_min.h 都是在 "./ "目录下的，即文件当前目录
  # 当你在命令行中输入 cmake .. 时，你正在告诉 CMake 在上一级目录下寻找 CMakeLists.txt 文件并根   据那个文件的指令进行构建
  # 然后就可以看到build目录下多了很多文件
  # 接着继续在build目录下，在终端中输入
  make # 这样就会生成一个名为main_cmake的可执行文件
  # 接着就可以通过执行这个可执行文件，来进行编译了
  ./main_cmake # 编译文件
  ```

### 5.2 CMake嵌套：树状管理大型项目

- 如果项目很大，或者项目中有很多的源码目录，在通过CMake管理项目的时候如果只使用一个CMakeLists.txt，那么这个文件相对会比较复杂，有一种化繁为简的方式就是给每个源码目录都添加一个CMakeLists.txt文件（头文件目录不需要），这样每个文件都不会太复杂，而且更灵活，更容易维护。

- ```shell
  $ tree
  .
  ├── build
  ├── calc
  │   ├── add.cpp
  │   ├── CMakeLists.txt
  │   ├── div.cpp
  │   ├── mult.cpp
  │   └── sub.cpp
  ├── CMakeLists.txt
  ├── include
  │   ├── calc.h
  │   └── sort.h
  ├── sort
  │   ├── CMakeLists.txt
  │   ├── insert.cpp
  │   └── select.cpp
  ├── test1
  │   ├── calc.cpp
  │   └── CMakeLists.txt
  └── test2
      ├── CMakeLists.txt
      └── sort.cpp
  
  6 directories, 15 files
  ```

- **节点关系**：

  众所周知，Linux的目录是树状结构，所以嵌套的 CMake 也是一个树状结构，最顶层的 CMakeLists.txt 是根节点，其次都是子节点。因此，我们需要了解一些关于 CMakeLists.txt 文件变量作用域的一些信息：

  根节点CMakeLists.txt中的变量全局有效
  父节点CMakeLists.txt中的变量可以在子节点中使用
  子节点CMakeLists.txt中的变量只能在当前节点中使用
  
- **添加子目录：**

  ```cmake
  add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
  ```

  source_dir：指定了CMakeLists.txt源文件和代码文件的位置，其实就是指定子目录
  binary_dir：指定了输出文件的路径，一般不需要指定，忽略即可。
  EXCLUDE_FROM_ALL：在子路径下的目标默认不会被包含到父路径的ALL目标里，并且也会被排除在IDE工程文件之外。用户必须显式构建在子路径下的目标。

  通过这种方式CMakeLists.txt文件之间的父子关系就被构建出来了。

- **大致的目录结构：**

  bin:存放生成的可执行文件

  include：存放编写的头文件

  库1

  库2

  build：cmake生成文件所在地

  
