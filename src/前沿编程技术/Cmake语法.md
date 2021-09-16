> 相关练习示例请移步：[https://github.com/BarretRen/CmakeUsage](https://github.com/BarretRen/CmakeUsage)
> 参考文档如下：


# 1 语法规则

1. 首行`cmake_minimum_required(VERSION 3.16)`
1. 变量使用${}方式取值，但是在 IF 控制语句中是直接使用变量名
1. 指令(参数 1 参数 2...)，参数使用括弧括起，参数之间使用空格或分号分开
1. 指令是大小写无关的，参数和变量是大小写相关的

# 2 CMake常用变量
CMake提供了很多有用的变量。以下仅列举常用的变量：

| **变量名** | **含义** |
| --- | --- |
| CMAKE_BINARY_DIR | 构建树的顶层路径 |
| CMAKE_COMMAND | 指向CMake可执行文件的完整路径 |
| CMAKE_CURRENT_BINARY_DIR | 当前正在被处理的二进制目录的路径 |
| CMAKE_CURRENT_SOURCE_DIR | 指向正在被处理的源码目录的路径 |
| CMAKE_HOME_DIRECTORY | 指向源码树顶层的路径 |
| CMAKE_PROJECT_NAME | 当前工程的工程名 |
| CMAKE_ROOT | CMake的安装路径 |
| CMAKE_SOURCE_DIR | 源码树的顶层路径 |
| CMAKE_VERSION | cmake的完整版本号 |
| PROJECT_BINARY_DIR | 指向当前编译工程构建的全路径 |
| PROJECT_SOURCE_DIR | 指向构建工程的全路径 |
| PROJECT_NAME | project命令传递的工程名参数 |
| <PROJECT-NAME>_VERSION | 项目的完整版本号 |
| CMAKE_CXX_STANDARD | C++编译器版本 |
| CMAKE_C_STANDARD | C编译器版本 |
| CMAKE_C_FLAGS | 设置C编译选项 |
| CMAKE_CXX_FLAGS | 设置C++编译选项 |
| EXECUTABLE_OUTPUT_PATH  | 编译后二进制文件的保存路径 |
| LIBRARY_OUTPUT_PATH  | 编译共享库的保存路径 |
| CMAKE_INSTALL_PREFIX   | 安装路径的前缀 |


# 3 CMake常用命令

- **cmake_minimum_required和project**：设置项目要求的CMake最低版本号 和 设置项目名称
```makefile
# 设置cmake 版本信息
cmake_minimum_required(VERSION 3.16)
# 设置 项目名称
project(demo)
```

- **add_library**：添加一个库文件
```makefile
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2 ...])
<name> : 表示库文件的名字
[STATIC | SHARED | MODULE] : 生成的库文件类型
STATIC 静态库，在链接其他目标时使用
SHARED 动态链接库，运行时加载
MODULE 不会被链接到其它目标中，但是可能会在运行时使用dlopen-系列的函数动态链接
例：
# create lib ， 在当前cmake 中 创建一个lib，名称为model_login，SHARED 动态库 ，后面为源文件列表
add_library(model_login SHARED
        lib_login_model/LoginService.cpp
        utils/common_utils.cpp)
add_library(lib_model_a model_utils.cpp)
// 设置别名
add_library(sub::modelA ALIAS lib_model_a)
```

- **add_subdirectory**：向构建中添加子目录(子目录中可以包含另一个cMakeLists.txt)。命令格式为
```makefile
add_subdirectory(source_dir [binary_dir]
                 [EXCLUDE_FROM_ALL])
source_dir 需要包含的子目录。
binary_dir 指定中间二进制和目标二进制存放的位置
EXCLUDE_FROM_ALL 编译过程中排除的文件
```

- **aux_source_directory**：查找目录中的所有源文件
```makefile
aux_source_directory(<dir> <variable>)
查找指定目录dir中所有源文件的名称，并将列表存储在提供的variable中
例：
aux_source_directory(. DIR_SRCS)
add_executable(${APP_NAME} ${DIR_SRCS})
```

- **target_link_directories**：指定链接器在链接给定目标时应在其中搜索库的路径
```makefile
target_link_directories(<target> [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])

<target>: add_library 创建的target名称 或者 add_executable 创建的target名称
target_link_libraries([TARGET_NAME] [链接库名字])  # 按名字添加
target_link_directories([链接库目录])  # 按目录添加
```

- **target_include_directories**：指定在编译给定目标时要使用的包含目录
```makefile
target_include_directories(<target> [SYSTEM] [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])

# 添加头文件的路径，以便查找到头文件
<target>: add_library 创建的target名称 或者 add_executable 创建的target名称
例：
target_include_directories(lib_model_a PUBLIC ${PROJECT_SOURCE_DIR})
```

- **find_library**：查找一个库文件。该命令用来查找一个库文件。一个名为<VAR>的cache条目会被创建来存储该命令的结果。如果找到了该库文件，那么结果会存储在该变量里，并且搜索过程将不再重复，除非该变量被清空。如果没有找到，结果变量将会是<VAR>-NOTFOUND，并且在下次使用相同变量调用find_library命令时，搜索过程会再次尝试。
```makefile
find_library(<VAR> name1 [path1 path2 ...])
例如：
link_directories（....） // 动态链接库或静态链接库的搜索路径
find_library(Foundation Foundation)
```

- **add_definitions**：为源文件的编译添加由-D引入的宏定义。命令格式为：
```makefile
add_definitions(-DFOO -DBAR ...)
```

- **add_dependencies**：使顶级目标依赖于其他顶级目标，以确保它们在该目标之前构建。这里的顶级目标是由add_executable，add_library或add_custom_target命令之一创建的目标
```makefile
add_library(model_login SHARED
        lib_login_model/LoginService.cpp
        utils/common_utils.cpp)
add_executable(demo main.cpp config.h.in Simple.cpp)
add_dependencies(demo model_login)
```

- **add_executable**：添加一个可执行文件
```makefile
# create exe
add_executable(demo main.cpp config.h.in Simple.cpp)
```

- **target_link_libraries**：将给定的库链接到一个目标上
```makefile
target_link_libraries(<target> ... <item>... ...)
例：
target_link_libraries(demo
        model_login)
```

- **include_directories**：将给定的目录添加到编译器用于搜索包含文件的目录。相对路径则相对于当前源目录。命令格式为：
```makefile
include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])
例：
// 指定头文件的搜索路径
include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/cocos
  ${CMAKE_CURRENT_SOURCE_DIR}/cocos/platform
  ${CMAKE_CURRENT_SOURCE_DIR}/extensions
  ${CMAKE_CURRENT_SOURCE_DIR}/external
)
```

- message：向用户显示消息。命令格式为：
```makefile
message([<mode>] "message to display" ...)
参数说明：
mode：
可选的值为none，STATUS，WARNING，AUTHOR_WARNING，SEND_ERROR，FATAL_ERROR，DEPRECATION。
例：
message(STATUS "This is BINARY dir " ${HELLO_BINARY_DIR})
```

- set：将一个CMAKE变量设置为给定值。命令格式为：
```makefile
set(<variable> <value>... [PARENT_SCOPE])
例：
set(COCOS2D_ROOT ${CMAKE_SOURCE_DIR}/cocos2d)
```

- **set(CMAKE_CXX_FLAGS)**：传递FLAGS给C++编译器:设置CMAKE_CXX_FLAGS变量
```makefile
set(CMAKE_CXX_COMPILER      "clang++" )         # 显示指定使用的C++编译器
set(CMAKE_CXX_FLAGS   "-std=c++11")             # c++11
set(CMAKE_CXX_FLAGS   "-g")                     # 调试信息
set(CMAKE_CXX_FLAGS   "-Wall")                  # 开启所有警告
set(CMAKE_CXX_FLAGS_DEBUG   "-O0" )             # 调试包不优化
set(CMAKE_CXX_FLAGS_RELEASE "-O2 -DNDEBUG " )   # release包优化
# CMAKE_CXX_FLAGS 是CMake传给C++编译器的编译选项，通过设置这个值就好比 g++ -std=c++11 -g -Wall
# CMAKE_CXX_FLAGS_DEBUG 是除了CMAKE_CXX_FLAGS外，在Debug配置下，额外的参数
# CMAKE_CXX_FLAGS_RELEASE 同理，是除了CMAKE_CXX_FLAGS外，在Release配置下，额外的参数
```

- configure_file：将文件复制到其他位置并修改其内容。命令格式为
```makefile
configure_file(<input> <output>
               [COPYONLY] [ESCAPE_QUOTES] [@ONLY]
               [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ])
例：
# 添加config 配置文件 版本信息 
set(demo_VERSION_MAJOR 1)
set(demo_VERSION_MINOR 0)
set(demo_VERSION_PATCH 10)
set(demo_VERSION_BUILD 100)
set(CONFIG_VERSION "${demo_VERSION_MAJOR}.${demo_VERSION_MINOR}.${demo_VERSION_PATCH}.${demo_VERSION_BUILD}")
configure_file(${PROJECT_SOURCE_DIR}/config.h.in ${PROJECT_BINARY_DIR}/config.h)
```
`config.h.in` 文件如下：
```makefile
#cmakedefine USE_MYMATH
#ifndef DEMO_CONFIG_H
#define DEMO_CONFIG_H
const char* version = "${CONFIG_VERSION}";
#endif //DEMO_CONFIG_H
```

- **file**：文件操作相关的命令。命令格式为：
```makefile
file(WRITE <filename> <content>...)
file(APPEND <filename> <content>...)
file(READ <filename> <variable>
     [OFFSET <offset>] [LIMIT <max-in>] [HEX])
file(STRINGS <filename> <variable> [<options>...])
file(<MD5|SHA1|SHA224|SHA256|SHA384|SHA512> <filename> <variable>)
file(GLOB <variable>
     [LIST_DIRECTORIES true|false] [RELATIVE <path>]
     [<globbing-expressions>...])
file(GLOB_RECURSE <variable> [FOLLOW_SYMLINKS]
     [LIST_DIRECTORIES true|false] [RELATIVE <path>]
     [<globbing-expressions>...])
file(RENAME <oldname> <newname>)
file(REMOVE [<files>...])
file(REMOVE_RECURSE [<files>...])
file(MAKE_DIRECTORY [<directories>...])
file(RELATIVE_PATH <variable> <directory> <file>)
file(TO_CMAKE_PATH "<path>" <variable>)
file(TO_NATIVE_PATH "<path>" <variable>)
file(DOWNLOAD <url> <file> [<options>...])
file(UPLOAD   <file> <url> [<options>...])
file(TIMESTAMP <filename> <variable> [<format>] [UTC])
file(GENERATE OUTPUT output-file
     <INPUT input-file|CONTENT content>
     [CONDITION expression])
file(<COPY|INSTALL> <files>... DESTINATION <dir>
     [FILE_PERMISSIONS <permissions>...]
     [DIRECTORY_PERMISSIONS <permissions>...]
     [NO_SOURCE_PERMISSIONS] [USE_SOURCE_PERMISSIONS]
     [FILES_MATCHING]
     [[PATTERN <pattern> | REGEX <regex>]
      [EXCLUDE] [PERMISSIONS <permissions>...]] [...])
file(LOCK <path> [DIRECTORY] [RELEASE]
     [GUARD <FUNCTION|FILE|PROCESS>]
     [RESULT_VARIABLE <variable>]
     [TIMEOUT <seconds>])
例：
# 查找src目录下所有以hello开头的文件并保存到SRC_FILES变量里
file(GLOB SRC_FILES "src/hello*")
# 递归查找
file(GLOB_RECURSE SRC_FILES "src/hello*")
```

- **install**：定义make install时的安装路径等配置
```makefile
install(TARGETS targets...	
	[
    	[ARCHIVE|LIBRARY|RUNTIME] [DESTINATION <dir>] # 静态库/动态库/可执行文件安装路径
    	[PERMISSIONS permissions...]
		[CONFIGURATIONS [Debug|Release|...]]
		[COMPONENT <component>]
		[OPTIONAL]
	] 
    [PUBLIC_HEADER DESTINATION include]  # 头文件安装路径
    [...])
```

- **set_target_properties**：设置目标的属性，比如名字、库版本号和API版本号等
```makefile
set_target_properties(target1 target2 ...
	PROPERTIES prop1 value1
	prop2 value2 ...)
```

# 4 流程控制语句

## 4.1 if语句
```makefile
IF(expression)
	# THEN section.
	COMMAND1(ARGS ...)COMMAND2(ARGS ...)
	...
ELSE(expression)
	# ELSE section.
	COMMAND1(ARGS ...)
	COMMAND2(ARGS ...)
	...
ENDIF(expression)
```
if条件判断格式：

- IF(NOT var )，与上述条件相反
- IF(var1 AND var2)，当两个变量都为真是为真
- IF(var1 OR var2)，当两个变量其中一个为真时为真
- IF(COMMAND cmd)，当给定的 cmd 确实是命令并可以调用是为真
- IF(EXISTS dir)或者 IF(EXISTS file)，当目录名或者文件名存在时为真。
- IF(string LESS number)
- IF(variable GREATER number)
- IF(variable EQUAL number)
- IF(string STRLESS string)
- IF(variable STRGREATER string)
- IF(string STREQUAL string)

## 4.2 while语句
```makefile
WHILE(condition)
	COMMAND1(ARGS ...)
	COMMAND2(ARGS ...)
	...
ENDWHILE(condition)
```

## 4.3 foreach语句
```makefile
FOREACH(loop_var arg1 arg2 ...)
    COMMAND1(ARGS ...)
    COMMAND2(ARGS ...)
    ...
ENDFOREACH(loop_var)
# 另一种格式范围
FOREACH(loop_var RANGE total)
ENDFOREACH(loop_var)
#范围和步进
FOREACH(loop_var RANGE start stop [step])
ENDFOREACH(loop_var)
```


