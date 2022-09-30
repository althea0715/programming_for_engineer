# Linear Algebra - OpenBLAS, LAPACK, CLAPACK, Eigen

선형 대수학을 하기위해 Python의 numpy 라이브러리를 사용하면 되지만, 좀 더 나은 속도를 얻고 싶거나 구조상 어쩔 수 없이 C/C++를 사용해야하는 경우가 있습니다. 그런 상황을 위해 이 문서를 작성합니다.

우선 C/C++로 선형대수학을 계산하기에 앞서 선택할 수 있는 다양한 라이브러리가 있습니다. 그 중에 LAPACK과 Eigen를 사용해 볼 것이고 편한 것을 선택하여 고르면 됩니다. LAPACK은 무료이면서 오래된 라이브러리라 선택하였으며 Eigen은 무료이면서 가장 많은 기능을 지원하기 때문에 선택하였습니다. (참고 : https://en.wikipedia.org/wiki/Comparison_of_linear_algebra_libraries)

## 0. 개발 환경

- MSVC 2022
- VSCODE

CMAKE를 사용하여 C/C++을 compile 하는 형태를 사용하며 CMAKE를 사용하는 대략적인 방법은 앞서 소개한바 있습니다. 그래도 CMakeLists.txt 작성 예시를 추가로 소개할 예정입니다.

## 1. BLAS/LAPACK

LAPACK을 사용하기 위해 소스를 직접 build 하거나, pre-build된 라이브러리를 사용할 수 있습니다. 해당 소스를 build하기 위한 준비과정이 많이 복잡하므로 저는 pre-build된 파일을 다운로드 받아서 사용하고자 합니다.

먼저 알아두어야할 것은 BLAS는 행렬 연산을 위한 라이브러리이며, LAPACK이 선형대수학을 연산하기 위한 라이브러리라는 것입니다. LAPACK은 BLAS를 사용하여 행렬 연산을 하며 BLAS는 다양한 버젼이 존재합니다. 혹여 BLAS만 설치하고 싶으시더라도 보통 pre-build된 경우 LAPACK까지 포함되어 있습니다.

### 1.1 OpenBLAS를 설치하는 방법

1. https://www.openblas.net 에 접속하면 아래의 다운로드 경로를 얻을 수 있습니다.
2. https://sourceforge.net/projects/openblas/files 에 접속하면 각종 버전이 나열되어 있습니다.
3. v0.2.14 버젼에 들어가시면 이미 windows 유저를 위해 pre-build된 OpenBLAS-v0.2.14-Win64-int64.zip 를 다운로드 받을 수 있습니다.
4. 단, 해당 압축파일에는 Fortran 관련한 라이브러리 파일이 없기 때문에 따로 찾아야합니다.

위에서 받을 수 있는 버전은 nuget에서도 받을 수 있으며 해당 link는 다음과 같습니다. link : https://www.nuget.org/packages/OpenBLAS/0.2.14.1

### 1.2 CLAPACK을 설치하는 방법

1. https://icl.utk.edu/lapack-for-windows/index.html 에 접속합니다.
2. https://icl.utk.edu/lapack-for-windows/clapack/index.html CLAPACK 메뉴에 접속합니다.
3. Prebuilt libraries에 있는 내용을 전부 다운로드 받습니다.

위에서 제공되는 파일은 32bit이므로 64bit를 설치하시려면 그냥 nuget에서 다운로드 받는 것이 편합니다. (nupkg 강제 압축 해제) 파일은 3개의 헤더 파일과 6개의 라이브러리 파일이 있습니다.

### 1.3 CLAPACK를 위한 CMakeLists.txt

저는 dll이 있어서 종속성이 있는 OpenBLAS 방식의 BLAS/LAPACK이 아닌 CLAPACK 64bit를 다운로드 받아 설치하였으며 LAPACK과 CLAPACK의 차이점은 LAPACK 함수명 뒤에 _가 추가되어 있는 것입니다.

.h 파일은 include 폴더에 넣어놓고, .lib 파일은 library 폴더에 넣어놓고 CMakeLists.txt 파일은 다음과 같이 작성하였습니다.

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(blas_excersize VERSION 0.1.0)

include(CTest)
enable_testing()

# Header 파일 경로
include_directories(include)

# compile될 대상
add_executable(blas_excersize main.cpp)

# Library 파일 읽어온 후 프로그램에 링크
file(GLOB LAPACK_Libraries library/*.lib)
target_link_libraries(blas_excersize ${LAPACK_Libraries})

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_V
```

### 1.4 CLAPACK tutorial

Reference : https://netlib.org/lapack/explore-html/index.html

LAPACK의 함수는 XYYZZ 혹은 XYYZZZ 형태의 5자 혹은 6자 입니다. 

X의 경우는 다음과 같습니다.

- S : Real
- D : Double Precision
- C : Complex
- Z : Complex*16 or Double Complex

YY의 경우는 종류가 너무 많으므로 아래 하단에 appendix에 기재합니다. ZZZ의 경우는 수행하는 알고리즘 명입니다. ZZZ의 경우는 작성조차 하기 힘들 정도로 정리할 수 없기 때문에 기재하지 않았습니다.

예를 들어 최소제곱법을 해결할 수 있는 LAPACK 함수 원형 중 한개인 DGELS은 Real 값에 General한 문제이며 LS(Least Squere) 입니다. DGELS를 사용하기 전에 계산에 사용할 변수는 다음과 같습니다. (Studentized Residual에서 사용했던 변수구성입니다.)

$$
X = \begin{bmatrix}1 & 1\\1 & 2\\1 & 3\\1 & 10\\\end{bmatrix}\qquad
Y = \begin{bmatrix}2.1\\3.8\\5.2\\2.1\\\end{bmatrix}
$$

위 수식의 실제 결과값은 $y=3.8-0.13x$입니다. 이를 코드로 작성해보겠습니다.

Reference
- https://netlib.org/lapack/explore-html/d7/d3b/group__double_g_esolve_ga225c8efde208eaf246882df48e590eac.html
- https://www.intel.com/content/www/us/en/develop/documentation/onemkl-lapack-examples/top/least-squares-and-eigenvalue-problems/linear-least-squares-lls-problems/gels-function/dgels-example/dgels-example-c.html

```c
#include <iostream>
#include <f2c.h>
#include <clapack.h>

#define M 4
#define N 2
#define NRHS 1

void print_matrix(char *desc, int m, int n, double *a, int lda);

int main(int argc, char **argv) {
    char trans = 'N';     // 전치 행렬 할지 말지 결정합니다.
    integer m = M;        // A 행렬의 행
    integer n = N;        // A 행렬의 열
    integer nrhs = NRHS;  // B 행렬의 열

    // A 행렬을 만들때 변수 위치에 신경써야합니다.
    // A 행렬은 사용 후 QR factorization를 볼 수 있는 변수가 기입됩니다.
    doublereal a[M * N] = {1, 1, 1, 1, 1, 2, 3, 10};
    integer lda = max(1, M);  // 이렇게 쓰라고 문서에 작성되어있습니다.

    // B 행렬을 만들때 변수 위치에 신경써야합니다.
    // B 행렬은 사용 후 Least Squere 답안이 작정됩니다.
    doublereal b[M * NRHS] = {2.1, 3.8, 5.2, 2.1};
    integer ldb = max(max(1, M), N);  // MAX(1, M, N);

    doublereal workOpt;  // lwork 값을 얻기 위해 임의로 선언 및 정의한 변수
    doublereal *work;    // 연산하기 위한 변수
    integer lwork = -1;  // lwork값을 -1로 하면 dgels_가 최적의 lwork를 얻기위한 함수로 동작합니다.
    integer info;        // 정상 동작 여부 확인

    // lwork 최적값을 workOpt로 얻기위해서 함수를 동작시킵니다.
    dgels_(&trans, &m, &n, &nrhs, a, &lda, b, &ldb, &workOpt, &lwork, &info);

    // lwork의 최적값을 구해서 work에 그 크기에 맞는 공간을 할당합니다.
    lwork = (integer)workOpt;
    work = (doublereal *)malloc(max(1, lwork) * sizeof(doublereal));

    // 실제로 연산을 하는 부분입니다.
    dgels_(&trans, &m, &n, &nrhs, a, &lda, b, &ldb, work, &lwork, &info);

    print_matrix("Least squares solution", n, nrhs, b, ldb);
    print_matrix("Details of QR factorization", m, n, a, lda);

    free(work);

    return 0;
}

void print_matrix(char *desc, int m, int n, double *a, int lda) {
    int i, j;
    printf("\n %s\n", desc);
    for (i = 0; i < m; i++) {
        for (j = 0; j < n; j++) printf(" %6.2f", a[i + j * lda]);
        printf("\n");
    }
}
```

위 코드는 실제 연습을 하지 않으면 알기 어려운 함수원형입니다. 처음 접할때 엄청 이해가 안됬는데, 공유해 드린 링크에는 각기 어떻게 사용할지 작성되어 있으며 적당한 예제 또한 포함되어있습니다.

## 2. Eigen

Eigen은 C++로 만들어있으며 소스 그대로 사용하면 됩니다. 단, BLAS/LAPACK과 함께 include 하게되면 서로 충돌이 일어나니 한개만 사용해야합니다.

### 1.1 Eigen를 설치하는 방법

홈페이지(https://eigen.tuxfamily.org/)에서 받으면 github 형태의 구조로 압축되어 있는데, 그 폴더에서 Eigen 폴더만 include 폴더로 복사 후 이동하였습니다. 

### 1.2 Eigen를 위한 CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(blas_excersize VERSION 0.1.0)

include(CTest)
enable_testing()

# Eigen 폴더 경로 추가 (헤더가 없어서 그냥 사용하면 됩니다.)
include_directories(include)
add_executable(blas_excersize main.cpp)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)

```

### 1.3 Eigen tutorial

CLAPACK을 사용할 때와 동일한 변수를 계산해보겠습니다.

$$
X = \begin{bmatrix}1 & 1\\1 & 2\\1 & 3\\1 & 10\\\end{bmatrix}\qquad
Y = \begin{bmatrix}2.1\\3.8\\5.2\\2.1\\\end{bmatrix}
$$

위 수식의 실제 결과값은 $y=3.8-0.13x$입니다. 이를 코드로 작성해보겠습니다.

Reference
- https://eigen.tuxfamily.org/dox-3.3/group__LeastSquares.html

```c
#include <iostream>
#include <Eigen/Dense>

using Eigen::MatrixXd;

int main() {
    MatrixXd a(4, 2);
    a << 1, 1,
        1, 2,
        1, 3,
        1, 10;

    MatrixXd b(4, 1);
    b << 2.1,
        3.8,
        5.2,
        2.1;

    // 상황에 따라 사용할 수 있는 함수가 달라집니다.
    MatrixXd result = a.colPivHouseholderQr().solve(b);

    std::cout << result << std::endl;
}
```

확실히 LAPACK 보다 구조적으로 깔끔한 것을 확인 할 수 있습니다. 이를 통하여 C/C++로 빠른 선형대수학 연산이 가능하도록 연습하면 됩니다.

## Appendix

### 1. LAPACK YY, Matrix types in the LAPACK naming scheme

Reference : https://netlib.org/lapack/lug/node24.html


- BD :	bidiagonal
- DI :	diagonal
- GB :	general band
- GE :	general (i.e., unsymmetric, in some cases rectangular)
- GG :	general matrices, generalized problem (i.e., a pair of general matrices)
- GT :	general tridiagonal
- HB :	(complex) Hermitian band
- HE :	(complex) Hermitian
- HG :	upper Hessenberg matrix, generalized problem (i.e a Hessenberg and a triangular matrix)
- HP :	(complex) Hermitian, packed storage
- HS :	upper Hessenberg
- OP :	(real) orthogonal, packed storage
- OR :	(real) orthogonal
- PB :	symmetric or Hermitian positive definite band
- PO :	symmetric or Hermitian positive definite
- PP :	symmetric or Hermitian positive definite, packed storage
- PT :	symmetric or Hermitian positive definite tridiagonal
- SB :	(real) symmetric band
- SP :	symmetric, packed storage
- ST :	(real) symmetric tridiagonal
- SY :	symmetric
- TB :	triangular band
- TG :	triangular matrices, generalized problem (i.e., a pair of triangular matrices)
- TP :	triangular, packed storage
- TR :	triangular (or in some cases quasi-triangular)
- TZ :	trapezoidal
- UN :	(complex) unitary
- UP :	(complex) unitary, packed storage