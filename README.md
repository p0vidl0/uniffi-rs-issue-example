# RC

> Example repo for issue [#1300](https://github.com/mozilla/uniffi-rs/issues/1300)

## Configure rust environment

* Install android NDK https://developer.android.com/ndk/downloads
* Fix error like `ld: error: unable to find library -lgcc`: https://github.com/rust-lang/rust/pull/85806#issuecomment-1096266946

```shell
cargo install cargo-ndk
rustup default nightly
rustup default stable
rustup update
rustup component add rust-src --toolchain nightly --target aarch64-linux-android
```

## Build
```shell
uniffi-bindgen scaffolding ./src/rsl.udl
uniffi-bindgen generate --language kotlin ./src/rsl.udl
cargo +nightly build --target aarch64-linux-android -Zbuild-std --verbose --release
```

## The issue
Android app falls with the error:
```
E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.example.myapplication, PID: 14718
    java.lang.UnsatisfiedLinkError: Error looking up function 'rsl_3ad6_mul2': undefined symbol: rsl_3ad6_mul2
        at com.sun.jna.Function.<init>(Function.java:252)
        at com.sun.jna.NativeLibrary.getFunction(NativeLibrary.java:600)
        ...
```

Exported FFi interface function name differs:
* `rsl_d7ce_mul2` in `librsl.so` binary
* `rsl_3ad6_mul2` in `rsl.kt` file.

Generated `rsl.kt` file part:
```
fun `mul2`(`a`: UInt, `b`: UInt): UInt {
    return FfiConverterUInt.lift(
    rustCall() { _status ->
    _UniFFILib.INSTANCE.rsl_3ad6_mul2(FfiConverterUInt.lower(`a`), FfiConverterUInt.lower(`b`), _status)
})
```

Compiled library `librsl.so`:
```shell
nm -gD librsl.so

                 U __cxa_atexit
                 U __cxa_finalize
                 U __errno
                 U __register_atfork
                 U __sF
                 U abort
                 U calloc
                 U clock_gettime
                 U close
                 U dl_iterate_phdr
0000000000042c6c T ffi_rsl_d7ce_rustbuffer_alloc
0000000000042c78 T ffi_rsl_d7ce_rustbuffer_free
0000000000042c70 T ffi_rsl_d7ce_rustbuffer_from_bytes
0000000000042c7c T ffi_rsl_d7ce_rustbuffer_reserve
                 U fflush
                 U fprintf
                 U free
                 U fstat
                 U fwrite
                 U getcwd
                 U getenv
                 U malloc
                 U memalign
                 U memcmp
                 U memcpy
                 U memmove
                 U memset
                 U mmap
                 U munmap
                 U open
                 U pthread_getspecific
                 U pthread_key_create
                 U pthread_key_delete
                 U pthread_rwlock_rdlock
                 U pthread_rwlock_unlock
                 U pthread_rwlock_wrlock
                 U pthread_setspecific
                 U realloc
                 U realpath
0000000000042c80 T rsl_d7ce_mul2
                 U stat
                 U strerror_r
                 U strlen
                 U syscall
0000000000043158 T uniffi_rustbuffer_alloc
00000000000431b4 T uniffi_rustbuffer_free
0000000000043188 T uniffi_rustbuffer_from_bytes
00000000000431d8 T uniffi_rustbuffer_reserve
                 U write
                 U writev
```