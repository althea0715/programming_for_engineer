# Python for C language

C에서 구현하기 힘든 로직들을 Python에서 구현하여 C로 읽어들이는 방법을 사용하면 효율적으로 엔지니어링에 도움되는 프로그램을 만들 수 있습니다. 대표적으로 Text Parsing이 있습니다. Python C/API를 통해 Python Library를 만드는 것이 쉽지 않을 뿐더러 단순히 Python으로 된 로직의 결과값을 받는 것이 필요합니다.

즉, 연산 자체의 속도에 중점을 두는 것은 C로 진행하고 연산 속도에 중점을 두지 않는 로직은 Python을 활용하기 위하여 Python C/API를 사용하고자 합니다.

이를 위한 방법을 정리해둡니다.

실제 사용에 앞서 예제로 사용할 Python 파일을 하나 만들었습니다.

```python
# python_lib.py

import numpy as np

def return_int():
    return 10

def return_float():
    return 1.1

def return_str():
    return "test_str"

def return_tuple():
    return (9, 8)

def return_list():
    return [7,6,5]

def return_ndarray():
    return np.array([1.0, 8.0, 7.0])

def input_test(a:int, b:float, c:str, d:tuple, e:list):
    print(a)
    print(b)
    print(c)
    print(d)
    print(e)
    
    return "input_test"


class Klass:
    
    number = 0
    
    def __init__(self, value):
        self.number = value
        
    def output(self, n):
        return self.number + n
    
    def add(self):
        self.number += 1
```

## 0. 개발 환경

*MSVC 설치가 되었다는 전제조건 하에 설명합니다. MSVC 설치는 Python Section의 Cython 적용방법에서 설명했습니다.*

Windows + VSCODE에서 Python + C 연동을 위한 방법 정리

제 환경은 다음과 같습니다.

- Python 3.10.6
- MSVC 2022
- VSCODE

사전에 Microsoft에서 제공하는 방법을 일단 읽어보는 것을 추천합니다. 

https://code.visualstudio.com/docs/cpp/config-msvc

기본적으로 Developer Command Prompt for Visual Studio를 사용하여 VSCODE를 열고 VSCODE의 C/C++ Extension Pack을 설치합니다. 그럼 이것저것 설치되는데 필요한 내용이 전부 설치되니 위에서 제공하는 방법을 굳이 진행하지 않아도 됩니다.

cmake가 설치되었으니 ctrl + shift + p 를 눌러서 나오는 명령행에서 cmake quick start를 실행하시고 적당히 입력하면 CMakeLists.txt 파일이 만들어지며 필요한 내용이 작성됩니다. 하지만 이 글의 주 언어가 파이썬이므로 Python.h를 include 하기위해서 아래와 같은 내용까지 추가하여 작성하여야합니다.

```cmake
# 실행 파일 이름 및 프로젝트 이름 설정
set(EXE_FILE_NAME python_for_c)

cmake_minimum_required(VERSION 3.0.0)
project(EXE_FILE_NAME VERSION 0.1.0)

include(CTest)
enable_testing()

# Python 폴더 설정 (Ref : https://cmake.org/cmake/help/latest/module/FindPython3.html)
find_package (Python3 COMPONENTS Interpreter Development)
include_directories(${Python3_INCLUDE_DIRS})

# 실제로 실행될 파일 설정
add_executable(EXE_FILE_NAME main.cpp)

# Python lib 파일 등록
target_link_libraries(EXE_FILE_NAME ${Python3_LIBRARIES})

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

미세팁을 추가합니다.

- 이상하게 Debug Mode로 Compile이 되지 않습니다. Release Mode로 Compile 하시면 됩니다.
- Python이 설치된 폴더에서 `python3.dll`를 EXE 파일이 만들어지는 경로에 붙여넣기 해주세요. 저는 저의 Project 폴더에서 build/Release 폴더입니다.
- 구현한 python_lib.py 파일은 build/Release/pylib 폴더를 만들어 넣어놨습니다.


## 1. C언어로 Python의 일반적인 변수를 읽는 법

보통의 경우 Python으로 구현된 로직의 결과값만 받아오면 됩니다. 이를 위한 c언어 로직을 소개합니다.

Reference : https://docs.python.org/ko/3.9/c-api/intro.html

```c
// main.cpp

