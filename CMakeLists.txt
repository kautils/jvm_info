cmake_minimum_required(VERSION 3.20)
set(__version 0.0.1)
project(KautilJniJvmInfo${__version}
        LANGUAGES C CXX
        VERSION ${__version}
        )
unset(__version)

if(NOT DEFINED KAUTIL_THIRD_PARTY_DIR)
    set(KAUTIL_THIRD_PARTY_DIR ${CMAKE_BINARY_DIR})
    file(MAKE_DIRECTORY "${KAUTIL_THIRD_PARTY_DIR}")
endif()

macro(git_clone url)
    get_filename_component(file_name ${url} NAME)
    if(NOT EXISTS ${KAUTIL_THIRD_PARTY_DIR}/kautil_cmake/${file_name})
        file(DOWNLOAD ${url} "${KAUTIL_THIRD_PARTY_DIR}/kautil_cmake/${file_name}")
    endif()
    include("${KAUTIL_THIRD_PARTY_DIR}/kautil_cmake/${file_name}")
    unset(file_name)
endmacro()

git_clone(https://raw.githubusercontent.com/kautils/CMakeLibrarytemplate/v0.0.1/CMakeLibrarytemplate.cmake)


set(${PROJECT_NAME}_m_evacu ${m})
set(m ${PROJECT_NAME}_evacu)

set(JVM_NAME "${JVM_NAME}" CACHE STRING "name of jvm. this is used when using multiple version of jvms" FORCE)
set(JVM_JNI_VERSION "${JVM_JNI_VERSION}" CACHE STRING "jni version such as JNI_VERSION_1_6. this is required officially. the detail should be found online." FORCE)
set(JVM_JDK_ROOT "${JVM_JDK_ROOT}" CACHE STRING "path to jdk directory. this directory should have bin,include,jmods and others." FORCE)


list(APPEND ${m}_unsetter ${m}_jvm_name ${m}_jvm_name_suffix)
if(NOT ${JVM_NAME} STREQUAL "")
    set(${m}_jvm_name "${JVM_NAME}")
    set(${m}_jvm_name_suffix "-${${m}_jvm_name}")
else()
    set(${m}_jvm_name "")
    set(${m}_jvm_name_suffix "")
endif()


if(NOT ${JVM_JNI_VERSION} STREQUAL "")
    set(KAUTIL_JNI_VERSION ${JNI_VERSION})
else()
    set(KAUTIL_JNI_VERSION JNI_VERSION_1_6)
endif()


set(KAUTIL_JDK_BUNDLED FALSE)
if(NOT ${JVM_JDK_ROOT} STREQUAL "")
    set(KAUTIL_JDK_ROOT "${JVM_JDK_ROOT}")
    string(REPLACE "\\" "/" KAUTIL_JDK_ROOT "${KAUTIL_JDK_ROOT}" )
else()
    set(KAUTIL_JDK_BUNDLED TRUE)
    set(KAUTIL_JDK_VERSION 18.0.2.9.1)
    set(KAUTIL_JDK_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/third_party/open_jdk.${KAUTIL_JDK_VERSION}")
endif()


if(${KAUTIL_JDK_BUNDLED})
    list(APPEND ${m}_unsetter ${m}_dst_suffix ${m}_url)
    git_clone(https://raw.githubusercontent.com/kautils/CMakeFetchZip/v0.0.1/CMakeFetchZip.cmake)
    set(${m}_dst_suffix ${CMAKE_HOST_SYSTEM})
    if("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows" )
        set(${m}_url https://corretto.aws/downloads/resources/${KAUTIL_JDK_VERSION}/amazon-corretto-${KAUTIL_JDK_VERSION}-windows-x64-jdk.zip)
        set(KAUTIL_JDK_INCLUDE_PATH_platform_dep "${KAUTIL_JDK_ROOT}/include/win32")
    elseif("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
        set(${m}_url https://corretto.aws/downloads/resources/${KAUTIL_JDK_VERSION}/amazon-corretto-${KAUTIL_JDK_VERSION}-linux-x64.tar.gz)
    endif()
    
    if(NOT DEFINED KAUTIL_JDK_ROOT OR NOT EXISTS "${KAUTIL_JDK_ROOT}")
        CMakeFetchZip(SOURCE ${${m}_url} DESTINATION ${KAUTIL_JDK_ROOT} FORCE_EXTRACT)
        set(${m}_done TRUE)
    endif()
    
    file(GLOB ${m}_want ${KAUTIL_JDK_ROOT}/*)
    foreach(__var ${${m}_want})
        set(KAUTIL_JDK_ROOT ${__var})
        break()
    endforeach()
endif()


set(KAUTIL_JDK_INCLUDE_PATH "${KAUTIL_JDK_ROOT}/include")
set(KAUTIL_JNI_PATH_TO_JVM_SHARED_LIB "${KAUTIL_JDK_ROOT}/bin/server/jvm${CMAKE_SHARED_LIBRARY_SUFFIX}")

if(NOT DEFINED KAUTIL_JNI_PATH_TO_JVM_SHARED_LIB OR NOT EXISTS "${KAUTIL_JNI_PATH_TO_JVM_SHARED_LIB}")
    message(FATAL_ERROR "jvm${CMAKE_SHARED_LIBRARY_SUFFIX} is invalid : \"${KAUTIL_JNI_PATH_TO_JVM_SHARED_LIB}\"")
endif()


set(module_name jvm_info)
unset(srcs)
set(${module_name}_common_pref
    MODULE_PREFIX kautil jni 
    MODULE_NAME ${module_name}${${m}_jvm_name_suffix}
    INCLUDES 
        $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}> 
        $<BUILD_INTERFACE:${KAUTIL_JDK_INCLUDE_PATH}>
    EXPORT_NAME_PREFIX ${PROJECT_NAME}${${m}_jvm_name_suffix}
    EXPORT_VERSION ${PROJECT_VERSION}
    EXPORT_VERSION_COMPATIBILITY AnyNewerVersion
        
    DESTINATION_CMAKE_DIR cmake
    DESTINATION_LIB_DIR lib
)


CMakeLibraryTemplate(${module_name} EXPORT_LIB_TYPE interface ${${module_name}_common_pref} )

# build_insterface
target_compile_definitions(${${module_name}_interface} INTERFACE
        $<BUILD_INTERFACE:JVM_SO="${KAUTIL_JNI_PATH_TO_JVM_SHARED_LIB}">
        $<BUILD_INTERFACE:JNI_VERSION_BUILD_TIME=${KAUTIL_JNI_VERSION}>
        )
#install_interface
if(${KAUTIL_JDK_BUNDLED})
    target_compile_definitions(${${module_name}_interface} INTERFACE
            $<INSTALL_INTERFACE:JVM_SO=$<INSTALL_PREFIX>/bin/${${m}_jvm_name}/server/jvm.dll>
            $<INSTALL_INTERFACE:JNI_VERSION_BUILD_TIME=${KAUTIL_JNI_VERSION}>
            )
    target_include_directories(${${module_name}_interface} INTERFACE $<INSTALL_INTERFACE:include/${${m}_jvm_name}>)
    if(${KAUTIL_JDK_BUNDLED})
        install(DIRECTORY ${KAUTIL_JDK_ROOT}/bin/ DESTINATION bin/${${m}_jvm_name})
        install(DIRECTORY ${KAUTIL_JDK_ROOT}/include/ DESTINATION include/${${m}_jvm_name})
        install(DIRECTORY ${KAUTIL_JDK_ROOT}/jmods/ DESTINATION jmods/${${m}_jvm_name})
    endif()
else()
    target_compile_definitions(${${module_name}_interface} INTERFACE
            $<INSTALL_INTERFACE:JVM_SO="${KAUTIL_JNI_PATH_TO_JVM_SHARED_LIB}">
            $<INSTALL_INTERFACE:JNI_VERSION_BUILD_TIME=${KAUTIL_JNI_VERSION}>
            )
    target_include_directories(${${module_name}_interface} INTERFACE $<INSTALL_INTERFACE:${KAUTIL_JDK_INCLUDE_PATH}>)
endif()


# could not solve "INTERFACE_INCLUDE_DIRECTORY which is prefixed in the source directory." error
set(KautilJniJvmInfo0.0.1${${m}_jvm_name_suffix}.interface_DIR ${CMAKE_CURRENT_BINARY_DIR})



foreach(__v ${${m}_unsetter})
    unset(${__v})
endforeach()
unset(${m}_unsetter)
set(m ${${PROJECT_NAME}_m_evacu})


