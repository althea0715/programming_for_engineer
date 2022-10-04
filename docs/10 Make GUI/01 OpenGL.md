# OpenGL

Reference
- 영문판 : https://learnopengl.com/
- 한글판 : https://heinleinsgame.tistory.com/3

## 1. GLFW 설치

설치 경로 : https://www.glfw.org/download.html

Windows pre-compiled binaries에서 windows 64bit를 다운로드 받습니다.

## 2. GLAD 설치

설치 경로 : https://glad.dav1d.de/

Language : C/C++
Specification : OpenGL
gl : Version 3.3 (상위도 괜찮으나 3.3 ImGui, Matplot++가 3버젼을 지원합니다.)
Profile : Core
Option : Generate a loader (선택해야합니다.)

나머지는 선택하지 않고 GENERATE 합니다.

glad.zip를 다운로드 받습니다.

## 3. CMakeLists.txt 설정

기본적으로 VSCODE에서 CMake: Quick Start를 했다고 가정하겠습니다.

1. 최상위 폴더에 include 폴더를 만듭니다.
   - GLFW 다운로드 받은 폴더의 include 폴더 내 GLFW 폴더를 그대로 복사합니다.
   - GLAD 다운로드 받은 폴더의 include 폴더 내 glad, KHR 폴더를 그대로 복사합니다.
2. 최상위 폴더에 library 폴더를 만듭니다.
   - GLFW 다운로드 받은 폴더에서 자신에게 맞는 버젼 폴더의 파일을 복사해 붙여넣습니다. (저는 lib-vc2022 입니다.)
   - (중요!) 단, dll 파일은 향후에 debug로 complie을 하거나 release로 하거나 exe 파일 만들어지는 폴더로 이동해야합니다.
3. 최상위 폴더에 GLAD 다운로드 받은 폴더의 src 폴더의 glad.c를 복사합니다.

위 상황에서 CMakeLists.txt 파일을 작성하겠습니다.

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(gl_project VERSION 0.1.0)

include(CTest)
enable_testing()

# GLFW, GLAD header 파일 연결
include_directories(include)

# glad.c 추가 compile 연결
add_executable(gl_project main.cpp glad.c)

# library내의 lib 파일 link
file(GLOB LIB_Files library/*.lib)
target_link_libraries(gl_project ${LIB_Files})

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

main.cpp 파일에서 반드시 glad를 먼저 include하고 glfw를 include 해야합니다. VSCODE에서 자동으로 include 순서를 정렬할 수 있으니 정렬 옵션을 해제하시면 편합니다. (C_Cpp: Clang_format_sort Includes)

신기하게 한글 주석때문에 정상 실행되지 않는 경우가 있습니다. 참고하시면 좋습니다.

```c
// main.cpp, 예외처리는 하지 않았습니다.
#include <glad/glad.h>
#include <GLFW/glfw3.h>

// 창크기 변경될 때마다 크기 조절해주는 callback 함수 설정
void framebuffer_size_callback(GLFWwindow* window, int width, int height);

int main(int argc, char** argv) {
    glfwInit();

    // OpenGL 3.3 버전 쓸거고 Legacy한 함수는 사용안하겠다는 뜻입니다.
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    // 사용할 window 만들고 주 컨텍스트로 설정
    GLFWwindow* window = glfwCreateWindow(800, 600, "OpenGLTutorial", NULL, NULL);
    glfwMakeContextCurrent(window);

    // 렌더링할 윈도우 사이즈 초기 설정 및 윈도우 사이즈 변경시마다 자동으로 수정해줄 callback 등록
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

    // glad 설정
    gladLoadGLLoader((GLADloadproc)glfwGetProcAddress);

    // window 종료 명령어 안나오면 계속 실행합니다.
    while (!glfwWindowShouldClose(window)) {
        glfwSwapBuffers(window);  // double buffering 실행
        glfwPollEvents();         // 이벤트 확인하여 callback 함수 실행
    }

    // 종료 함수
    glfwTerminate();
    return 0;
}

void framebuffer_size_callback(GLFWwindow* window, int width, int height) {
    glViewport(0, 0, width, height);
}
```


