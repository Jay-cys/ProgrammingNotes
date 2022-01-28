## Basic executable
```
project(t1)
message(STATUS " binary path is" ${PROJECT_BINARY_DIR})

set(CMAKE_CXX_STANDARD 17)
add_executable(t1 main)
```
## sub directoryand make install path
```
cmake_minimum_required(VERSION 3.16)

project(t2)
message(STATUS " binary path is" ${PROJECT_BINARY_DIR})
message(STATUS " source path is" ${PROJECT_SOURCE_DIR})

set(CMAKE_CXX_STANDARD 17)
# 添加子目录(子目录也有CMkaeLists,用于本目录的编译)
add_subdirectory(src bin)

# 安装readme和版权文件
install(FILES COPYRIGHT README DESTINATION share/doc/cmake/t2)
# 安装sh文件
install(PROGRAMS runhello.sh DESTINATION bin)
# 安装doc hello.txt
install(DIRECTORY doc/ DESTINATION share/doc/cmake/t2)
# 安装可执行文件
install(PROGRAMS ${PROJECT_BINARY_DIR}/bin/t2 DESTINATION bin)
```
## build Static/Dynamic library
```
cmake_minimum_required(VERSION 3.16)

set(CMAKE_CXX_STANDARD 17)

message(STATUS "the library output path is" ${LIBRARY_OUTPUT_PATH})

# 编译动态库，不是executable
add_library(t3 SHARED hello.cpp)
# 设置动态库版本号
set_target_properties(t3 PROPERTIES VERSION 1.2 SOVERSION 1)

# 编译静态库
add_library(t3_static STATIC hello.cpp)
# 重命名静态库
set_target_properties(t3_static PROPERTIES OUTPUT_NAME "t3")

# 安装库文件
install(TARGETS t3 t3_static
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib)
# 安装头文件
install(FILES hello.hpp DESTINATION include/t3)
```
## add link for Static/Dynamic library
```
cmake_minimum_required(VERSION 3.16)

add_executable(t4 main.cpp)
# 添加头文件搜索路径
include_directories(/home/barret/t3/include/t3)

# 查找lib文件
# find_library(LIB_PATH libt3.a /home/barret/t3/lib) # 静态库
find_library(LIB_PATH libt3.so /home/barret/t3/lib) # 动态库
if(NOT LIB_PATH)
    MESSAGE(FATAL_ERROR "libt3 not found")
endif(NOT LIB_PATH)

message(STATUS "lib path" ${LIB_PATH})
# 链接库
target_link_libraries(t4 ${LIB_PATH})
```
