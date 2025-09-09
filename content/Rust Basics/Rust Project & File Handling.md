Rust does not use **header files** like C or C++; instead, it organizes code with modules and uses the `mod` and `use` keywords to manage visibility and imports between code files and crates.[reddit+2](https://www.reddit.com/r/learnrust/comments/gtwitw/header_files_in_rust/)

## Header Files vs. Rust Modules

- Header files in C/C++ (.h) are used for **declarations** of functions, structs, and classes which are then implemented in corresponding source files (.c/.cpp).[cel.brown+1](https://cel.cs.brown.edu/crp/idioms/encapsulation/headers.html)
- Rust combines both declaration and implementation in a single `.rs` file, controlling visibility via the `pub` keyword, and does not require forward declarations or separate header files.[rust-lang+1](https://users.rust-lang.org/t/converting-simple-idiomatic-c-to-idiomatic-rust/44972)

## Importing Code in Rust

- To share code between Rust files, define a module (e.g., `mod person;`) and import it using the `use` statement (e.g., `use person::*;`).[stackoverflow+1](https://stackoverflow.com/questions/45519176/how-do-i-use-or-import-a-local-rust-file)
- The Rust compiler recognizes submodules based on file organization (e.g., `main.rs` and `person.rs`), similar to Python modules but without file extension.[stackoverflow](https://stackoverflow.com/questions/45519176/how-do-i-use-or-import-a-local-rust-file)

## How Visibility and API Exposure Work

- Using `pub` makes structs, functions, or traits public for other modules or crates to access.[cel.brown](https://cel.cs.brown.edu/crp/idioms/encapsulation/headers.html)
- Only public items within a module are accessible when imported; private fields and methods remain hidden, ensuring encapsulation without using header files.[cel.brown](https://cel.cs.brown.edu/crp/idioms/encapsulation/headers.html)

## FFI and C Header Integration

- To use C header files in Rust, tools like `bindgen` or libraries such as `c_import` help generate Rust bindings to C functions and types defined in headers.[stackoverflow+1](https://stackoverflow.com/questions/73835917/how-to-use-a-single-header-library-c-file-in-rust)
- For exposing Rust code as a C ABI (acting like "reverse header"), annotations like `pub extern "C" fn` and `#[repr(C)]` on structs are used.[reddit](https://www.reddit.com/r/rust/comments/pa3fns/is_it_possible_to_implement_an_existing_c/)

## Common Questions

- There is no direct equivalent for "declare now, implement later" as in C++ header files. However, functions can be stubbed out with macros like `todo!()` or `unimplemented!()` during development.[reddit](https://www.reddit.com/r/learnrust/comments/gtwitw/header_files_in_rust/)
- Documentation tools such as `cargo doc` can generate overviews of all public APIs, similar to reading a header file's summary of declarations.[reddit](https://www.reddit.com/r/learnrust/comments/gtwitw/header_files_in_rust/)

**Key differences:** Rust does not use header files; instead, it manages code sharing with module imports and public API declarations.[rust-lang+2](https://users.rust-lang.org/t/converting-simple-idiomatic-c-to-idiomatic-rust/44972)






1. [https://www.reddit.com/r/learnrust/comments/gtwitw/header_files_in_rust/](https://www.reddit.com/r/learnrust/comments/gtwitw/header_files_in_rust/)
2. [https://users.rust-lang.org/t/how-to-do-header-files-separate-compilation-in-rust/87537](https://users.rust-lang.org/t/how-to-do-header-files-separate-compilation-in-rust/87537)
3. [https://stackoverflow.com/questions/73835917/how-to-use-a-single-header-library-c-file-in-rust](https://stackoverflow.com/questions/73835917/how-to-use-a-single-header-library-c-file-in-rust)
4. [https://users.rust-lang.org/t/rust-and-lack-of-header-files/132176](https://users.rust-lang.org/t/rust-and-lack-of-header-files/132176)
5. [https://cel.cs.brown.edu/crp/idioms/encapsulation/headers.html](https://cel.cs.brown.edu/crp/idioms/encapsulation/headers.html)
6. [https://docs.rs/c_import](https://docs.rs/c_import)
7. [https://www.reddit.com/r/rust/comments/pa3fns/is_it_possible_to_implement_an_existing_c/](https://www.reddit.com/r/rust/comments/pa3fns/is_it_possible_to_implement_an_existing_c/)
8. [https://immunant.com/blog/2019/12/header_merging/](https://immunant.com/blog/2019/12/header_merging/)
9. [https://google.github.io/autocxx/tutorial.html](https://google.github.io/autocxx/tutorial.html)
10. [https://stackoverflow.com/questions/45519176/how-do-i-use-or-import-a-local-rust-file](https://stackoverflow.com/questions/45519176/how-do-i-use-or-import-a-local-rust-file)
11. [https://docs.rust-embedded.org/book/interoperability/c-with-rust.html](https://docs.rust-embedded.org/book/interoperability/c-with-rust.html)
12. [https://www.reddit.com/r/rust/comments/ozhcqa/dont_you_love_headers/](https://www.reddit.com/r/rust/comments/ozhcqa/dont_you_love_headers/)
13. [https://stackoverflow.com/questions/69058366/calling-c-from-rust-with-external-headers-in-c-file](https://stackoverflow.com/questions/69058366/calling-c-from-rust-with-external-headers-in-c-file)
14. [https://users.rust-lang.org/t/defining-structs-enums-based-on-h-file-input/41240](https://users.rust-lang.org/t/defining-structs-enums-based-on-h-file-input/41240)
15. [https://users.rust-lang.org/t/converting-simple-idiomatic-c-to-idiomatic-rust/44972](https://users.rust-lang.org/t/converting-simple-idiomatic-c-to-idiomatic-rust/44972)
16. [https://internals.rust-lang.org/t/a-rust-rust-declarative-abi/15381](https://internals.rust-lang.org/t/a-rust-rust-declarative-abi/15381)