#define PY_SSIZE_T_CLEAN

#include <stdio.h>
#include <Python.h>

int main(int, char **) {
    Py_Initialize();

    // pyLibPath : py 파일이 저장될 위치, 여기서는 exe 파일이 있는 곳의 상대위치를 뜻합니다.
    // pyLibName : py 파일 이름, 확장자는 포함하지 않습니다.
    const char *pyLibPath = "pylib";
    const char *pyLibName = "python_lib";

    // 사용자 라이브러리를 찾는 경로가 설정되어야합니다.
    // 만약, 아래 코드가 없다면 반드시 exe 파일과 같은 위치에 py 파일이 있어야합니다.
    PyObject *sysPath = PySys_GetObject((char *)"path");
    PyList_Append(sysPath, (PyUnicode_FromString(pyLibPath)));

    // py 파일을 읽고 py 파일 자체를 모듈로 만듭니다.
    PyObject *pName = PyUnicode_DecodeFSDefault(pyLibName);
    PyObject *pModule = PyImport_Import(pName);

    Py_DECREF(pName);  // 파일 경로는 썼으니 Reference Counter를 줄입니다. (사실상 메모리 해제)

    // 함수를 호출하는 방법
    PyObject *pFuncGetInt = PyObject_GetAttrString(pModule, "return_int");
    PyObject *pFuncGetFloat = PyObject_GetAttrString(pModule, "return_float");
    PyObject *pFuncGetString = PyObject_GetAttrString(pModule, "return_str");
    PyObject *pFuncGetTuple = PyObject_GetAttrString(pModule, "return_tuple");
    PyObject *pFuncGetList = PyObject_GetAttrString(pModule, "return_list");

    // 모듈 썼으니 Reference Counter를 줄입니다. (사실상 메모리 해제)
    Py_DECREF(pModule);

    // 함수를 지정합니다.
    PyObject *pValueInt = PyObject_CallObject(pFuncGetInt, NULL);
    PyObject *pValueFloat = PyObject_CallObject(pFuncGetFloat, NULL);
    PyObject *pValueString = PyObject_CallObject(pFuncGetString, NULL);
    PyObject *pValueTuple = PyObject_CallObject(pFuncGetTuple, NULL);
    PyObject *pValueList = PyObject_CallObject(pFuncGetList, NULL);

    // 함수를 썼으니 메모리 해제합니다.
    Py_XDECREF(pFuncGetInt);
    Py_XDECREF(pFuncGetFloat);
    Py_XDECREF(pFuncGetString);
    Py_XDECREF(pFuncGetTuple);
    Py_XDECREF(pFuncGetList);

    printf("Int Output : %ld\n", PyLong_AsLong(pValueInt));          // Python에서는 int가 없고 long입니다.
    printf("Float Output : %lf\n", PyFloat_AsDouble(pValueFloat));   // python에서는 float가 없고 double입니다.
    printf("String Output : %s\n", PyUnicode_AsUTF8(pValueString));  // 일반적인 text는 utf-8입니다.

    printf("Tuple Output : ");  // Tuple은 한개씩 받아서 처리하면 됩니다.
    for (int i = 0; i < PyTuple_GET_SIZE(pValueTuple); i++) {
        PyObject *item = PyTuple_GetItem(pValueTuple, i);
        printf("%d, ", PyLong_AS_LONG(item));
    }
    printf("\n");

    printf("List Output : ");  // List도 Tuple과 동일하게 한개씩 받아서 처리합니다.
    for (int i = 0; i < PyTuple_GET_SIZE(pValueList); i++) {
        PyObject *item = PyList_GetItem(pValueList, i);
        printf("%d, ", PyLong_AS_LONG(item));
    }
    printf("\n");

    // 사용한 변수들 메모리 해제
    Py_XDECREF(pValueInt);
    Py_XDECREF(pValueFloat);
    Py_XDECREF(pValueString);
    Py_XDECREF(pValueTuple);
    Py_XDECREF(pValueList);

    Py_FinalizeEx();

    return 0;
}
```

위 cpp 파일을 complie 하면 python_lib.py 에서 만들어 놓은 결과물이 화면에 출력됩니다.

## 2. C언어로 Numpy를 읽는 법

python 구현체인 python_lib.py 파일은 그대로 둔체 main.cpp 파일을 수정하여 작성합니다. 단, numpy를 읽기 위해선 CMakeLists.txt 파일에서 numpy를 등록해줘야합니다.

```cmake
# Python Numpy Include 설정
include_directories(${Python3_SITEARCH}/numpy/core/include)

