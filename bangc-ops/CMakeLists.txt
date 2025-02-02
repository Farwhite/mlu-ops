cmake_minimum_required(VERSION 3.3)

include_directories("${CMAKE_CURRENT_SOURCE_DIR}")
set(EXECUTABLE_OUTPUT_PATH "${CMAKE_BINARY_DIR}/test")
set(LIBRARY_OUTPUT_PATH "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -fPIC -std=c++11 -pthread -pipe")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${CMAKE_CXX_FLAGS} -O3")
set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} -Wl,--gc-sections -fPIC")

################################################################################
# Neuware Evironment and BANG Setup
################################################################################

# check `NEUWARE_HOME` env
message(${NEUWARE_HOME})
if(EXISTS ${NEUWARE_HOME})
  include_directories("${NEUWARE_HOME}/include")
  link_directories("${NEUWARE_HOME}/lib64")
  link_directories("${NEUWARE_HOME}/lib")
  set(NEUWARE_ROOT_DIR "${NEUWARE_HOME}")
else()
  message(FATAL_ERROR "NEUWARE directory cannot be found, refer README.md to prepare NEUWARE_HOME environment.")
endif()

# setup cmake search path
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH}
  "${CMAKE_SOURCE_DIR}/cmake"
  "${NEUWARE_HOME}/cmake"
  "${NEUWARE_HOME}/cmake/modules"
)

# include FindBANG.cmake and check cncc
find_package(BANG)
if(NOT BANG_FOUND)
  message(FATAL_ERROR "BANG cannot be found.")
elseif (NOT BANG_CNCC_EXECUTABLE)
  message(FATAL_ERROR "cncc not found, please ensure cncc is in your PATH env or set variable BANG_CNCC_EXECUTABLE from cmake. Otherwise you should check path used by find_program(BANG_CNCC_EXECUTABLE) in FindBANG.cmake")
endif()

# setup cncc flags
set(BANG_CNCC_FLAGS "${BANG_CNCC_FLAGS} -fPIC -Wall -Werror -std=c++11 -pthread")
set(BANG_CNCC_FLAGS "${BANG_CNCC_FLAGS} -O3")
set(BANG_CNCC_FLAGS "${BANG_CNCC_FLAGS}" "--bang-mlu-arch=mtp_220"
                                         "--bang-mlu-arch=mtp_270"
                                         "--bang-mlu-arch=mtp_290"
                                         "--bang-mlu-arch=mtp_372"
)
file(GLOB_RECURSE src_files ${src_files} "${CMAKE_CURRENT_SOURCE_DIR}/kernels/*.mlu" "${CMAKE_CURRENT_SOURCE_DIR}/core/*.cpp")
# set(src_files ${src_files} "${CMAKE_CURRENT_SOURCE_DIR}/test/main.cpp")
bang_add_library(mluops SHARED ${src_files})
target_link_libraries(mluops cnrt cndrv)

################################################################################
# Build MLUOP GTEST
################################################################################
option(MLU_OP_BUILD_GTEST "Build mlu-ops gtest" ON)
message("-- MLU_OP_BUILD_GTEST=${MLU_OP_BUILD_GTEST}")
if(${MLU_OP_BUILD_GTEST} MATCHES "ON")
  message("-- Build MLUOP Gtest")
  add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/test/mlu_op_gtest" "mlu_op_gtest")
endif()
