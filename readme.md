# QuickJSPP

This is a modification of [FTK/quickjspp](https://github.com/ftk/quickjspp)

* Uses FTK's `sharedptr` branch (aka quickjspp v2)
* The bundled `quickjs` library code is the latest from c-smile/quickjspp with Windoes MSVC build support
* Added support for compiling with `meson` and `ninja` 

## Compiling

* Clone the repo
* `meson build`
* `cd build`
* Configure your build. Suggest `meson configure -Ddefault_library=static -Db_ndebug=if-release -Dbuildtype=release -Ddebug=false -Dstrip=true`
* `ninja -C .`

Original readme below.

---

QuickJSPP is QuickJS wrapper for C++. It allows you to easily embed Javascript engine into your program.

QuickJS is a small and embeddable Javascript engine. It supports the ES2020 specification including modules, asynchronous generators and proxies. More info: <https://bellard.org/quickjs/>

## Example
```cpp
#include "quickjspp.hpp"
#include <iostream>

class MyClass
{
public:
    MyClass() {}
    MyClass(std::vector<int>) {}

    double member_variable = 5.5;
    std::string member_function(const std::string& s) { return "Hello, " + s; }
};

void println(qjs::rest<std::string> args) {
    for (auto const & arg : args) std::cout << arg << " ";
    std::cout << "\n";
}

int main()
{
    qjs::Runtime runtime;
    qjs::Context context(runtime);
    try
    {
        // export classes as a module
        auto& module = context.addModule("MyModule");
        module.function<&println>("println");
        module.class_<MyClass>("MyClass")
                .constructor<>()
                .constructor<std::vector<int>>("MyClassA")
                .fun<&MyClass::member_variable>("member_variable")
                .fun<&MyClass::member_function>("member_function");
        // import module
        context.eval(R"xxx(
            import * as my from 'MyModule';
            globalThis.my = my;
        )xxx", "<import>", JS_EVAL_TYPE_MODULE);
        // evaluate js code
        context.eval(R"xxx(
            let v1 = new my.MyClass();
            v1.member_variable = 1;
            let v2 = new my.MyClassA([1,2,3]);
            function my_callback(str) {
              my.println("at callback:", v2.member_function(str));
            }
        )xxx");

        // callback
        auto cb = (std::function<void(const std::string&)>) context.eval("my_callback");
        cb("world");
    }
    catch(qjs::exception)
    {
        auto exc = context.getException();
        std::cerr << (std::string) exc << std::endl;
        if((bool) exc["stack"])
            std::cerr << (std::string) exc["stack"] << std::endl;
        return 1;
    }
}
```

## Installation
QuickJSPP is header-only - put quickjspp.hpp into your include search path.
Compiler that supports C++17 or later is required.
The program needs to be linked against QuickJS.
Sample CMake project files, and a meson build file,
 are provided.

## License
QuickJSPP is licensed under [CC0](https://creativecommons.org/publicdomain/zero/1.0/). QuickJS is licensed under [MIT](https://opensource.org/licenses/MIT).
