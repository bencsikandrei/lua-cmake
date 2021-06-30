# Lua - built with CMake

Build lua using CMake.

*Note* This is the result of my search for a good CMake build for lua. Wanted something automated, something easy to use, easy to install, etc... I haven't found one. Took ideas from a bunch of places and I hope I mentioned all of them.

The minimum requirements for this projects are:

1) a C (or C++) compiler which supports C99 and C++98
2) CMake >= 3.10
3) a build tool (eg.: make, ninja, ...)

## Building it

To build Lua, we first download it from the official FTP.
The supported version, for now, is only 5.3.5. Can easily be extended.

The build script downloads the sources for you and then uses those to create the lua library (or tools).

The example is for *nix based systems, but can be extended:

```sh
# in the root
mkdir build && cd build
cmake -Bbuild -S. # change the generator at will, say `-GNinja`
cmake --build build --target all

# when done you can install it
cmake --build build --target install # will also make CMake config files
```

## Options

When building you can make several packages by using the available options:

`LUA_COMPILE_AS_CPP` will compile lua as a c++ library (for the implications of this, take a look at: http://lua-users.org/wiki/BuildingLua)

`LUA_SKIP_TOOLS` skip `luac` and `lua` (default: ON)

`LUA_PACKAGE_TEST` run a simple package test

## Using it

You can add this as a subfolder, you can install it and use it via the `FindLua.cmake` that ships with cmake, of even set `Lua_ROOT` and use the `Lua-config.cmake` that this project creates.

When using it as a subproject or with `find_package(Lua CONFIG)` you also get `Lua::lua` as a target to link to. Otherwise, for portability, use: `LUA_LIBRARIES` and `LUA_INCLUDE_DIR`.

## CMake files

When installing the project we also create Lua-config.cmake for `CONFIG` based `find_package`.

## Inspired from

https://github.com/ThePhD/sol2
https://github.com/conan-io/conan-center-index/tree/master/recipes/lua
https://raw.githubusercontent.com/microsoft/vcpkg/master/ports/lua/CMakeLists.txt

## TODO

1) Provide multiple versions of Lua
2) Set the version of Lua (also, on linux set the .so version with that)
3) Add a CI
