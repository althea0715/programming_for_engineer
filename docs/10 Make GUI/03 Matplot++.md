# Matplot++

ImGui에서 ImPlot widget을 쓰면 어느정도 그래프를 그릴 수 있긴합니다만, 실제로 공학관련 Plot을 그리기엔 상당히 부족합니다. 보통 결국 Python의 matplotlib를 사용하게되는데, 이와 매우 유사한 라이브러리가 있고 이를 OpenGL 내로 구현이 가능하여 소개하고자 글을 작성합니다.

Documents : https://alandefreitas.github.io/matplotplusplus/
Github : https://github.com/alandefreitas/matplotplusplus

## 1. Matplot++ 설치와 CMakeLists.txt

Github에서 다운로드 받으면 기존에 OpenGL 및 ImGui와 다르게 폴더 전부 복사해 넣어야합니다. CMakeLists.txt가 연계된 형태로 다음과 같은 절차를 따릅니다.

1. 폴더 통째로 복사합니다. 저는 matplotplusplus-master 폴더에서 -master를 제외하고 코드의 최상단 폴더에 넣었습니다.
2. CMakeLists.txt 파일에 다음 코드를 작성합니다. (my_target은 프로젝트명으로 적절히 수정합니다.)
   - add_subdirectory(matplotplusplus)
   - target_link_libraries(my_target PUBLIC matplot)

CMakeLists.txt 작성은 다음과 같습니다.

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(gl_project VERSION 0.1.0)

include(CTest)
enable_testing()

add_compile_options("/std:c++latest")

# matplot++ 폴더 추가
add_subdirectory(matplotplusplus)

include_directories(include)

# IMGUI soruce 설정
file(GLOB IMGUI_SRC include/imgui/*.cpp)

# Matplot++ OpenGL에 적용하기 위한 Backend 코드 추가
file(GLOB MATPLOT_SRC matplotplusplus/source/matplot/backend/*.cpp)

set(PROJECT_SRC
    main.cpp        # main 파일
    glad.c          # OpenGL 소스 파일
    ${IMGUI_SRC}    # ImGui 소스 파일
    ${MATPLOT_SRC}  # Matplot++ Backend 소스 파일 추가
)

add_executable(gl_project ${PROJECT_SRC})

# library내의 lib 파일 link
file(GLOB LIB_Files library/*.lib)

# matplot++ 라이브러리 추가
target_link_libraries(gl_project ${LIB_Files} matplot)  

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

## 2. Complie 하면 마주하게될 문제 해결 방법

matplot++의 core 코드 중에 line_spec.cpp에서 line 76, 392에 `custom_marker_` 변수를 정하는 코드가 있는데 `▶`이 적용되질 않아서 임의로 `>>`로 수정하였습니다.

OpenGL을 backend로 사용하려면 Backend 코드를 추가해야합니다. 위에서 언급된 CMakeLists.txt 에는 추가되어있습니다. 사용상에 있어서 파일을 save 하는 형태를 취한다면 backend를 사용하면 안됩니다. 그냥 코드로 그리고 save 하는 형태를 취할 수 밖에 없습니다.

## 3. 실제 코드

OpenGL + ImGui + Matplot 결합된 기본 형태 코드 구현

```cpp
#include <iostream>

#include <glad/glad.h>
#include <GLFW/glfw3.h>

#include <imgui/imgui_impl_glfw.h>
#include <imgui/imgui_impl_opengl3.h>
#include <imgui/imgui.h>

#include <imgui/implot.h>
#include <imgui/imfilebrowser.h>

#include <matplot/backend/opengl_embed.h>
#include <matplot/matplot.h>

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

    ImGui::FileBrowser fileDialog;

    fileDialog.SetTitle("title");
    fileDialog.SetTypeFilters({".h", ".cpp"});

    // Plot을 그리기 위한 figure 설정
    auto f = matplot::figure<matplot::backend::opengl_embed>(true);
    auto ax = f->current_axes();
    ax->xlim({0., 2. * matplot::pi});
    ax->ylim({-1.5, 1.5});
    ax->yticks(matplot::iota(-1.5, 0.5, +1.5));
    ax->xticks(matplot::iota(0., 1., 2. * matplot::pi));

    // OpenGL Loop
    while (!glfwWindowShouldClose(window)) {
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // Matplot++ 와 상호작용을 위해 만들어놓은 input 함수
        matplot::backend::opengl_embed::process_input(window);

        // Plot 설정
        double timeValue = glfwGetTime();
        std::vector<double> x = matplot::linspace(0., 2. * matplot::pi);
        std::vector<double> y = matplot::transform(x, [&](auto x) { return sin(x + timeValue); });
        ax->hold(matplot::off);
        ax->plot(x, y, "-o");
        ax->hold(matplot::on);
        ax->plot(x, matplot::transform(y, [](auto y) { return -y; }), "--xr");
        ax->plot(x, matplot::transform(x, [](auto x) { return x / matplot::pi - 1.; }), "-:gs");
        ax->plot({1.0, 0.7, 0.4, 0.0, -0.4, -0.7, -1}, "k");

        // 실제로 그리는 함수
        f->draw();

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

## 4. ImGui widget 내에 Matplot++를 넣는 방법

이 방법은 찾아보고 있습니다.


## 5. Matplot++ Plot Save

Plot을 Save 하기 위해선 gnuplot이 있어야합니다.