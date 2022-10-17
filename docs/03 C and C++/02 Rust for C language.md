# Rust for C language

C에서 구현하기 힘든 로직들을 Rust에서 구현하여 C로 읽어들이는 방법을 사용하면 효율적으로 엔지니어링에 도움되는 프로그램을 만들 수 있습니다. 대표적으로 Text Parsing이 있습니다. Rust의 FFI를 통해 C library를 만드는 것은 그리 어렵지 않으나 상당히 번거로운 작업입니다. 다만 여러 개발상에 유리합니다.

굳이 C/C++로 구현하는것이 유리하지 않은 로직은 Rust로 구현하고자 합니다. 이를 위한 방법을 정리해둡니다.

단, 221013자로 cbindgen을 사용하는 것은 권장하지 않습니다. release로 build시에 V3에서 트로이목마로 cbindgen이 생성하는 exe 파일을 잡습니다.

Reference
- http://jakegoulding.com/rust-ffi-omnibus/basics/
- https://snacky.blog/en/string-ffi-rust.html
- https://github.com/alexcrichton/rust-ffi-examples
- https://github.com/shepmaster/rust-ffi-omnibus
- https://stackoverflow.com/questions/66196972/how-to-pass-a-reference-pointer-to-a-rust-struct-to-a-c-ffi-interface

## 1. Cargo.toml 설정

lib 항목에 `cdylib`항목을 추가합니다. `staticlib`으로 build하면 생각보다 용량이 큽니다. 그리고 `cdylib` build를 하게되면 `*.dll` 파일과 `*.dll.lib` 파일이 생성되는데, 두 파일이 전부 필요합니다. 또한, cbindgen을 사용하지 않기 때문에 직접 header 파일일 작성해야합니다. 하지만 cbindgen document는 header 파일 작성하는데 도움이되니 참고하도록 합니다. 아래 `[lib]`를 추가하시면 됩니다.

```toml
# Cargo.toml 일부
[lib]
name = "rust_cdynlib"
crate-type = ["cdylib"]      # Creates dynamic lib
# crate-type = ["staticlib"] # Creates static lib
```

## 2. 간단한 예제 : int, float, char

실제로 이런 자료형을 넘길일은 없지만 개념을 익히기 위해 작성합니다.

```rs
// lib.rs
use std::os::raw::{c_char, c_double, c_int};

/// 정수를 받아서 정수를 리턴하는 함수
#[no_mangle]
pub extern "C" fn rust_int(x: c_int, y: c_int) -> c_int {
    x + y
}

/// 실수를 받아서 실수를 리턴하는 함수
#[no_mangle]
pub extern "C" fn rust_float(x: c_double, y: c_double) -> c_double {
    x + y
}

/// 문자 두개를 받아 문자 한개를 리턴하는 함수
#[no_mangle]
pub extern "C" fn rust_char(x: c_char, y: c_char, z: c_int) -> c_char {
    if z == 0 {
        x
    } else {
        y
    }
}
```

위 lib.rs 파일을 cargo build --lib 명령을 통해 build 하였습니다. 제 프로젝트 명은 rust_cdylib이니 `rust_cdylib.dll`과 `rust_cdylib.dll.lib` 파일이 생겼습니다. 위에서 언급한대로 `*.lib`는 종속성 설정을 해야하며, `*.dll`은 exe가 실행될 위치에 복사해주고 `CMakeLists.txt`파일을 작성하면 다음과 같습니다. 단, `*.lib` 파일은 root/libs 폴더에 넣어놨습니다.

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(c_simple VERSION 0.1.0)

include(CTest)
enable_testing()

add_executable(c_simple main.c)

