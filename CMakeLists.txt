cmake_minimum_required(VERSION 3.0)

# ---- project ----

project("cmake-glad-generator" VERSION "1.1.0" LANGUAGES "C"
  HOMEPAGE_URL "https://github.com/robertefry/cmake-glad-generator"
  DESCRIPTION "Generate GLAD OpenGL loaders on demand")

# ---- add dependencies via CPM ----
# see https://github.com/cpm-cmake/CPM.cmake for more info
include(.cmake/get_cpm.cmake)

set(GLAD_GIT_TAG "v2.0.2" CACHE STRING "" FORCE)

find_package(PythonInterp 3.0 REQUIRED)
find_package(PythonLibs 3.0 REQUIRED)

CPMAddPackage(NAME "glad"
  GITHUB_REPOSITORY "Dav1dde/glad" GIT_TAG ${GLAD_GIT_TAG}
)

# ---- generate glad sources ----

set(GEN_GLAD_API "core=1.0" CACHE STRING "The version of OpenGL to generate a loader for.")
set(GEN_GLAD_OPTS "" CACHE STRING "The generator options.")

message(STATUS "Building the GLAD generator")
execute_process(
  COMMAND ${PYTHON_EXECUTABLE} setup.py build
  WORKING_DIRECTORY ${glad_SOURCE_DIR})

message(STATUS "Running the GLAD generator")
execute_process(
  COMMAND ${PYTHON_EXECUTABLE} -m glad
    --out-path ${CMAKE_CURRENT_SOURCE_DIR}
    --api gl:${GEN_GLAD_API} c ${GEN_GLAD_OPTS}
  WORKING_DIRECTORY ${glad_SOURCE_DIR})

# ---- project target ----

set(BINARY_NAME "glad2")
add_library(${BINARY_NAME} STATIC "")

target_include_directories(${BINARY_NAME}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
  PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
)

file(GLOB_RECURSE GLOB_SOURCES CONFIGURE_DEPENDS
  "src/*.c" "src/*.cc" "src/*.cpp" "src/*.cxx")
target_sources(${BINARY_NAME} PRIVATE ${GLOB_SOURCES})

# ---- glad generated library installable ----
# this allows users to install and find the library via `find_package`.
CPMAddPackage("gh:TheLartians/PackageProject.cmake@1.10.0")

packageProject(
  NAME "glad2"
  VERSION ${GLAD_GIT_TAG}
  BINARY_DIR ${PROJECT_BINARY_DIR}
)
