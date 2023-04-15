# Copyright (c) Facebook, Inc. and its affiliates.
# All rights reserved.
#
# This source code is licensed under the BSD-style license found in the
# LICENSE file in the root directory of this source tree.

cmake_minimum_required(VERSION 3.23.1 FATAL_ERROR)

# Set the OpenMP flags directly
if (APPLE)
  set(OpenMP_CXX_FLAGS "-Xpreprocessor -fopenmp -I/opt/homebrew/opt/libomp/include")
  set(OpenMP_omp_LIBRARY "/opt/homebrew/opt/libomp/lib/libomp.dylib")
endif()

if(UNIX AND NOT APPLE)
  execute_process(COMMAND find /usr -iname "omp.h" -exec dirname {} \; 2>/dev/null
                  COMMAND head -n 1
                  OUTPUT_VARIABLE OPENMP_INCLUDE_PATH
                  OUTPUT_STRIP_TRAILING_WHITESPACE)
  execute_process(COMMAND find /usr -iname "libomp.so*" -exec dirname {} \; 2>/dev/null
                  COMMAND head -n 1
                  OUTPUT_VARIABLE OPENMP_LIB_PATH
                  OUTPUT_STRIP_TRAILING_WHITESPACE)

  set(OpenMP_C_FLAGS "-Xpreprocessor -fopenmp -I${OPENMP_INCLUDE_PATH}")
  set(OpenMP_CXX_FLAGS "-Xpreprocessor -fopenmp -I${OPENMP_INCLUDE_PATH}")
  set(OpenMP_CXX_LIB_NAMES "omp")
  set(OpenMP_omp_LIBRARY "${OPENMP_LIB_PATH}/libomp.so")
endif()

message(STATUS "OpenMP_CXX_FLAGS: ${OpenMP_CXX_FLAGS}")
message(STATUS "OpenMP_omp_LIBRARY: ${OpenMP_omp_LIBRARY}")

set(OpenMP_CXX_LIB_NAMES "omp")

project(faiss
  VERSION 1.7.3
  DESCRIPTION "A library for efficient similarity search and clustering of dense vectors."
  HOMEPAGE_URL "https://github.com/facebookresearch/faiss"
  LANGUAGES CXX)
include(GNUInstallDirs)

set(CMAKE_CXX_STANDARD 11)

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

# Valid values are "generic", "avx2".
option(FAISS_OPT_LEVEL "" "generic")
option(FAISS_ENABLE_GPU "Enable support for GPU indexes." OFF)
option(FAISS_ENABLE_PYTHON "Build Python extension." ON)
option(FAISS_ENABLE_C_API "Build C API." OFF)

if(FAISS_ENABLE_GPU)
  set(CMAKE_CUDA_HOST_COMPILER ${CMAKE_CXX_COMPILER})
  enable_language(CUDA)
endif()

add_subdirectory(faiss)

if(FAISS_ENABLE_GPU)
  add_subdirectory(faiss/gpu)
endif()

if(FAISS_ENABLE_PYTHON)
  add_subdirectory(faiss/python)
endif()

if(FAISS_ENABLE_C_API)
  add_subdirectory(c_api)
endif()

add_subdirectory(demos)
add_subdirectory(benchs)
add_subdirectory(tutorial/cpp)

# CTest must be included in the top level to enable `make test` target.
include(CTest)
if(BUILD_TESTING)
  add_subdirectory(tests)

  if(FAISS_ENABLE_GPU)
    add_subdirectory(faiss/gpu/test)
  endif()
endif()
