# librime-uos20-build-wiki 
# 背景：librime官方升级了编译依赖，在UOS v20（本质上基于debian 10）仓库里的boost，glog，gcc等包全部不能满足编译要求，在UOS v20上使用常规打包方法编译最新版本librime变得异常困难
# 本文用于指导静态编译librime，以下步骤由本人在UOS v20下的验证通过，推荐在pbuilder下进行，避免破坏本机UOS系统依赖
1. 安装cmake 3.2x或以上，liblua5.x-dev<br>*3.22版本cmake可以从社区版deepin 20.9源安装（请不要在UOS上装社区版源里面的其它无关的包，避免破坏UOS系统）
```
deb https://community-packages.deepin.com/deepin/ apricot main contrib non-free
```
2. 同步最新librime代码 ```git clone https://github.com/rime/librime.git```
3. 拉取submodule ```git submodule update --init```
4. 执行```bash install-plugins.sh hchunhui/librime-lua lotem/librime-octagram``` 添加两个插件，charcode 不推荐添加，本文步骤下不能成功编译
5. 修改deps目录下leveldb的CMakeLists.txt，避免后续编译失败
   <br>在```check_library_exists(tcmalloc malloc "" HAVE_TCMALLOC)```下面添加以下内容：<br>
   ```add_compile_options(-fPIC)```
   <br>或者在deps.mk中的leveldb区加上一句：<br>
   ```-DCMAKE_POSITION_INDEPDENT_CODE=TRUE```
7. 执行 ```make deps```
8. 执行 ```bash install-boost.sh```，建议手动下载对应的包到deps目录下再执行，脚本网络下载时特别慢
   并配置环境变量 ```export BOOST_ROOT=${librime_src_dir}/deps/boost-1.84.0```

9. 修改CMakeLists.txt，有两处需要修改

```option(BUILD_STATIC "Build with dependencies as static libraries" OFF)```  由OFF改为ON，意为静态编译

在```add_subdirectory(src)```
下添加如下代码：
```
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)
    if(GCC_VERSION VERSION_LESS 9.0)
        cmake_policy(SET CMP0079 NEW)
        target_link_libraries(${rime_library} stdc++fs)
    endif()
endif()
```
9. 执行```make merged-plugins```  然后取librime目录下的build目录下的lib下的so
   <br>本机编译安装，再执行```sudo make install```即可，如果要拷贝so到其它机器使用可能要手动安装liblua5.x

*10. 执行```objdump -x librime.so.1.11.2 | grep NEEDED```  如果出现类似以下结果，则表明，该so为静态编译完成
```
  NEEDED               libpthread.so.0
  NEEDED               liblua5.3.so.0
  NEEDED               libstdc++.so.6
  NEEDED               libm.so.6
  NEEDED               libgcc_s.so.1
  NEEDED               libc.so.6
  NEEDED               ld-linux-x86-64.so.2
```
