cmake_minimum_required (VERSION 2.8)


set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

project (Tutorial)



# setup CUDA
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/utils/cuda" )
find_package(CUDA)
message("-- CUDA version: ${CUDA_VERSION}")

set(
	CUDA_NVCC_FLAGS
	${CUDA_NVCC_FLAGS}; 
    -O3 
	-gencode arch=compute_53,code=sm_53
	-gencode arch=compute_62,code=sm_62
)

if(CUDA_VERSION_MAJOR GREATER 9)
	message("-- CUDA ${CUDA_VERSION_MAJOR} detected, enabling SM_72")

	set(
		CUDA_NVCC_FLAGS
		${CUDA_NVCC_FLAGS}; 
		-gencode arch=compute_72,code=sm_72
	)

	# OpenCV used for findHomography() and decomposeHomography()
	# OpenCV version >= 3.0.0 required for decomposeHomography()
	find_package(OpenCV 3.0.0 COMPONENTS core calib3d REQUIRED)
endif()

# setup project output paths
set(PROJECT_OUTPUT_DIR  ${PROJECT_BINARY_DIR}/${CMAKE_SYSTEM_PROCESSOR})
set(PROJECT_INCLUDE_DIR ${PROJECT_OUTPUT_DIR}/include)

file(MAKE_DIRECTORY ${PROJECT_INCLUDE_DIR})
file(MAKE_DIRECTORY ${PROJECT_OUTPUT_DIR}/bin)

message("-- system arch:  ${CMAKE_SYSTEM_PROCESSOR}")
message("-- output path:  ${PROJECT_OUTPUT_DIR}")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_OUTPUT_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_OUTPUT_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_OUTPUT_DIR}/lib)


# build C/C++ library
#include_directories(${PROJECT_INCLUDE_DIR} ${PROJECT_INCLUDE_DIR}/jetson-inference ${PROJECT_INCLUDE_DIR}/jetson-utils)
#include_directories(/usr/include/gstreamer-1.0 /usr/lib/aarch64-linux-gnu/gstreamer-1.0/include /usr/include/glib-2.0 /usr/include/libxml2 /usr/lib/aarch64-linux-gnu/glib-2.0/include/)

#file(GLOB inferenceSources c/*.cpp c/*.cu calibration/*.cpp plugins/*.cpp)
#file(GLOB inferenceIncludes c/*.h c/*.cuh calibration/*.h)

#cuda_add_library(jetson-inference SHARED ${inferenceSources})
#target_link_libraries(jetson-inference nvcaffe_parser nvinfer)		# gstreamer-0.10 gstbase-0.10 gstapp-0.10 


# transfer all headers to the include directory
#file(MAKE_DIRECTORY ${PROJECT_INCLUDE_DIR}/jetson-inference)

#foreach(include ${inferenceIncludes})
#	message("-- Copying ${include}")
#	configure_file(${include} ${PROJECT_INCLUDE_DIR}/jetson-inference COPYONLY)
#endforeach()


# create symbolic link for network data
#execute_process( COMMAND "${CMAKE_COMMAND}" "-E" "create_symlink" "${PROJECT_SOURCE_DIR}/data/networks" "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/networks" )
  
  
# copy image data
#file(GLOB imageData ${PROJECT_SOURCE_DIR}/data/images/*)

#foreach(image ${imageData})
#	message("-- Copying ${image}")
#	file(COPY ${image} DESTINATION ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
#	#configure_file(${include} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} COPYONLY)
#endforeach()


# build subdirectories
#add_subdirectory(docs)
#add_subdirectory(examples)
#add_subdirectory(tools)
#add_subdirectory(utils)
#add_subdirectory(python)



add_library(nvinfer SHARED IMPORTED) # or STATIC instead of SHARED
set_target_properties(nvinfer PROPERTIES
  IMPORTED_LOCATION /home/tondelli/TensorRT-5.1.5.0/lib/libnvinfer.so
  #INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_SOURCE_DIR}/include/libbar"
)




file(GLOB SOURCES
    /home/tondelli/TensorRT-5.1.5.0/include/*.h
    /home/tondelli/TensorRT-5.1.5.0/samples/common/*.h
    /home/tondelli/TensorRT-5.1.5.0/samples/common/*.cpp
)




# serve per includere le librerire dinamiche
link_directories(/home/tondelli/TensorRT-5.1.5.0/lib)


# cuda_add_executable serve se compilo progetti con cuda
cuda_add_executable(Tutorial sampleMNIST.cpp ${SOURCES})


# library

target_include_directories(Tutorial PRIVATE /home/tondelli/TensorRT-5.1.5.0/include)

target_include_directories(Tutorial PRIVATE /home/tondelli/TensorRT-5.1.5.0/samples/common)


target_include_directories(Tutorial PRIVATE ${CUDA_INCLUDE_DIRS})

# set linker options
target_link_libraries(Tutorial nvinfer nvinfer_plugin nvcaffe_parser)



if(CUDA_VERSION_MAJOR GREATER 9)
	target_link_libraries(Tutorial nvonnxparser opencv_core opencv_calib3d)
endif()










