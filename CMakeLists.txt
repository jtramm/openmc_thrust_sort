cmake_minimum_required(VERSION 3.8.2)

# If you want to properly search for Thrust, you'll need a FindThrust.cmake 
# script, which constitutes a "CMake module". You place it under cmake/Modules
# in your source directory and make it available by uncommenting the following
# line:
#list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules")

project(openmc_thrust_sort
  DESCRIPTION "Sort EventQueueItem arrays in OpenMC on NVIDIA devices with Thrust"
    LANGUAGES CXX CUDA)

# Don't do this. Set your build type explicitly, once; and then it's
# cached and you don't have to worry about it when you run make.
#
#if (NOT CMAKE_BUILD_TYPE)
#    set(CMAKE_BUILD_TYPE Release)
#endif ()

# In the future, this should not be necessary, but we need it for 
# cuda_select_nvcc_arch_flags
include(FindCUDA) 

# This will set the appropriate gencode parameters for the hardware
# on your system (although you could always force it manually)
#cuda_select_nvcc_arch_flags(CUDA_ARCH_FLAGS_TMP Auto)
#set(CUDA_ARCH_FLAGS ${CUDA_ARCH_FLAGS_TMP} CACHE STRING "CUDA -gencode parameters")
#string(REPLACE ";" " " CUDA_ARCH_FLAGS_STR "${CUDA_ARCH_FLAGS}")
#set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} ${CUDA_ARCH_FLAGS_STR}")

#cuda_select_nvcc_arch_flags(ARCH_FLAGS 8.0)
#list(APPEND CUDA_NVCC_FLAGS ${ARCH_FLAGS})

# The above may produce something like:
#
#  -gencode;arch=compute_60,code=sm_60;-gencode;arch=compute_61,code=sm_61;-gencode;arch=compute_70,code=sm_70;-gencode;arch=compute_75,code=sm_75;-gencode;arch=compute_70,code=compute_70;-gencode;arch=compute_75,code=compute_75
#
# But it may include older micro-architectures which have been 
# deprecated/removed, in which case you'll need to edit that 
# with ccmake and only keep what you need.


list(APPEND openmc_thrust_sort_SOURCES
  src/openmc_thrust_sort.cu)

add_library(openmc_thrust_sort SHARED ${openmc_thrust_sort_SOURCES})

#add_executable(openmc_thrust_sort src/openmc_thrust_sort.cu)

set_target_properties(
    openmc_thrust_sort
    PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED YES
    CXX_EXTENSIONS NO
    CUDA_ARCHITECTURES "80"
)

# Note: I haven't added flags for compiling with warnings

# Thrust is very finickey: It provies a configuration script, but
# only for CMake >= 3.15 . And - it doesn't provide a FindThrust.cmake
# script itself with targets appropriate for CMake >= 3.
#
# See https://github.com/NVIDIA/thrust/blob/main/thrust/cmake/README.md
#
# With CMake 3.15 or later you can enable the following two lines:
#
#find_package(Thrust REQUIRED CONFIG)
#thrust_create_target(Thrust)
#target_link_libraries(openmc_thrust_sort Thrust)
#
# With earlier CMake versions, get yourself a proper FindThrust.cmake
# script (which creates a Thrust::Thrust target I suppose) and
# then uncomment the following two lines:
#
#find_package(Thrust REQUIRED)
#target_link_libraries(openmc_thrust_sort Thrust::Thrust)


# The following sets -rdc=true , but you don't actually need that for your example
set_target_properties(openmc_thrust_sort PROPERTIES CUDA_SEPARABLE_COMPILATION ON)
