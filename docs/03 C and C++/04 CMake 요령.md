# CMake 요령

## 1. CMake로 MSVC compile 할때 최신 버젼 사용하는 방법

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(my_test VERSION 0.1.0)

add_compile_options("/std:c++latest")   # 이 부분 참조

include(CTest)
enable_testing()

add_executable(my_test main.cpp)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

## 2. MSVCRT Warning 제거

OpenGL 실행시 아래와 같은 Warning이 발생할 수 있습니다.

> [build] LINK : warning LNK4098: defaultlib 'MSVCRT' conflicts with use of other libs; use /NODEFAULTLIB:library 

그렇다면 cmake에 다음과 같은 키워드를 추가하면 됩니다.

```cmake
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /NODEFAULTLIB:MSVCRT")
```