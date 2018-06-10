cmake_minimum_required(VERSION 3.4)

project(Petaca Fortran C)

set(CMAKE_MODULE_PATH "${Petaca_SOURCE_DIR}/cmake")

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# The parameter list modules require the yajl library. If necessary, set
# YAJL_ROOT or the YAJL_INCLUDE_DIR and YAJL_LIBRARY CMake variables.
find_package(YAJL "2.0.1" REQUIRED)

#set(Fortran_COMPILE_DEFINITIONS)  # empty to start

# NAG Fortran compiler. Version 6.0 and later.
if(CMAKE_Fortran_COMPILER_ID MATCHES "NAG")
  include(NAGFortranCompilerVersion)  # for older NAG versions
  set(CMAKE_Fortran_FLAGS_RELEASE "-O3 -w=uda -DNDEBUG")
  set(CMAKE_Fortran_FLAGS_DEBUG   "-g -gline -C")
endif()

# Intel Fortran compiler. Version 16 and later.
if (CMAKE_Fortran_COMPILER_ID MATCHES "Intel")
  set(CMAKE_Fortran_FLAGS_RELEASE "-O3 -DNDEBUG")
  set(CMAKE_Fortran_FLAGS_DEBUG   "-g -C")
  if(CMAKE_Fortran_COMPILER_VERSION VERSION_LESS "17.0.0")
    string(APPEND CMAKE_Fortran_FLAGS " -assume realloc_lhs")
  endif()
  list(APPEND Fortran_COMPILE_DEFINITIONS NO_2008_GENERIC_RESOLUTION)
  list(APPEND Fortran_COMPILE_DEFINITIONS NO_2008_LHS_POLY_REALLOC)
  list(APPEND Fortran_COMPILE_DEFINITIONS INTEL_BUG20180115)
  list(APPEND Fortran_COMPILE_DEFINITIONS INTEL_BUG20140921)
  if(CMAKE_Fortran_COMPILER_VERSION VERSION_GREATER "18.0.1")
    list(APPEND Fortran_COMPILE_DEFINITIONS INTEL_BUG20171115)
  endif()
endif()

# GCC Fortran compiler. Versions 7.3.1 and later, 8.1.1 and later
if(CMAKE_Fortran_COMPILER_ID MATCHES "GNU")
  set(CMAKE_Fortran_FLAGS_RELEASE "-O3 -DNDEBUG")
  set(CMAKE_Fortran_FLAGS_DEBUG "-g -fbacktrace -fcheck=all")
  string(APPEND CMAKE_Fortran_FLAGS " -ffree-line-length-none")
  list(APPEND Fortran_COMPILE_DEFINITIONS NO_2008_GENERIC_RESOLUTION)
endif()

# Add the Fortran preprocessor definitions to the compiler options.
if(Fortran_COMPILE_DEFINITIONS)
  foreach(def ${Fortran_COMPILE_DEFINITIONS})
    add_compile_options("$<$<COMPILE_LANGUAGE:Fortran>:-D${def}>")
  endforeach()
endif()

add_subdirectory(src)

option(BUILD_TESTS "Build unit tests" ON)
if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(test)
endif()

option(BUILD_EXAMPLES "Build example programs" OFF)
if(BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()

option(BUILD_HTML "Build HTML documentation (requires Sphinx)" OFF)
if(BUILD_HTML)
  find_package(Sphinx REQUIRED)
  add_subdirectory(doc/sphinx)
endif()
