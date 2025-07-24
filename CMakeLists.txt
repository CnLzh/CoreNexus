cmake_minimum_required(VERSION 3.25)
project(CoreNexus LANGUAGES CXX)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 编译器支持检查
if(NOT (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC"
  OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU"
  OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang"))
    message(FATAL_ERROR "Unsupported compiler: only MSVC, GCC, Clang are supported.")
endif()

message(STATUS "Compiler: ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

# 编译选项接口库
add_library(core_nexus_options INTERFACE)

# 编译选项
target_compile_options(core_nexus_options INTERFACE
  # MSVC
  $<$<CXX_COMPILER_ID:MSVC>:
    /W4
    /WX
    /EHs-c-
    /GR-
    /permissive-
    /Zc:__cplusplus
  >
  
  # GCC/Clang 
  $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:
    -Wall -Wextra -Wpedantic
    -Werror
    -fno-exceptions
    -fno-rtti
    -Wconversion
    -Wsign-conversion
    -Wunused
    -Wold-style-cast
  >
  
  # GCC
  $<$<CXX_COMPILER_ID:GNU>:
    -Wduplicated-cond
    -Wduplicated-branches
    -Wlogical-op
  >

  # Clang
  $<$<CXX_COMPILER_ID:Clang>:
    -Wextra-semi
  >
)

target_compile_options(core_nexus_options INTERFACE
  # Debug
  $<$<CONFIG:Debug>:
    $<$<CXX_COMPILER_ID:MSVC>:/Od /Zi /RTC1>
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-O0 -g3 -DDEBUG>
  >

  # RelWithDebInfo
  $<$<CONFIG:RelWithDebInfo>:
    $<$<CXX_COMPILER_ID:MSVC>:/O2 /Zi /DNDEBUG>
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-O2 -g -DNDEBUG>
  >

  # Release
  $<$<CONFIG:Release>:
    $<$<CXX_COMPILER_ID:MSVC>:/O2 /DNDEBUG>
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-O3 -DNDEBUG>
  >
)

# 编译定义
target_compile_definitions(core_nexus_options INTERFACE
  $<$<CXX_COMPILER_ID:MSVC>:_HAS_EXCEPTIONS=0>
  $<$<CONFIG:Debug>:CORE_NEXUS_DEBUG>
)

# 链接
target_link_options(core_nexus_options INTERFACE
  $<$<CXX_COMPILER_ID:MSVC>:
    $<$<CONFIG:Debug>:/DEBUG>
  >
  $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:
    $<$<CONFIG:Debug>:-g>
    $<$<CONFIG:Release>:-s>
  >
)