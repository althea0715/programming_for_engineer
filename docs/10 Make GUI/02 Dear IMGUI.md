# Dear ImGui

## 1. imgui 개요

소스코드로만 이루어진 MIT 라이센스 C++ GUI Library 입니다. 

https://github.com/ocornut/imgui

자신의 C/C++ 프로젝트 내에 파일을 직접 복사하여 개발하는 형태를 띄는데 종속성이 없고 대부분의 상황을 지원해서 인기가 꽤 있는 모양입니다. 특히 게임과 관련된 곳에서 원활이 사용되고 있습니다.

imgui의 branch로 docking이 있는데, 원래 imgui에서 widget을 docking 할 수 있는 기능이 추가됬습니다. docking 기능 있는 것이 실제로 더 유용할 것이라고 판단되므로 docking branch를 기준으로 해당 문서를 작성합니다.

## 2. OpenGL

이 문서는 이미 작성된 OpenGL 문서 기준에서 작성되었습니다.

## 3. Docking branch와 CMakeLists.txt

imgui docking branch를 선택하셔서 다운로드 받습니다. 

1. 다운로드 받은 폴더의 최상단에 존재하는 모든 .h와 .cpp를 미리 만들어 놓은 프로젝트의 include/imgui 폴더에 복사합니다.
   - imstb_truetype.h, imconfig.h, imgui.cpp, imgui.h, imgui_demo.cpp, imgui_draw.cpp, imgui_internal.h, imgui_tables.cpp, imgui_widgets.cpp, imstb_rectpack.h, imstb_textedit.h
2. 다운로드 받은 폴더의 backends 폴더에서 glfw, opengl3와 관련있는 파일들을 include/imgui 폴더에 복사합니다.
   - imgui_impl_opengl3_loader.h, imgui_impl_glfw.cpp,imgui_impl_glfw.h, imgui_impl_opengl3.cpp,imgui_impl_opengl3.h

OpenGL에서 추가된 CMakeLists.txt 는 아래와 같이 작성하였습니다.

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(gl_project VERSION 0.1.0)

include(CTest)
enable_testing()

include_directories(include)

