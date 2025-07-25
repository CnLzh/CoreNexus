cmake_minimum_required(VERSION 3.25)
project(CoreNexus LANGUAGES CXX)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 项目配置
option(CORE_NEXUS_WARNINGS_AS_ERRORS   "Treat warnings as errors"                  OFF)
option(CORE_NEXUS_ENABLE_LTO           "Enable IPO/LTO in Release/RelWithDebInfo"  OFF)

# 全局配置
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_VISIBILITY_PRESET hidden)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)

# MSVC 运行时
if(MSVC)
  set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")
endif()

# LTO / IPO 
if(CORE_NEXUS_ENABLE_LTO)
    include(CheckIPOSupported)
    check_ipo_supported(RESULT CORE_NEXUS_HAS_IPO)
    
    if(CORE_NEXUS_HAS_IPO)
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELEASE TRUE)
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELWITHDEBINFO TRUE)
        message(STATUS "LTO/IPO enabled for Release builds")
    else()
        message(WARNING "LTO/IPO requested but not supported")
    endif()
endif()

# 编译项接口库
add_library(core_nexus_options INTERFACE)

# 基础编译项
target_compile_options(core_nexus_options INTERFACE
    # MSVC
    $<$<CXX_COMPILER_ID:MSVC>:
        /W4 /EHs-c- /GR- /permissive- /Zc:__cplusplus
        $<$<BOOL:${CORE_NEXUS_WARNINGS_AS_ERRORS}>:/WX>
    >
    
    # GCC/Clang
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:
        -Wall -Wextra -Wpedantic -fno-exceptions -fno-rtti
        -Wconversion -Wsign-conversion -Wshadow
        $<$<BOOL:${CORE_NEXUS_WARNINGS_AS_ERRORS}>:-Werror>
    >
    
    # 构建类型相关
    $<$<CONFIG:Debug>:
        $<$<CXX_COMPILER_ID:MSVC>:/Od /Zi>
        $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-O0 -g3>
    >
    $<$<CONFIG:Release>:
        $<$<CXX_COMPILER_ID:MSVC>:/O2>
        $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-O3>
    >
    $<$<CONFIG:RelWithDebInfo>:
        $<$<CXX_COMPILER_ID:MSVC>:/O2 /Zi>
        $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-O2 -g>
    >
)

# 编译定义
target_compile_definitions(core_nexus_options INTERFACE
    $<$<CXX_COMPILER_ID:MSVC>:_HAS_EXCEPTIONS=0>
    $<$<CONFIG:Debug>:CORE_NEXUS_DEBUG>
    $<$<NOT:$<CONFIG:Debug>>:NDEBUG>
)

# 链接
target_link_options(core_nexus_options INTERFACE
    $<$<AND:$<CXX_COMPILER_ID:MSVC>,$<CONFIG:Debug>>:/DEBUG>
    $<$<AND:$<NOT:$<CXX_COMPILER_ID:MSVC>>,$<CONFIG:Release>>:-s>
)
