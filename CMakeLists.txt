cmake_minimum_required(VERSION 3.10)

project(build_libtorch)

set(PYTORCH_SRC_DIR ../pytorch)

include(CMakeDependentOption)
option(RESET "Restore and modify PyTorch source tree")
CMAKE_DEPENDENT_OPTION(NO_BUILD_SHARED_LIBS "Do not build shared libraries" OFF RESET ON)
set(ENV{DEBUG} 1) #will enable debug builds (-g -O0)
set(ENV{BUILD_TEST} 0) # will disable building C++ test binaries
set(ENV{USE_CUDA} 0) #disable compiling CUDA, to save compile time
set(ENV{USE_DISTRIBUTED} 0) #disable distributed (c10d, gloo, mpi, etc.) build
set(ENV{USE_MKLDNN} 0) #disable using MKL-DNN
set(ENV{USE_FBGEMM} 0) #disable using FBGEMM (quantized 8-bit server operators)
set(ENV{USE_NNPACK} 0) #disable compiling with NNPACK
set(ENV{USE_QNNPACK} 0) #disable QNNPACK build (quantized 8-bit operators)
set(ENV{USE_XNNPACK} 0) #disable compiling with XNNPACK
set(ENV{BUILD_PYTHON} 0) #Build Python binaries
set(ENV{BUILD_CAFFE2_OPS} 0) #Build Caffe2 operators

if(RESET)
  message("xxx RESET")
  find_package(Git)
  if(Git_FOUND)
    execute_process(COMMAND ${GIT_EXECUTABLE} restore :/ WORKING_DIRECTORY ${PYTORCH_SRC_DIR} RESULT_VARIABLE RESULTV OUTPUT_VARIABLE OUTPUTV ERROR_VARIABLE ERRORV)
    message("xxx restore RESULTV ${RESULTV}")
    message("xxx restore OUTPUTV ${OUTPUTV}")
    message("xxx restore ERRORV ${ERRORV}")
    execute_process(COMMAND ${GIT_EXECUTABLE} clean -df WORKING_DIRECTORY ${PYTORCH_SRC_DIR} RESULT_VARIABLE RESULTV OUTPUT_VARIABLE OUTPUTV ERROR_VARIABLE ERRORV)
    message("xxx clean RESULTV ${RESULTV}")
    message("xxx clean OUTPUTV ${OUTPUTV}")
    message("xxx clean ERRORV ${ERRORV}")
  endif()

  if(NO_BUILD_SHARED_LIBS)
    file(READ ${PYTORCH_SRC_DIR}/CMakeLists.txt FILE_CONTENTS)
    string(REPLACE "option(BUILD_SHARED_LIBS \"Build libcaffe2.so\" ON)" "option(BUILD_SHARED_LIBS \"Build libcaffe2.so\")" FILE_CONTENTS "${FILE_CONTENTS}")
    file(WRITE ${PYTORCH_SRC_DIR}/CMakeLists.txt "${FILE_CONTENTS}")
  endif()
endif()

find_package(Python)
if(Python_Interpreter_FOUND)
  execute_process(COMMAND ${Python_EXECUTABLE} tools/build_libtorch.py WORKING_DIRECTORY ${PYTORCH_SRC_DIR})
endif()