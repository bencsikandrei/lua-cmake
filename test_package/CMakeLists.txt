project(TestInstalledPackage)

include(ExternalProject)

set(PROJECT "LuapackageTest")
message(STATUS "Installing: ${PROJECT}")
set(_THIS_GENERATED_FILES_LOCATION "${CMAKE_BINARY_DIR}/package_test/${PROJECT}-download")

if(NOT EXISTS "${_THIS_GENERATED_FILES_LOCATION}")
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/This.cmake.in 
    ${_THIS_GENERATED_FILES_LOCATION}/CMakeLists.txt @ONLY)
  execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
    WORKING_DIRECTORY "${_THIS_GENERATED_FILES_LOCATION}"
  )
  execute_process(COMMAND "${CMAKE_COMMAND}" --build .
    WORKING_DIRECTORY "${_THIS_GENERATED_FILES_LOCATION}"
  )
endif()

list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR}/package_test/${PROJECT}-install)

find_package(Lua REQUIRED)

add_executable(LuaPackageTest main.cpp)

target_link_libraries(LuaPackageTest PRIVATE ${LUA_LIBRARIES})
target_include_directories(LuaPackageTest PRIVATE ${LUA_INCLUDE_DIR})

message(STATUS "Test passed!")
