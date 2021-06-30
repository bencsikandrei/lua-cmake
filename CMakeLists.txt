cmake_minimum_required(VERSION 3.10)

project(LuaCMake VERSION 1.0.0 LANGUAGES C CXX
  DESCRIPTION "CMake build for Lua"
)

set(_Lua "Lua")

option(LUA_COMPILE_AS_CPP "Compile the source code as C++" OFF)
option(LUA_SKIP_TOOLS "Skip installing tools" ON)
option(LUA_PACKAGE_TEST "Run package test" OFF)

set(LUA_VERSION_TO_BUILD "5.3.5")

set(_LUA_ROOT_DIR "${CMAKE_CURRENT_BINARY_DIR}/lua-src")

if(NOT EXISTS ${CMAKE_CURRENT_BINARY_DIR}/lua-download)
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/LuaDownload.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/lua-download/CMakeLists.txt)
  execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lua-download"
  )
  execute_process(COMMAND "${CMAKE_COMMAND}" --build .
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lua-download"
  )
endif()

set(_LUA_SOURCE_DIR "${_LUA_ROOT_DIR}/src")
set(_LUA_LIB_SRC
  "${_LUA_SOURCE_DIR}/lapi.c"
  "${_LUA_SOURCE_DIR}/lcode.c"
  "${_LUA_SOURCE_DIR}/lctype.c"
  "${_LUA_SOURCE_DIR}/ldebug.c"
  "${_LUA_SOURCE_DIR}/ldo.c"
  "${_LUA_SOURCE_DIR}/ldump.c"
  "${_LUA_SOURCE_DIR}/lfunc.c"
  "${_LUA_SOURCE_DIR}/lgc.c"
  "${_LUA_SOURCE_DIR}/llex.c"
  "${_LUA_SOURCE_DIR}/lmem.c"
  "${_LUA_SOURCE_DIR}/lobject.c"
  "${_LUA_SOURCE_DIR}/lopcodes.c"
  "${_LUA_SOURCE_DIR}/lparser.c"
  "${_LUA_SOURCE_DIR}/lstate.c"
  "${_LUA_SOURCE_DIR}/lstring.c"
  "${_LUA_SOURCE_DIR}/ltable.c"
  "${_LUA_SOURCE_DIR}/ltm.c"
  "${_LUA_SOURCE_DIR}/lundump.c"
  "${_LUA_SOURCE_DIR}/lvm.c"
  "${_LUA_SOURCE_DIR}/lzio.c"
  "${_LUA_SOURCE_DIR}/lauxlib.c"
  "${_LUA_SOURCE_DIR}/lbaselib.c"
  "${_LUA_SOURCE_DIR}/lcorolib.c"
  "${_LUA_SOURCE_DIR}/ldblib.c"
  "${_LUA_SOURCE_DIR}/liolib.c"
  "${_LUA_SOURCE_DIR}/lmathlib.c"
  "${_LUA_SOURCE_DIR}/loadlib.c"
  "${_LUA_SOURCE_DIR}/loslib.c"
  "${_LUA_SOURCE_DIR}/lstrlib.c"
  "${_LUA_SOURCE_DIR}/ltablib.c"
  "${_LUA_SOURCE_DIR}/lutf8lib.c"
  "${_LUA_SOURCE_DIR}/linit.c"
)

set(_LUA_COMPILER_SRC ${_LUA_SOURCE_DIR}/luac.c)
set(_LUA_INTERPRETER_SRC ${_LUA_SOURCE_DIR}/lua.c)

add_library(lua-obj OBJECT ${_LUA_LIB_SRC})

