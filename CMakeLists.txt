project(PolyJIT)
cmake_minimum_required(VERSION 3.2)

include(ExternalProject)

set(ISL_GIT "https://github.com/PolyJIT/isl.git")
set(ISL_CPP_GIT "https://github.com/PolyJIT/isl-cpp.git")
set(ISL_INSTALL_ROOT "" CACHE PATH "Root of ISL install.")
if( NOT EXISTS ${ISL_INSTALL_ROOT}/include/isl )
  message("ISL_INSTALL_ROOT (${ISL_INSTALL_ROOT}) is not a valid ISL installation.")
  message("Get isl @ ${ISL_GIT}")
  message("Get isl-cpp @ ${ISL_CPP_GIT}")
  message(FATAL_ERROR  "Please provide isl & isl-cpp via ISL_INSTALL_ROOT")
endif(NOT EXISTS ${ISL_INSTALL_ROOT}/include/isl)

ExternalProject_Add(llvm
  GIT_REPOSITORY git@github.com:PolyJIT/llvm.git
  GIT_TAG master
  STEP_TARGETS build install
  INDEPENDENT_STEP_TARGETS download
  CMAKE_ARGS
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    -DLLVM_TARGETS_TO_BUILD=X86
    -DBUILD_SHARED_LIBS=On
  )

ExternalProject_Get_Property(llvm BINARY_DIR)
set(LLVM_INSTALL_ROOT ${BINARY_DIR})

ExternalProject_Add(clang
  DEPENDS llvm
  GIT_REPOSITORY git@github.com:PolyJIT/clang.git
  GIT_TAG master
  EXCLUDE_FROM_ALL 1
  STEP_TARGETS build install
  INDEPENDENT_STEP_TARGETS download
  CMAKE_ARGS
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    -DBUILD_SHARED_LIBS=On
    -DLLVM_CONFIG=${LLVM_INSTALL_ROOT}/bin/llvm-config
  )

ExternalProject_Add(polly
  DEPENDS llvm
  GIT_REPOSITORY git@github.com:PolyJIT/polly.git
  GIT_TAG devel
  STEP_TARGETS build install
  INDEPENDENT_STEP_TARGETS download
  CMAKE_ARGS
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    -DBUILD_SHARED_LIBS=On
    -DLLVM_INSTALL_ROOT=${LLVM_INSTALL_ROOT}
  )

ExternalProject_Get_Property(polly BINARY_DIR)
ExternalProject_Get_Property(polly SOURCE_DIR)
set(POLLY_INSTALL_ROOT ${BINARY_DIR})
set(POLLY_SOURCE_ROOT ${SOURCE_DIR})

ExternalProject_Add(polli
  DEPENDS llvm polly clang 
  GIT_REPOSITORY git@github.com:PolyJIT/polli.git
  GIT_TAG next
  STEP_TARGETS build install
  INDEPENDENT_STEP_TARGETS download
  CMAKE_ARGS
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    -DBUILD_SHARED_LIBS=On
    -DLLVM_INSTALL_ROOT=${LLVM_INSTALL_ROOT}
    -DPOLLY_INSTALL_ROOT=${POLLY_INSTALL_ROOT}
    -DISL_INSTALL_ROOT=${ISL_INSTALL_ROOT}
  )