# Python Numpy lib 등록
target_link_libraries(EXE_FILE_NAME ${Python3_SITEARCH}/numpy/core/lib/npymath.lib)
```

위에 추가된 내용을 포함하여 전체 CMakeLists.txt은 다음과 같습니다.

```cmake
# 실행 파일 이름 및 프로젝트 이름 설정
set(EXE_FILE_NAME python_for_c)

cmake_minimum_required(VERSION 3.0.0)
project(EXE_FILE_NAME VERSION 0.1.0)

include(CTest)
enable_testing()

# Python 폴더 설정 (Reference : https://cmake.org/cmake/help/latest/module/FindPython3.html)
find_package (Python3 COMPONENTS Interpreter Development)
include_directories(${Python3_INCLUDE_DIRS})

# Python Numpy Include 설정
include_directories(${Python3_SITEARCH}/numpy/core/include)

# 실제로 실행될 파일 설정
add_executable(EXE_FILE_NAME main.cpp)

# Python lib 파일 등록
target_link_libraries(EXE_FILE_NAME ${Python3_LIBRARIES})

# Python Numpy lib 등록
target_link_libraries(EXE_FILE_NAME ${Python3_SITEARCH}/numpy/core/lib/npymath.lib)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

기존에 `PyObject_CallObject` 함수를 사용하여 `PyObject`를 가져왔다면 이제는 numpy object를 가져와야합니다. numpy object는 `PyArrayObject`이므로 `PyObject`를 `PyArrayObject`로 casting 해줍니다. numpy 객체에서 가져올 수 있는 다양한 데이터는 함수를 제공하니 Reference를 확인하시면 됩니다. 

Reference 
- https://numpy.org/doc/stable/reference/c-api/types-and-structures.html
- https://numpy.org/doc/stable/reference/c-api/array.html

```c
#define PY_SSIZE_T_CLEAN

#include <stdio.h>
#include <Python.h>
#include <numpy/arrayobject.h>  // numpy 라이브러리를 읽습니다.

int main(int, char **) {
    Py_Initialize();

    const char *pyLibPath = "pylib";
    const char *pyLibName = "python_lib";

    PyObject *sysPath = PySys_GetObject((char *)"path");
    PyList_Append(sysPath, (PyUnicode_FromString(pyLibPath)));

    PyObject *pName = PyUnicode_DecodeFSDefault(pyLibName);
    PyObject *pModule = PyImport_Import(pName);
    Py_DECREF(pName);

    // 함수를 지정합니다.
    PyObject *pFunc = PyObject_GetAttrString(pModule, "return_ndarray");
    Py_DECREF(pModule);

    // 그냥 PyObject가 아니므로 Casting이 필요합니다.
    PyArrayObject *pValue = (PyArrayObject *)PyObject_CallObject(pFunc, NULL);
    Py_DECREF(pFunc);

    // data를 가져오고 Shape를 가져옵니다.
    double *data = (double *)PyArray_DATA(pValue);
    npy_intp *shape = PyArray_SHAPE(pValue);

    printf("%d\n", shape[0]);
    printf("%lf\n", data[0]);
    printf("%lf\n", data[1]);

    free(data);
    free(shape);
    Py_XDECREF(pValue);

    Py_FinalizeEx();

    return 0;
}
```

## 3. C언어로 Python에 Arguments를 입력하는 방법

몇가지 방법이 있지만, 아래 소개하는 방법만 사용하는 것이 편리합니다.

Ref : https://docs.python.org/3/c-api/arg.html