if(BUILD_SHARED_LIBS)
  set_target_properties(lua-obj PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()

if(LUA_COMPILE_AS_CPP)
  set_source_files_properties(${_LUA_LIB_SRC} ${_LUA_COMPILER_SRC} ${_LUA_INTERPRETER_SRC}
    PROPERTIES LANGUAGE CXX
  )
  set_target_properties(lua-obj PROPERTIES
    CXX_STANDARD 98
    CXX_EXTENSIONS OFF
  )
else()
  set_target_properties(lua-obj PROPERTIES
    C_STANDARD 99
    C_EXTENSIONS OFF
  )
endif()

if(WIN32)
  target_compile_definitions(lua-obj PUBLIC _CRT_SECURE_NO_WARNINGS)
endif()

if(UNIX)
  if(NOT ANDROID)
    target_compile_definitions(lua-obj PUBLIC LUA_USE_POSIX)
  endif()
  find_library(LIB_MATH NAMES m)
  if(LIB_MATH)
    target_link_libraries(lua-obj PUBLIC ${LIB_MATH})
  endif()
  mark_as_advanced(LIB_MATH)
  if(APPLE)
    target_compile_definitions(lua-obj PRIVATE LUA_USE_DLOPEN)
  else()
    find_library(LIB_DLOPEN NAMES dl)
    if(LIB_DLOPEN)
      target_compile_definitions(lua-obj PRIVATE LUA_USE_DLOPEN)
      target_link_libraries(lua-obj PRIVATE ${LIB_DLOPEN})
    endif()
    mark_as_advanced(LIB_DLOPEN)
  endif()
endif()

add_library(lua $<TARGET_OBJECTS:lua-obj>)
# for compatibility with the FindLua.cmake module
add_library(Lua::lua ALIAS lua)
target_link_libraries(lua PUBLIC $<BUILD_INTERFACE:lua-obj>)

if(LUA_COMPILE_AS_CPP)
  set_target_properties(lua PROPERTIES OUTPUT_NAME "lua-cxx")
endif()

if(BUILD_SHARED_LIBS AND WIN32)
  target_compile_definitions(lua PUBLIC LUA_BUILD_AS_DLL)
endif()

# test
if(LUA_PACKAGE_TEST)
  add_subdirectory(test_package)
endif()

# install
install(
  FILES
    ${_LUA_SOURCE_DIR}/lualib.h
    ${_LUA_SOURCE_DIR}/lua.h
    ${_LUA_SOURCE_DIR}/luaconf.h
    ${_LUA_SOURCE_DIR}/lauxlib.h
  DESTINATION include
  COMPONENT dev
)

# when compiling as c++, skip the `extern "C"` wrapper
if(NOT LUA_COMPILE_AS_CPP)
  install(
    FILES
      ${_LUA_SOURCE_DIR}/lua.hpp
      DESTINATION include
      COMPONENT dev
  )
endif()

include(GNUInstallDirs)
install(
  TARGETS lua
  EXPORT Lua-targets
  COMPONENT dev
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

include(CMakePackageConfigHelpers)
set(_LuaCMakePath "${CMAKE_CURRENT_BINARY_DIR}/cmake/${_Lua}")
set(_LuaVersionFile "${_LuaCMakePath}/${_Lua}-config-version.cmake")
write_basic_package_version_file(${_LuaVersionFile}
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

set(_LuaConfigFile "${_LuaCMakePath}/${_Lua}-config.cmake")
configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/${_Lua}-config.cmake.in
  ${_LuaConfigFile}
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${_Lua})

install(
  EXPORT 
    ${_Lua}-targets
  FILE 
    ${_Lua}-targets.cmake
  NAMESPACE ${_Lua}::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${_Lua}
)

install(
  FILES
    ${_LuaVersionFile}
    ${_LuaConfigFile}
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${_Lua}
)

if(NOT LUA_SKIP_TOOLS)
  # luac uses non-exported APIs
  add_executable(luac ${_LUA_COMPILER_SRC} $<TARGET_OBJECTS:lua-obj>)
  target_link_libraries(luac PRIVATE $<BUILD_INTERFACE:lua-obj>)
  # interpreter
  add_executable(luai ${_LUA_INTERPRETER_SRC})
  target_link_libraries(luai PRIVATE lua)
  set_target_properties(luai PROPERTIES OUTPUT_NAME lua PDB_NAME luai)
  if(UNIX)
    if(CMAKE_SYSTEM_NAME STREQUAL FreeBSD)
      set(_LIB_READLINE_NAME edit)
    else()
      set(_LIB_READLINE_NAME readline)
    endif()
    find_library(LIB_READLINE NAMES ${_LIB_READLINE_NAME})
    if(LIB_READLINE)
      target_compile_definitions(luai PRIVATE LUA_USE_READLINE)
      target_link_libraries(luai PRIVATE ${LIB_READLINE})
      if(_LIB_READLINE_NAME STREQUAL edit)
        TARGET_INCLUDE_DIRECTORIES (luai PUBLIC /usr/include/edit)
      endif()
    endif()
  endif()
  mark_as_advanced(LIB_READLINE)
  install(
    TARGETS
      luai
      luac 
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}/
    COMPONENT bin
  )
endif()