# *.lib 종속성 연결
file(GLOB libs lib/*.lib)
target_link_libraries(c_simple ${libs})

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

C 언어에 연결하기 위해서 가장 쉬운 방법은 extern으로 연결하는 방법입니다. (**단, C++에는 이 방법으로 연결을 할 수 없습니다.**)

```c
#include <stdio.h>
#include <inttypes.h>

extern int32_t rust_int(int32_t, int32_t);
extern double rust_float(double, double);
extern char rust_char(char, char, int32_t);

int main(int argc, char** argv) {
    int _int = rust_int(1, 5);
    double _float = rust_float(1.1, 5.1);
    char _char1 = rust_char('A', 'B', 0);
    char _char2 = rust_char('A', 'B', 1);

    printf("from rust : %d, %f, %c, %c", _int, _float, _char1, _char2);

    return 0;
}
```

C++에도 연결을 하기 위해선 함수를 아래와 같이 수정해야합니다.

```c++
extern "C" int32_t rust_int(int32_t, int32_t);
extern "C" double rust_float(double, double);
extern "C" char rust_char(char, char, int32_t);
```

C/C++ 범용성을 위해 매크로를 동원하면 다음과 같이 작성할 수 있습니다. 아래 설명코드에선 이 매크로를 항상 적용한다고 가정합니다.

```c++
#ifdef __cplusplus
extern "C" {
#endif

int32_t rust_int(int32_t, int32_t);
double rust_float(double, double);
char rust_char(char, char, int32_t);

#ifdef __cplusplus
}
#endif
```

## 3. 문자열 예제 : String

문자열은 char을 배열로 할당하기 때문에 많은 메모리가 소비됩니다. 그래서 해당 메모리를 할당 해제해야하는데, rust에서 만든 메모리 할당은 rust에서 해제 해야합니다. 그래야 안전합니다.

```rust
// lib.rs
use std::ffi::{CStr, CString};
use std::os::raw::c_char;

/// 문자열을 받아 수정 후 수정된 문자열 리턴
#[no_mangle]
pub extern "C" fn rust_string(s: *const c_char) -> *mut c_char {
    let _s = s.clone(); // C/C++ 에서 올 데이터 안전하게 처리하기 위해 복사
    let text = unsafe { CStr::from_ptr(_s).to_str().unwrap() };
    let mut text = String::from(&text.to_string());

    text.push_str(" : from rust!");

    let ptr = CString::new(text).unwrap();
    ptr.into_raw()
}

/// 러스트에서 할당한 메모리는 반드시 러스트에서 해제해야 합니다.
#[no_mangle]
pub extern "C" fn rust_string_free(s: *mut c_char) {
    unsafe {
        if s.is_null() {
            return;
        }
        CString::from_raw(s) // 소유권을 넘기면서 함수가 알아서 메모리 할당 해제
    };
}
```

위에서 메모리 할당 해제 함수 만들어 놓은 것을 반드시 C/C++에서 사용하도록 합니다.

```cpp
#include <stdio.h>

#ifdef __cplusplus
extern "C" {
#endif

char *rust_string(const char *);
void rust_string_free(char *);

#ifdef __cplusplus
}
#endif

int main(int argc, char **argv) {
    // 문자열 생성
    char *string = rust_string("Hello World");

    printf("%s", string);

    // 문자열 메모리 해제
    rust_string_free(string);

    return 0;
}
```

## 4. 배열 예제 : Vector(Array)

문자열과 동일하게 Vector로 할당된 메모리는 Rust에서 지우는 형태로 구현을 합니다. 다만 Pointer를 넘길때 요령이 다릅니다. 해당 내용참고하시기 바랍니다.

```rust
// lib.rs
use std::os::raw::{c_double, c_uint};

/// 배열을 받을 array pointer와 만들 숫자 size를 받아 0.1~ 부터의 실수를 return 받습니다.
#[no_mangle]
pub extern "C" fn rust_array(vec: *mut *mut c_double, size: c_uint) -> c_uint {
    // 실수 array를 만들기 위한 임의의 코드입니다.
    let mut counted: Vec<_> = (0..).take(size as usize).map(|x| x as f64 + 0.1).collect();
    counted.shrink_to_fit(); // length와 capacity를 동일하게 맞추기 위해 실행합니다.

    let len = counted.len();
    unsafe { *vec = counted.as_mut_ptr() }; // 포인터 만들어주기
    std::mem::forget(counted); // forget을 안하면 함수 끝나는 즉시 메모리에서 삭제됩니다.

    len as c_uint
}

/// 러스트에서 할당한 메모리는 반드시 러스트에서 해제해야 합니다.
#[no_mangle]
pub extern "C" fn rust_array_free(arr: *mut c_double, size: c_uint) {
    unsafe {
        if arr.is_null() {
            return;
        }
        let size = size as usize;
        Vec::from_raw_parts(arr, size, size) // 소유권을 넘기면서 함수가 알아서 메모리 할당 해제
    };
}
```

소유권 해제는 String 예제와 컨셉이 유사합니다. 사용 방법도 유사합니다. 다만 포인터를 건네줘야하므로 약간의 코드가 더 필요합니다.

```cpp
#include <stdio.h>
#include <inttypes.h>

#ifdef __cplusplus
extern "C" {
#endif

uint32_t rust_array(double **vec, uint32_t size);
void rust_array_free(double *vec, uint32_t size);

#ifdef __cplusplus
}
#endif

int main(int argc, char **argv) {
    double *vec;  // 저장할 배열 만들기
    uint32_t len = rust_array(&vec, 10);

    for (uint32_t i = 0; i < len; i++) {
        printf("%lf, ", vec[i]);
    }

    // 반드시 배열 할당 해제
    rust_array_free(vec, len);

    return 0;
}
```

## 5. 구조체 예제 1 : struct

구조체를 Pointer 없이 그냥 raw 값으로 처리하는 방법부터 작성하겠습니다.

```rust
// lib.rs
#[repr(C)]  // 이 코드를 넣어야 C언어에 대응되는 구조체가 됩니다.
pub struct Tuple {
    x: f64,
    y: f64,
}

#[no_mangle]
pub extern "C" fn rust_tuple(tuple: Tuple) -> Tuple {
    let mut tuple = tuple;
    tuple.x += 1.0;
    tuple.y += 2.0;

    tuple
}
```

당연히 사용법도 간단합니다.

```cpp
#include <stdio.h>

typedef struct {
    double x;
    double y;
} tuple_t;

#ifdef __cplusplus
extern "C" {
#endif

tuple_t rust_tuple(tuple_t);

#ifdef __cplusplus
}
#endif

int main(int argc, char **argv) {
    tuple_t tuple;
    tuple.x = 10;
    tuple.y = 20;

    tuple_t new_tuple = rust_tuple(tuple);

    printf("%lf, %lf", new_tuple.x, new_tuple.y);

    return 0;
}
```

## 5. 구조체 예제 2 : *struct

지금까지는 그 개체를 직접 Pointer로 변경하였다면, 이제는 좀 더 rust하게 Box에 넣어서 pointer를 만들었습니다. 이와같은 방식을 사용한다면, 구조체 중심의 포인터를 rust로 처리하면서 기존 rust 코드를 무리 없이 치환하여 사용할 수 있습니다.

```rust
// lib.rs
// Reference : https://doc.rust-lang.org/std/boxed/struct.Box.html
#[repr(C)]
pub struct Tuple {
    x: f64,
    y: f64,
}

#[no_mangle]
pub extern "C" fn rust_tuple() -> *mut Tuple {
    let instance = Tuple { x: 0.0, y: 1.0 };
    let ptr = Box::new(instance); // Box에 넣어서 Pointer를 만들 준비를 합니다.
    Box::into_raw(ptr) // as_ptr + std::mem::forget을 동시에 쓰는 효과입니다.
}

#[no_mangle]
pub extern "C" fn rust_tuple_free(instance: *mut Tuple) {
    unsafe {
        Box::from_raw(instance); // into_raw를 할당하는 방법입니다.
    }
}
```

사용법은 기존의 free 함수를 사용했던 방식과 동일합니다.

```cpp
#include <stdio.h>

typedef struct {
    double x;
    double y;
} tuple_t;

#ifdef __cplusplus
extern "C" {
#endif

tuple_t* rust_tuple();
void rust_tuple_free(tuple_t*);

#ifdef __cplusplus
}
#endif

int main(int argc, char** argv) {
    tuple_t* new_tuple = rust_tuple();

    printf("%lf, %lf", new_tuple->x, new_tuple->y);

    rust_tuple_free(new_tuple);

    return 0;
}
```