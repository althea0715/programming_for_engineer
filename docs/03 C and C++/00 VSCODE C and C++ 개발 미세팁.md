# VSCODE C and C++ 개발 미세팁

## 1. 한글 경로가 되지 않을 때

```c++
// 방법1
setlocale(LC_ALL, "")

// 방법2
std::locale::global ( std::locale("") );
```

위 방법은 아무런 소용이 없고, 오른쪽 하단부에 텍스트 인코딩이 있는데, 이를 EUC-KR로 바꾸면 됩니다.