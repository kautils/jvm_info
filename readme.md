### kautil_jvm_info
* the infomation of JVM (java virtual machine) to use it from c/c++.
* currently only support mingw.

### example

```cmake
set(KautilJniJvmInfo0.0.1.interface_DIR  "path_to KautilJniJvmInfo0.0.1.interfaceConfig.cmake dir")
find_package(KautilJniJvmInfo0.0.1.interface REQUIRED)
add_library(some static)
target_link_libraries(some PRIVATE kautil::jni::jvm_info::0.0.1::interface)
```



