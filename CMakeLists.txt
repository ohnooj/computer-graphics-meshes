cmake_minimum_required(VERSION 3.20)
project(meshes)

# C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

### libIGL options: choose between header only and compiled static library
option(LIBIGL_USE_STATIC_LIBRARY    "Use libigl as static library" OFF)
option(LIBIGL_OPENGL                "Use OpenGL"         ON)
option(LIBIGL_STB                   "Use STB"            ON)
option(LIBIGL_GLFW                  "Use GLFW"           ON)

include(FetchContent)
FetchContent_Declare(
  libigl
  GIT_REPOSITORY https://github.com/libigl/libigl.git
  GIT_TAG        v2.6.0
)
FetchContent_MakeAvailable(libigl)

# Add your project files
option(USE_SOLUTION "Use solution files or src files" OFF)
set(SRCFILES "${CMAKE_CURRENT_SOURCE_DIR}/src")
if(USE_SOLUTION)
  set(SRCFILES "${CMAKE_CURRENT_SOURCE_DIR}/solution")
endif()
file(GLOB SRCFILES CONFIGURE_DEPENDS "${SRCFILES}/*.cpp")

# core library
add_library(core STATIC ${SRCFILES})
target_include_directories(core
  PUBLIC
  "${CMAKE_CURRENT_SOURCE_DIR}/include"
)

# link to core library
target_link_libraries(core PUBLIC igl::core)
if(LIBIGL_OPENGL)
  target_link_libraries(core PUBLIC igl::opengl)
endif()

if(LIBIGL_STB)
  target_link_libraries(core PUBLIC igl::stb)
endif()

if(LIBIGL_GLFW)
  target_link_libraries(core PUBLIC igl::glfw)
endif()

# Linke core to executables
add_executable(normals "normals.cpp")
target_link_libraries(normals PRIVATE core)

add_executable(quad_subdivision "quad_subdivision.cpp")
target_link_libraries(quad_subdivision PRIVATE core)

add_executable(obj "obj.cpp")
target_link_libraries(obj PRIVATE core)

# Warnings
if(MSVC)
  target_compile_options(core PRIVATE /W4 /permissive-)
else()
  target_compile_options(core PRIVATE -Wall -Wextra -Wpedantic)
endif()
