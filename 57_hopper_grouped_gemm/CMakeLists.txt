cmake_minimum_required(VERSION 3.25.0)
project(hopper_grouped_gemm LANGUAGES CXX CUDA)

set(TARGET "hopper_grouped_gemm")

set(CMAKE_BUILD_TYPE Release)

# Prohibit in-source builds
if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
  message(FATAL_ERROR "In-source build are not supported")
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/../cmake")

set(CMAKE_CXX_STANDARD
    17
    CACHE STRING "The C++ standard whose features are requested." FORCE)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_CUDA_STANDARD
    17
    CACHE STRING "The CUDA standard whose features are requested." FORCE)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)

# Set host compiler flags. Enable all warnings and treat them as errors
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -Wall")

find_package(CUDAToolkit QUIET REQUIRED)
enable_language(CUDA)
set(CMAKE_CUDA on)

set(CUDA_PROPAGATE_HOST_FLAGS OFF)

set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} --expt-relaxed-constexpr")

include(select_compute_arch)
cuda_select_nvcc_arch_flags(ARCH_LIST Auto)
list(APPEND CUDA_NVCC_FLAGS ${ARCH_LIST})
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-gencode arch=compute_90a,code=sm_90a")

include_directories("${PROJECT_SOURCE_DIR}/../cutlass/include")
include_directories("${PROJECT_SOURCE_DIR}/../cutlass/tools/util/include")
include_directories("${PROJECT_SOURCE_DIR}/../cutlass/examples/common")

# ===============================================================

set(TEST_RANDOM --iterations=0) # Random problem sizes
set(TEST_RANDOM_LARGE_GROUP --groups=500 --iterations=0) # Random problem sizes

set(TEST_EPILOGUE --alpha=0.5 --beta=0.5 --iterations=0) # Random problem sizes
set(TEST_EPILOGUE_LARGE_GROUP --alpha=1.5 --beta=2.0 --groups=500
                              --iterations=0) # Random problem sizes

set(TEST_EPILOGUE_OP --beta=0.5 --iterations=1) # Random problem sizes
set(TEST_EPILOGUE_OP_LARGE_GROUP --alpha=1.5 --iterations=1) # Random problem
                                                             # sizes

set(TEST_FIXED --m=2048 --n=5120 --k=8192 --groups=50 --iterations=0) # Fixed
                                                                      # problem
                                                                      # sizes
set(TEST_FIXED_LARGE_GROUP --m=2048 --n=512 --k=512 --groups=512 --iterations=0
)# Fixed problem sizes

set(TEST_SMALL --m=256 --n=128 --iterations=0) # Small problem sizes
set(TEST_SMALL_LARGE_GROUP --m=128 --n=128 --groups=500 --iterations=0
)# Small problem sizes

set(TEST_RANDOM_PERF --iterations=10) # Random problem sizes
set(TEST_RANDOM_PERF_LARGE_GROUP --groups=500 --iterations=10) # Random problem
                                                               # sizes

# ===============================================================

add_executable(${TARGET} 57_hopper_grouped_gemm.cu)
set_target_properties(${TARGET} PROPERTIES CUDA_ARCHITECTURES 90a)
