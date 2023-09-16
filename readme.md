### kautil_jvm_info
* the infomation of JVM (java virtual machine) to use it from c/c++.
* currently only support mingw.
* generate cmake configuration files which can avoid version conflicts.

### build
* use jdk whcih is download by embedded code. 
```shell
mkdir build
cmake -S . -B ./build 
cmake --install . --prefix=path_to_install_dir
```
* use jdk which exists on your system with no name.

```shell
mkdir build
cmake -S . -B ./build -DJVM_JDK_ROOT=path_to_jdk_on_your_file_system
cmake --install . --prefix=path_to_install_dir 
```
* use jdk which exists on your system with name.

```shell
mkdir build
cmake -S . -B ./build -DJVM_JDK_ROOT=path_to_jdk_on_your_file_system -DJVM_NAME='system_jdk'
cmake --install . --prefix=path_to_install_dir 
```

### cmake

* build  with no name
```cmake
set(KautilJniJvmInfo0.0.1.interface_DIR  "path_to KautilJniJvmInfo0.0.1.interfaceConfig.cmake dir")
find_package(KautilJniJvmInfo0.0.1.interface REQUIRED)
add_library(some static)
target_link_libraries(some PRIVATE kautil::jni::jvm_info::0.0.1::interface)
```

* build  with name 
* assume that build with like below with name 'corretto-18.0.2'
* cmake -S . -B ./build -DJVM_NAME='corretto-18.0.2'
```cmake
set(KautilJniJvmInfo-corretto-18.0.2-0.0.1.interface_DIR  "path_to KautilJniJvmInfo-corretto-18.0.2-0.0.1.interfaceConfig.cmake dir")
find_package(KautilJniJvmInfo-corretto-18.0.2-0.0.1.interface REQUIRED)
add_library(some static)
target_link_libraries(some PRIVATE kautil::jni::jvm_info-corretto-18.0.2::0.0.1::interface)
```
* assume that build with like below with name 'system_jdk'
* cmake -S . -B ./build -DJVM_NAME='system_jdk'
```cmake
set(KautilJniJvmInfo-system_jdk-0.0.1.interface_DIR  "path_to KautilJniJvmInfo-system_jdk-0.0.1.interfaceConfig.cmake dir")
find_package(KautilJniJvmInfo-system_jdk-0.0.1.interface REQUIRED)
add_library(some static)
target_link_libraries(some PRIVATE kautil::jni::jvm_info-system_jdk::0.0.1::interface)
```