# IMGUI soruce 설정
file(GLOB IMGUI_SRC include/imgui/*.cpp)

set(PROJECT_SRC
    main.cpp        # main 파일
    glad.c          # OpenGL 소스 파일
    ${IMGUI_SRC}    # ImGui 소스 파일
)

add_executable(gl_project ${PROJECT_SRC})

# library내의 lib 파일 link
file(GLOB LIB_Files library/*.lib)
target_link_libraries(gl_project ${LIB_Files})

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

## 4. ImGui Demo 실행

Demo 에는 ImGui에서 구현할 수 있는 대부분의 widget을 구경할 수 있습니다. 데모를 실행하는 코드는 다음과 같이 작성할 수 있습니다. (기본이 되는 코드이며, 향후 수정하는 코드의 기초가 됩니다.)

```cpp
// main.cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>

#include <imgui/imgui_impl_glfw.h>
#include <imgui/imgui_impl_opengl3.h>
#include <imgui/imgui.h>

// OpenGL 강의자료에서 참조하였습니다.
const char glsl_version[] = "#version 330 core";

void framebuffer_size_callback(GLFWwindow* window, int width, int height);

int main(int argc, char** argv) {
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    GLFWwindow* window = glfwCreateWindow(800, 600, "OpenGLTutorial", NULL, NULL);
    glfwMakeContextCurrent(window);
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

    gladLoadGLLoader((GLADloadproc)glfwGetProcAddress);

    // 데모 윈도우를 열기위해 작성합니다.
    bool show_demo_window = true;

    // IMGUI 기본 설정
    IMGUI_CHECKVERSION();
    ImGui::CreateContext();
    ImGuiIO& io = ImGui::GetIO();

    // 갖은 flag 설정이 가능한데, Docking 기능을 일단 중심으로 작성하였습니다.
    io.ConfigFlags |= ImGuiConfigFlags_NavEnableKeyboard;  // Enable Keyboard Controls
    io.ConfigFlags |= ImGuiConfigFlags_DockingEnable;      // Enable Docking
    io.ConfigFlags |= ImGuiConfigFlags_ViewportsEnable;    // Enable Multi-Viewport / Platform Windows

    ImGui::StyleColorsDark();  // 테마 설정 가능합니다.

    ImGuiStyle& style = ImGui::GetStyle();
    if (io.ConfigFlags & ImGuiConfigFlags_ViewportsEnable) {
        style.WindowRounding = 0.0f;
        style.Colors[ImGuiCol_WindowBg].w = 1.0f;
    }

    // OpenGL과 ImGui를 연결하기 위한 함수 설정입니다.
    ImGui_ImplGlfw_InitForOpenGL(window, true);
    ImGui_ImplOpenGL3_Init(glsl_version);

    while (!glfwWindowShouldClose(window)) {
        // ImGui 창을 열기위한 설정입니다.
        ImGui_ImplOpenGL3_NewFrame();
        ImGui_ImplGlfw_NewFrame();
        ImGui::NewFrame();

        // 실제로 여기에 Widget을 작성하면 됩니다.
        ImGui::ShowDemoWindow(&show_demo_window);

        // ImGui를 Rendering 하기 위한 영역입니다.
        ImGui::Render();
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);  // gl 함수를 통해 화면을 청소합니다.
        glClear(GL_COLOR_BUFFER_BIT);
        ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());

        // 열려있는 창 밖으로 꺼냈을 때 동작하는 설정입니다.
        if (io.ConfigFlags & ImGuiConfigFlags_ViewportsEnable) {
            GLFWwindow* backup_current_context = glfwGetCurrentContext();
            ImGui::UpdatePlatformWindows();
            ImGui::RenderPlatformWindowsDefault();
            glfwMakeContextCurrent(backup_current_context);
        }

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    // ImGui 종료시 메모리 정리합니다.
    ImGui_ImplOpenGL3_Shutdown();
    ImGui_ImplGlfw_Shutdown();
    ImGui::DestroyContext();

    glfwDestroyWindow(window);  // OpenGL 정리
    glfwTerminate();
    return 0;
}

void framebuffer_size_callback(GLFWwindow* window, int width, int height) {
    glViewport(0, 0, width, height);
}
```

## 5. ImPlot Demo 실행

ImGui third party 중에 사용하면 좋을 것 같은 library입니다. ImGui의 widget에서 Plot을 그릴 수가 있습니다. 다만, 개인적으로 vecter map(quiver)가 필요하므로 제 입장에서 한계점은 존재합니다. 역으로 직접 구현하는건 쉽지 않아 보입니다.

https://github.com/epezent/implot

다운로드 받은 모든 .h/.cpp 파일 include/imgui 폴더에 넣습니다. 동일한 경로에 넣기 때문에 CMakeLists.txt파일은 수정할 필요가 없습니다. 

implot를 위한 header 파일을 포함하여 실제로 작성되어야하는 코드는 지극히 적습니다. 

```cpp
// main.cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>

#include <imgui/imgui_impl_glfw.h>
#include <imgui/imgui_impl_opengl3.h>
#include <imgui/imgui.h>

#include <imgui/implot.h>

const char glsl_version[] = "#version 330 core";

void framebuffer_size_callback(GLFWwindow* window, int width, int height);

int main(int argc, char** argv) {
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    GLFWwindow* window = glfwCreateWindow(800, 600, "OpenGLTutorial", NULL, NULL);
    glfwMakeContextCurrent(window);
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

    gladLoadGLLoader((GLADloadproc)glfwGetProcAddress);

    bool show_demo_window = true;

    IMGUI_CHECKVERSION();
    ImGui::CreateContext();
    ImPlot::CreateContext();  // ImPlot을 위한 설정이 추가됩니다.
    ImGuiIO& io = ImGui::GetIO();

    io.ConfigFlags |= ImGuiConfigFlags_NavEnableKeyboard;
    io.ConfigFlags |= ImGuiConfigFlags_DockingEnable;
    io.ConfigFlags |= ImGuiConfigFlags_ViewportsEnable;

    ImGui::StyleColorsDark();

    ImGuiStyle& style = ImGui::GetStyle();
    if (io.ConfigFlags & ImGuiConfigFlags_ViewportsEnable) {
        style.WindowRounding = 0.0f;
        style.Colors[ImGuiCol_WindowBg].w = 1.0f;
    }

    ImGui_ImplGlfw_InitForOpenGL(window, true);
    ImGui_ImplOpenGL3_Init(glsl_version);

    while (!glfwWindowShouldClose(window)) {
        ImGui_ImplOpenGL3_NewFrame();
        ImGui_ImplGlfw_NewFrame();
        ImGui::NewFrame();

        ImGui::ShowDemoWindow(&show_demo_window);

        // ImPlot Demo
        ImPlot::ShowDemoWindow(&show_demo_window);

        ImGui::Render();
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);
        ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());

        if (io.ConfigFlags & ImGuiConfigFlags_ViewportsEnable) {
            GLFWwindow* backup_current_context = glfwGetCurrentContext();
            ImGui::UpdatePlatformWindows();
            ImGui::RenderPlatformWindowsDefault();
            glfwMakeContextCurrent(backup_current_context);
        }

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    ImGui_ImplOpenGL3_Shutdown();
    ImGui_ImplGlfw_Shutdown();
    ImPlot::DestroyContext();  // ImPlot을 위한 설정이 추가됩니다.
    ImGui::DestroyContext();

    glfwDestroyWindow(window);
    glfwTerminate();
    return 0;
}

void framebuffer_size_callback(GLFWwindow* window, int width, int height) {
    glViewport(0, 0, width, height);
}
```

## 6. imgui-filebrowser Demo 실행

ImGui github에는 몇가지 File Dialog 라이브러리가 링크되어 있습니다. 다만, OpenGL을 사용하는 한 선택에 제약이 있습니다.

<center>

|     Widget 명     | OpenGL 적용 가능 | 한글 가능 | 디자인 |
| :---------------: | :--------------: | :-------: | :----: |
|   ImFileDialog    |        X         |     -     |   O    |
|   L2DFileDialog   |        X         |     -     |   -    |
|  ImGuiFileDialog  |        O         |     X     |   O    |
| imgui-filebrowser |        O         |     O     |   -    |
| g's ImGui-Addons  |        O         |     -     |   X    |
| F's ImGui-Addons  |        O         |     -     |   X    |
</center>

ImFileDialog은 SDL에 종속이라 사용이 어려우며 ImGuiFileDialog는 한글을 인식을 못합니다. 결과적으로 선택지는 하나 밖에 없고 해당 Library를 추가로 사용법을 작성하겠습니다

두 가지 주의할 점이 있습니다.
1. 한글을 적용해야합니다. 해당 코드를 유심히 확인해주시길 바랍니다.
2. std:c++17 이상의 compiler가 필요합니다. (C and C++ 챕터의 CMake 요령을 참고)

https://github.com/AirGuanZ/imgui-filebrowser

링크를 찾아가셔서 다운로드 받으시면 include/imgui에 옮길 파일은 imfilebrowser.h 밖에 없습니다.

그리고 CMake 상단에 `add_compile_options("/std:c++latest")`를 추가해야합니다. (필자는 MSVC 19.33.31629이므로 c++20을 지원합니다. 그래서 latest로 작성하면 17을 포함하는 20 complier가 적용됩니다.)

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(gl_project VERSION 0.1.0)

include(CTest)
enable_testing()

# compiler 옵션 추가, 최신 버젼의 compiler 적용
add_compile_options("/std:c++latest")

include_directories(include)

# IMGUI soruce 설정
file(GLOB IMGUI_SRC include/imgui/*.cpp)

set(PROJECT_SRC
    main.cpp        # main 파일
    glad.c          # OpenGL 소스 파일
    ${IMGUI_SRC}    # ImGui 소스 파일
)

add_executable(gl_project ${PROJECT_SRC})

# library내의 lib 파일 link
file(GLOB LIB_Files library/*.lib)
target_link_libraries(gl_project ${LIB_Files})

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

main.cpp에 한글을 적용하는 방법은 `io.Fonts->AddFontFromFileTTF()` 함수를 사용하면 됩니다. 한글을 적용함과 동시에 Demo를 작성하겠습니다.

한글 적용 Reference : https://dlemrcnd.tistory.com/65

```c
#include <iostream>

#include <glad/glad.h>
#include <GLFW/glfw3.h>

#include <imgui/imgui_impl_glfw.h>
#include <imgui/imgui_impl_opengl3.h>
#include <imgui/imgui.h>

#include <imgui/implot.h>
#include <imgui/imfilebrowser.h>

const char glsl_version[] = "#version 330 core";

void framebuffer_size_callback(GLFWwindow* window, int width, int height);

int main(int argc, char** argv) {
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    GLFWwindow* window = glfwCreateWindow(800, 600, "OpenGLTutorial", NULL, NULL);
    glfwMakeContextCurrent(window);
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

    gladLoadGLLoader((GLADloadproc)glfwGetProcAddress);

    bool show_demo_window = true;

    IMGUI_CHECKVERSION();
    ImGui::CreateContext();
    ImPlot::CreateContext();
    ImGuiIO& io = ImGui::GetIO();

    // 한글 적용
    io.Fonts->AddFontFromFileTTF("C://windows/Fonts/malgun.ttf", 18.0f, NULL, io.Fonts->GetGlyphRangesKorean());

    io.ConfigFlags |= ImGuiConfigFlags_NavEnableKeyboard;
    io.ConfigFlags |= ImGuiConfigFlags_DockingEnable;
    io.ConfigFlags |= ImGuiConfigFlags_ViewportsEnable;

    ImGui::StyleColorsDark();

    ImGuiStyle& style = ImGui::GetStyle();
    if (io.ConfigFlags & ImGuiConfigFlags_ViewportsEnable) {
        style.WindowRounding = 0.0f;
        style.Colors[ImGuiCol_WindowBg].w = 1.0f;
    }

    ImGui_ImplGlfw_InitForOpenGL(window, true);
    ImGui_ImplOpenGL3_Init(glsl_version);

    // File Dialog를 위한 초기 설정입니다.
    ImGui::FileBrowser fileDialog;

    fileDialog.SetTitle("title");
    fileDialog.SetTypeFilters({".h", ".cpp"});

    // OpenGL Loop
    while (!glfwWindowShouldClose(window)) {
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        ImGui_ImplOpenGL3_NewFrame();
        ImGui_ImplGlfw_NewFrame();
        ImGui::NewFrame();

        // ImGui Demo
        ImGui::ShowDemoWindow(&show_demo_window);

        // ImPlot Demo
        ImPlot::ShowDemoWindow(&show_demo_window);

        // File Browser Demo
        if (ImGui::Begin("dummy window")) {
            // open file dialog when user clicks this button
            if (ImGui::Button("open file dialog"))
                fileDialog.Open();
        }
        ImGui::End();

        fileDialog.Display();

        if (fileDialog.HasSelected()) {
            std::cout << "Selected filename" << fileDialog.GetSelected().string() << std::endl;
            fileDialog.ClearSelected();
        }

        ImGui::Render();
        ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());

        if (io.ConfigFlags & ImGuiConfigFlags_ViewportsEnable) {
            GLFWwindow* backup_current_context = glfwGetCurrentContext();
            ImGui::UpdatePlatformWindows();
            ImGui::RenderPlatformWindowsDefault();
            glfwMakeContextCurrent(backup_current_context);
        }

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    ImGui_ImplOpenGL3_Shutdown();
    ImGui_ImplGlfw_Shutdown();
    ImPlot::DestroyContext();  // ImPlot을 위한 설정이 추가됩니다.
    ImGui::DestroyContext();

    glfwDestroyWindow(window);
    glfwTerminate();
    return 0;
}

void framebuffer_size_callback(GLFWwindow* window, int width, int height) {
    glViewport(0, 0, width, height);
}
```

저는 개인적으로 이 라이브러리의 ok, cancel 버튼 앞글자가 대문자였으면 좋겠어서 해당 text를 임의로 수정하였습니다.

## 7. Keyboard & Mouse Event 예제

확인 중입니다.