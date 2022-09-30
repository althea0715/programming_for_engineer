# Cython 적용 방법

해당 명령어로 Cython을 설치합니다.

```cmd
python -m pip install setuptools Cython
```

아래 2개의 파일을 만듭니다.

```python
# hello.py
def say_hello_to(name):
    print("Hello %s!" % name)
```

```python
# setup.py
from setuptools import setup
from Cython.Build import cythonize

setup(
    name="Hello world app",
    ext_modules=cythonize("hello.py"),
    zip_safe=False,
)
```

아래 명령어로 build를 진행합니다.

```cmd
python setup.py build_ext --inplace
```

당연하지만 C build 도구가 없으면 build를 할 수 없기 때문에 에러가 납니다. 저는 윈도우 기반이니 VC가 없다고 하네요.

```cmd
building 'hello' extension
error: Microsoft Visual C++ 14.0 or greater is required. Get it with "Microsoft C++ Build Tools": https://visualstudio.microsoft.com/visual-cpp-build-tools/  
```

친절하게도 link를 제공해줍니다. 다운로드 받아서 C/C++을 설치하도록 합니다. 저는 약 5.2GB가 설치되었습니다. 다시 build를 시도했더니 `hello.cp310-win_amd64.pyd` 파일이 생성되었습니다. 이를 읽어서 사용하면됩니다. 다만 build 라고 생성된 폴더는 삭제하셔도 됩니다.

편의상 hello.pyi 파일을 하나 만드시면 자동완성까지 편하게 사용할 수 있습니다.

```python
# hello.pyi
def say_hello_to(name: str) -> None: ...
```

아래처럼 사용하시면 됩니다.

```python
# main.py
from hello import say_hello_to

say_hello_to("althea")
```