```c
Py_BuildValue("i", 1)
Py_BuildValue("f", 1.1)
Py_BuildValue("s", "abc")
```

위와 같은 예시를 통하여 코드를 작성하면 다음과 같습니다.

```c
#define PY_SSIZE_T_CLEAN

#include <stdio.h>
#include <Python.h>

int main(int, char **) {
    Py_Initialize();

    const char *pyLibPath = "pylib";
    const char *pyLibName = "python_lib";

    PyObject *sysPath = PySys_GetObject((char *)"path");
    PyList_Append(sysPath, (PyUnicode_FromString(pyLibPath)));

    PyObject *pName = PyUnicode_DecodeFSDefault(pyLibName);
    PyObject *pModule = PyImport_Import(pName);
    Py_DECREF(pName);

    // 함수를 지정합니다.
    PyObject *pFunc = PyObject_GetAttrString(pModule, "input_test");
    Py_DECREF(pModule);

    // 입력할 변수를 설정합니다.
    PyObject *pArgs = Py_BuildValue("lds(l,l)[lds]", 9, 9.8, "abc", 8, 7, 6, 6.9, "def");

    // Return 값에 관심 없으니 pValue 변수를 사용하지 않습니다.
    PyObject_CallObject(pFunc, pArgs);
    Py_DECREF(pArgs);
    Py_DECREF(pFunc);

    Py_FinalizeEx();

    return 0;
}
```

```python
# 화면 출력된 결과
9
9.8
abc
(8, 7)
[6, 6.9, 'def']
```

이런 입력 방법을 사용하면 다양한 input값을 처리할 수 있습니다.

## 4. C언어로 Python Class를 사용하는 방법

대망의 Class를 읽는 방법이며, 다소 불편합니다. 편의성을 도모하기 위해 PyObject_CallMethod 함수를 도입했으며 그에 따라 위에서 언급한 방법들도 비교적 짧게 함수를 사용할 수 있습니다.

```c
#define PY_SSIZE_T_CLEAN

#include <stdio.h>
#include <iostream>
#include <Python.h>

int main(int, char **) {
    Py_Initialize();

    const char *pyLibPath = "pylib";
    const char *pyLibName = "python_lib";

    PyObject *sysPath = PySys_GetObject((char *)"path");
    PyList_Append(sysPath, (PyUnicode_FromString(pyLibPath)));

    PyObject *pName = PyUnicode_DecodeFSDefault(pyLibName);
    PyObject *pModule = PyImport_Import(pName);
    Py_DECREF(pName);

    // 클래스를 지정합니다.
    PyObject *pClass = PyObject_GetAttrString(pModule, "Klass");
    Py_DECREF(pModule);

    // 클래스의 Arguments는 tuple 형태로 기입해야합니다.
    // 당연하게도 __init__는 실행됩니다.
    PyObject *pArgs = Py_BuildValue("(i)", 3);
    PyObject *pInst = PyObject_CallObject(pClass, pArgs);

    // 클래스 내의 add 함수를 불러옵니다.
    // 단, add 함수는 Arguments가 없습니다. 그리고 return 값이 없습니다.
    PyObject *pInstAdd = PyObject_GetAttrString(pInst, "add");
    PyObject_CallObject(pInstAdd, NULL);

    // 클래스 내의 output 함수를 불러옵니다.
    // 여기서 편의상 PyObject_CallMethod 함수를 사용합니다.
    // 단, 클래스에서 PyObject_CallObject를 사용하여 Argument 입력할 땐 Tuple 형태로 기입해야합니다.
    PyObject *pResult = PyObject_CallMethod(pInst, "output", "i", 1);

    printf("Output : %d", PyLong_AsLong(pResult));

    Py_DECREF(pArgs);
    Py_DECREF(pInst);
    Py_DECREF(pInstAdd);
    Py_DECREF(pResult);
    Py_DECREF(pClass);

    Py_FinalizeEx();

    return 0;
}
```

클래스까지 쓸 일이 있을까 싶지만, 해당부분을 잘 정리해둔 곳이 따로 없어서 작성해둡니다.