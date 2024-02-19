#cmake version
cmake_minimum_required(VERSION 3.20.2)

# description of project
project(FastEddy
    VERSION 1
    DESCRIPTION "FastEddy® (FE) is a large-eddy simulation (LES) model developed by the Research Applications Laboratory (RAL) at the National Center for Atmospheric Research (NCAR) in Boulder, Colorado, USA"
    LANGUAGES C CXX CUDA)

find_package(CUDA REQUIRED)
find_package(CUDAToolkit REQUIRED)
find_package(MPI REQUIRED)
# find_package(NetCDF REQUIRED)

set( CMAKE_C_COMPILER /usr/bin/mpicc )

add_definitions(
    -DCUB_IGNORE_DEPRECATED_CPP_DIALECT 
    -DTHRUST_IGNORE_DEPRECATED_CPP_DIALECT
    -DOMPI_SKIP_MPICXX
)

add_compile_options(
  -Wall 
  -m64
#   -arch=sm_70
  -std=c++11
)
include_directories(SYSTEM ${MPI_INCLUDE_PATH})
include_directories(SYSTEM ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES})

include_directories(
    SRC/FEMAIN
    SRC/TIME_INTEGRATION
    SRC/TIME_INTEGRATION/CUDA
    SRC/HYDRO_CORE
    SRC/HYDRO_CORE/CUDA
    SRC/GRID
    SRC/GRID/CUDA
    SRC/FECUDA
    SRC/IO
    SRC/MEM_UTILS
    SRC/FEMPI
    SRC/PARAMETERS
)

add_library(
    io
    OBJECT
    SRC/IO/io.c
    SRC/IO/io_netcdf.c
    SRC/IO/io_binary.c
)
target_compile_features( io PUBLIC cxx_std_11 )
set_target_properties( io PROPERTIES CUDA_SEPARABLE_COMPILATION ON )

add_library(
    fecuda_Device
    OBJECT
    SRC/FECUDA/fecuda_Device.cu
	SRC/FECUDA/fecuda_Utils.cu
	SRC/FECUDA/fecuda_PlugIns.cu
)
target_compile_features( fecuda_Device PUBLIC cxx_std_11 )
set_target_properties( fecuda_Device PROPERTIES CUDA_SEPARABLE_COMPILATION ON )

add_library(
    cuda_hydroCoreDevice
    OBJECT
    SRC/HYDRO_CORE/CUDA/cuda_hydroCoreDevice.cu
	SRC/HYDRO_CORE/CUDA/cuda_BaseStateDevice.cu
	SRC/HYDRO_CORE/CUDA/cuda_buoyancyDevice.cu
	SRC/HYDRO_CORE/CUDA/cuda_coriolisDevice.cu
	SRC/HYDRO_CORE/CUDA/cuda_pressureDevice.cu
	SRC/HYDRO_CORE/CUDA/cuda_BCsDevice.cu
	SRC/HYDRO_CORE/CUDA/cuda_rayleighDampingDevice.cu
	SRC/HYDRO_CORE/CUDA/cuda_advectionDevice.cu
	SRC/HYDRO_CORE/CUDA/cuda_molecularDiffDevice.cu
	SRC/HYDRO_CORE/CUDA/cuda_surfaceLayerDevice.cu
	SRC/HYDRO_CORE/CUDA/cuda_sgsTurbDevice.cu
    SRC/HYDRO_CORE/CUDA/cuda_sgstkeDevice.cu
    SRC/HYDRO_CORE/CUDA/cuda_largeScaleForcingsDevice.cu
    SRC/HYDRO_CORE/CUDA/cuda_moistureDevice.cu
    SRC/HYDRO_CORE/CUDA/cuda_filtersDevice.cu
    # ../TIME_INTEGRATION/time_integration.h \
    # ../TIME_INTEGRATION/CUDA/cuda_timeInt.h \
    # ../TIME_INTEGRATION/CUDA/cuda_timeIntDevice_cu.h \
	SRC/HYDRO_CORE/hydro_core.h
	SRC/HYDRO_CORE/CUDA/cuda_hydroCore.h
	SRC/HYDRO_CORE/CUDA/cuda_hydroCoreDevice_cu.h
	# ../GRID/grid.h \
	# ../GRID/CUDA/cuda_grid.h \
	# ../GRID/CUDA/cuda_gridDevice_cu.h \
    # ../FECUDA/fecuda.h \
    # ../FECUDA/fecuda_Device_cu.h \
	# ../IO/io.h \
	# ../MEM_UTILS/mem_utils.h \
	# ../FEMPI/fempi.h \
	# ../PARAMETERS/parameters.h \
	# ../PARAMETERS/hashTable.h
)
target_compile_features( cuda_hydroCoreDevice PUBLIC cxx_std_11 )
set_target_properties( cuda_hydroCoreDevice PROPERTIES CUDA_SEPARABLE_COMPILATION ON )

add_library(
    cuda_timeIntDevice
    OBJECT
    SRC/TIME_INTEGRATION/CUDA/cuda_timeIntDevice.cu
    SRC//TIME_INTEGRATION/CUDA/cuda_RKschemes.cu
)
target_compile_features( cuda_timeIntDevice PUBLIC cxx_std_11 )
set_target_properties( cuda_timeIntDevice PROPERTIES CUDA_SEPARABLE_COMPILATION ON )

add_executable(
    FastEddy
    SRC/FEMAIN/FastEddy.c
    SRC/TIME_INTEGRATION/time_integration.c
    SRC/TIME_INTEGRATION/CUDA/cuda_timeInt.c
	SRC/HYDRO_CORE/hydro_core.c
	SRC/HYDRO_CORE/CUDA/cuda_hydroCore.c
	SRC/GRID/grid.c
	SRC/GRID/CUDA/cuda_grid.c
	SRC/GRID/CUDA/cuda_gridDevice.cu
    SRC/FECUDA/fecuda.c
	SRC/IO/ioVarsList.c
	SRC/MEM_UTILS/mem_utils.c
	SRC/FEMPI/fempi.c
	SRC/PARAMETERS/parameters.c
	SRC/PARAMETERS/hashTable.c
)

target_link_libraries( FastEddy io )
target_link_libraries( FastEddy fecuda_Device )
target_link_libraries( FastEddy cuda_hydroCoreDevice )
target_link_libraries( FastEddy ${MPI_C_LIBRARIES} )