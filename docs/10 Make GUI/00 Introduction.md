# introduction

사내에서 Python으로 PyQT를 활용하여 배포하고자 하면 항상 용량문제가 발생합니다. 그래서 여러가지 고민하고 찾아본 끝에 ImGui라는 게임을 위한 무료 GUI Library을 발견했고, Python의 Matplotlib와 매우 유사한 C++ Library를 발견했습니다. 이는 둘 다 OpenGL로 구동이되므로 실제 개발할 수 있는 환경이 될지 테스트 하고자 합니다. 꼭 안되더라도 OpenGL, ImGui, Matplot++ 전부 개별로 배우기에도 충분히 가치가 있는 Library라고 생각합니다.

C나 C++로는 데이터 처리가 매우 번거로운 점을 참작하여 Python이나 Rust를 연동하여 일반적인 데이터 처리를 하고자하며, 용량 문제를 주 이슈로 삼은 만큼 Rust가 데이터 처리의 주력언어로 사용해보려고 합니다.

*MSVC 설치가 되었다는 전제조건 하에 설명합니다. MSVC 설치는 Python Section의 Cython 적용방법에서 설명하였습니다.*
*추가적인 설정장법은 C and C++ Section의 Python for C language에서 설명하였습니다.*
*VSCODE + CMAKE 개발환경에서 작성되었습니다.*