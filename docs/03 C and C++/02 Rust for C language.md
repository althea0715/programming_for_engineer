# Rust for C language

C에서 구현하기 힘든 로직들을 Rust에서 구현하여 C로 읽어들이는 방법을 사용하면 효율적으로 엔지니어링에 도움되는 프로그램을 만들 수 있습니다. 대표적으로 Text Parsing이 있습니다. Rust의 FFI를 통해 C library를 만드는 것은 그리 어렵지 않으나 상당히 번거로운 작업입니다. 다만 여러 개발상에 유리합니다.

굳이 C/C++로 구현하는것이 유리하지 않은 로직은 Rust로 구현하고자 합니다. 이를 위한 방법을 정리해둡니다.

단 221013자로 cbindgen을 사용하는 것은 권장하지 않습니다. release로 build시에 V3에서 트로이목마로 exe 파일을 잡습니다.

Reference
- http://jakegoulding.com/rust-ffi-omnibus/basics/
- https://snacky.blog/en/string-ffi-rust.html
- https://github.com/alexcrichton/rust-ffi-examples

## 1. Cargo.toml 설정

lib 항목에 `cdylib`항목을 추가합니다. `staticlib`으로 build하면 생각보다 용량이 큽니다. 그리고 `cdylib` build를 하게되면 `*.dll` 파일과 `*.dll.lib` 파일이 생성되는데, 두 파일이 전부 필요합니다. 또한, cbindgen을 사용하지 않기 때문에 직접 header 파일일 작성해야합니다. 하지만 cbindgen document는 header 파일 작성하는데 도움이되니 참고하도록 합니다. 아래 `[lib]`항목만 참고하면 됩니다.

```toml
# Cargo.toml
[package]
name = "rust_cdylib"
version = "0.1.0"
edition = "2021"

[lib]
name = "rust_cdynlib"
crate-type = ["cdylib"]      # Creates dynamic lib
# crate-type = ["staticlib"] # Creates static lib

[dependencies]
```

## 2. 간단한 예제 모음

이렇게 코드 작성하는 것을 추천할 수는 없지만, 이해를 위해 반드시 넘어가야하는 부분입니다. int, float는 작은 용량을 차지하는 형태이나, string과 array는 용량을 많이 차지할 가능성이 많습니다. 그래도 일반적인 arguments와 return 형태를 잘 보면서 참고하시기 바랍니다.

```rs
// lib.rs
use std::ffi::{CStr, CString};
use std::os::raw::{c_char, c_float, c_int};

/// 정수를 받아서 정수를 리턴하는 함수
#[no_mangle]
pub extern "C" fn rust_int(x: c_int, y: c_int) -> c_int {
    x + y
}

/// 실수를 받아서 실수를 리턴하는 함수
#[no_mangle]
pub extern "C" fn rust_float(x: c_float, y: c_float) -> c_float {
    x + y
}

/// 문장을 받아서 문장을 리턴하는 함수
#[no_mangle]
pub extern "C" fn rust_string(text: *const c_char) -> *const c_char {
    let text = unsafe { CStr::from_ptr(text).to_str().unwrap() };
    let text = format!("Rust : {}", text);
    let text = CString::new(text).unwrap();

    let ptr = text.as_ptr();

    std::mem::forget(text); // 메모리가 해지되면 안되므로 메모리 정리 안합니다.

    ptr
}

// 정수 2개를 받아서 Array를 리턴하는 함수
#[no_mangle]
pub extern "C" fn rust_array(x: c_int, y: c_int) -> *mut c_int {
    let mut master = vec![x, y];
    let ptr = master.as_mut_ptr();

    std::mem::forget(master); // 메모리가 해지되면 안되므로 메모리 정리 안합니다.

    ptr
}
```


