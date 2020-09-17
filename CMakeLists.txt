########################################################################
# Note: CMake support is community-based. The maintainers do not use CMake
# internally.
#
# CMake build script for Google Test.
#
# To run the tests for Google Test itself on Linux, use 'make test' or
# ctest.  You can select which tests to run using 'ctest -R regex'.
# For more options, run 'ctest --help'.

# When other libraries are using a shared version of runtime libraries,
# Google Test also has to use one.

set(gtest_force_shared_crt OFF)
set(gtest_build_samples ON)
option(gtest_force_shared_crt "Use shared (DLL) run-time lib even when Google Test is built as static lib."  OFF)
option(gtest_disable_pthreads "Disable uses of pthreads in gtest." OFF)

########################################################################
#
# Project-wide settings

# Name of the project.
#
# CMake files in this project can refer to the root source directory
# as ${gtest_SOURCE_DIR} and to the root binary directory as
# ${gtest_BINARY_DIR}.
# Language "C" is required for find_package(Threads).

# Project version:

if (CMAKE_VERSION VERSION_LESS 3.0)
  project(gtest CXX C)
  set(PROJECT_VERSION ${GOOGLETEST_VERSION})
else()
  cmake_policy(SET CMP0048 NEW)
  project(gtest VERSION ${GOOGLETEST_VERSION} LANGUAGES CXX C)
endif()
cmake_minimum_required(VERSION 2.6.4)

if (POLICY CMP0063) # Visibility
  cmake_policy(SET CMP0063 NEW)
endif (POLICY CMP0063)


set(GOOGLETEST_VERSION 1.10.0)

if (CMAKE_VERSION VERSION_GREATER "3.0.2")
  if(NOT CYGWIN AND NOT MSYS AND NOT ${CMAKE_SYSTEM_NAME} STREQUAL QNX)
    set(CMAKE_CXX_EXTENSIONS OFF)
  endif()
endif()

enable_testing()

#include(CMakeDependentOption)
#include(GNUInstallDirs)


mark_as_advanced(
  gtest_build_samples
  gtest_disable_pthreads)

# Define helper functions and macros used by Google Test.

include(internal_utils.cmake)

config_compiler_and_linker()  # Defined in internal_utils.cmake.

# Where Google Test's .h files can be found.
set(gtest_build_include_dirs
  "${gtest_SOURCE_DIR}/include"
  "${gtest_SOURCE_DIR}")
  
include_directories(${gtest_build_include_dirs})

########################################################################
#
# Defines the gtest & gtest_main libraries.  User tests should link
# with one of them.

# Google Test libraries.  We build them using more strict warnings than what
# are used for other targets, to ensure that gtest can be compiled by a user
# aggressive about warnings.
cxx_library(gtest "${cxx_strict}" src/gtest-all.cc)
set_target_properties(gtest PROPERTIES VERSION ${GOOGLETEST_VERSION})
cxx_library(gtest_main "${cxx_strict}" src/gtest_main.cc)
set_target_properties(gtest_main PROPERTIES VERSION ${GOOGLETEST_VERSION})
# If the CMake version supports it, attach header directory information
# to the targets for when we are part of a parent build (ie being pulled
# in via add_subdirectory() rather than being a standalone build).
if (DEFINED CMAKE_VERSION AND NOT "${CMAKE_VERSION}" VERSION_LESS "2.8.11")
  target_include_directories(gtest SYSTEM INTERFACE
    "$<BUILD_INTERFACE:${gtest_build_include_dirs}>"
    "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>")
  target_include_directories(gtest_main SYSTEM INTERFACE
    "$<BUILD_INTERFACE:${gtest_build_include_dirs}>"
    "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>")
endif()

target_link_libraries(gtest_main gtest)

########################################################################
#
# Install rules
#install_project(gtest gtest_main)

########################################################################
#
# Samples on how to link user tests with gtest or gtest_main.
#
# They are not built by default.  To build them, set the
# gtest_build_samples option to ON.  You can do it by running ccmake
# or specifying the -Dgtest_build_samples=ON flag when running cmake.

if (gtest_build_samples)
  cxx_executable(sample_unittest samples gtest_main samples/sample.c)
endif()
