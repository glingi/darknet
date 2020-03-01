cmake_minimum_required(VERSION 3.0.0)

# set the project name
project(YoloV3)

option(WITH_CUDA "Include NVidia Cuda Runtime support" OFF)
option(WITH_CUDNN "Include NVIDIA CUDA Deep Neural Network (cuDNN) library support" OFF)
option(WITH_OPENCV "Include OpenCV Runtime support" OFF)
option(WITH_OPENMP "Include OpenMP support" OFF)

# specify the C++14 standard and disable non-standard extensions
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14 -Wall -Ofast -Wfatal-errors -fPIC")

# set output binary name
set(bin_name Yolov3)

set(HEADER_FILES 
	src/activation_layer.h
	src/activations.h
	src/avgpool_layer.h
	src/batchnorm_layer.h
	src/blas.h
	src/box.h
	src/classifier.h
	src/col2im.h
	src/connected_layer.h
	src/convolutional_layer.h
	src/cost_layer.h
	src/crnn_layer.h
	src/crop_layer.h
	src/cuda.h
	src/data.h
	src/deconvolutional_layer.h
	src/demo.h
	src/detection_layer.h
	src/dropout_layer.h
	src/gemm.h
	src/gru_layer.h
	src/im2col.h
	src/image.h
	src/iseg_layer.h
	src/l2norm_layer.h
	src/layer.h
	src/list.h
	src/local_layer.h
	src/logistic_layer.h
	src/lstm_layer.h
	src/matrix.h
	src/maxpool_layer.h
	src/network.h
	src/normalization_layer.h
	src/option_list.h
	src/parser.h
	src/region_layer.h
	src/reorg_layer.h
	src/rnn_layer.h
	src/route_layer.h
	src/shortcut_layer.h
	src/softmax_layer.h
	src/stb_image.h
	src/tree.h
	src/stb_image_write.h
	src/upsample_layer.h
	src/utils.h
	src/yolo_layer.h
)

# CUDA files
set(CUDA_FILES
	src/activation_kernels.cu
	src/convolutional_kernels.cu
	src/im2col_kernels.cu
	src/avgpool_layer_kernels.cu
	src/crop_layer_kernels.cu
	src/maxpool_layer_kernels.cu
	src/blas_kernels.cu
	src/deconvolutional_kernels.cu
	src/col2im_kernels.cu
	src/dropout_layer_kernels.cu
)

# Source files
set(SOURCE_FILES 
	src/activation_layer.c
	src/activations.c
	src/avgpool_layer.c
	src/batchnorm_layer.c
	src/blas.c
	src/box.c
	src/col2im.c
	src/connected_layer.c
	src/convolutional_layer.c
	src/cost_layer.c
	src/crnn_layer.c
	src/crop_layer.c
	src/cuda.c
	src/data.c
	src/deconvolutional_layer.c
	src/demo.c
	src/detection_layer.c
	src/dropout_layer.c
	src/gemm.c
	src/gru_layer.c
	src/im2col.c
	src/image.c
	src/iseg_layer.c
	src/l2norm_layer.c
	src/layer.c
	src/list.c
	src/local_layer.c
	src/logistic_layer.c
	src/lstm_layer.c
	src/matrix.c
	src/maxpool_layer.c
	src/network.c
	src/normalization_layer.c
	src/option_list.c
	src/parser.c
	src/region_layer.c
	src/reorg_layer.c
	src/rnn_layer.c
	src/route_layer.c
	src/shortcut_layer.c
	src/softmax_layer.c
	src/tree.c
	src/upsample_layer.c
	src/utils.c
	src/yolo_layer.c
	src/image_opencv.cpp
	examples/darknet.c
	#examples/attention.c
	examples/art.c
	examples/captcha.c
	examples/cifar.c
	examples/classifier.c
	examples/coco.c
	examples/detector.c
	#examples/dice.c
	examples/go.c
	examples/instance-segmenter.c
	examples/lsd.c
	examples/nightmare.c
	examples/regressor.c
	examples/rnn.c
	#examples/rnn_vid.c
	examples/segmenter.c
	examples/super.c
	#examples/swag.c
	examples/tag.c
	#examples/voxel.c
	#examples/writing.c
	examples/yolo.c
)

# Add target for CUDA, refer to OpenCV CMakeLists.txt

add_library(CUDA INTERFACE)
if(WITH_CUDA)
    find_package(CUDA)
    target_include_directories(CUDA INTERFACE ${CUDA_INCLUDE_DIRS})
    target_link_libraries(CUDA INTERFACE ${CUDA_LIBRARIES} ${CUDA_curand_LIBRARY} ${CUDA_CUBLAS_LIBRARIES} ${CUDA_npp_LIBRARY})
    
    add_definitions(-DGPU)
    if(WITH_CUDNN)
	add_definitions(-DCUDNN)
    endif()

    # Pass options to NVCC
    set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} --compiler-options -fno-strict-aliasing -lineinfo -use_fast_math -Xptxas -dlcm=cg")
#   list(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_50,code=sm_50)    #for maxwell
#   list(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_52,code=sm_52)    #for maxwell
#   list(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_60,code=sm_60)    #for pascal
    set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -gencode arch=compute_61,code=sm_61")    #for pascal
#   list(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_62,code=sm_62)    #for jetson tx2
#   list(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_75,code=sm_75)    #for turing
endif()

# Add target for OpenCV 
add_library(OpenCV INTERFACE)
if(WITH_OPENCV)
    find_package(OpenCV)
    if(WIN32)
    elseif(UNIX)
        target_include_directories(OpenCV INTERFACE ${OpenCV_INCLUDE_DIRS})
        target_link_libraries(OpenCV INTERFACE ${OpenCV_LIBRARIES})
    endif()

    add_definitions(-DOPENCV)
endif()

message(STATUS ${CUDA_LIBRARIES})
message(STATUS ${CUDA_CUBLAS_LIBRARIES})
message(STATUS ${CUDA_curand_LIBRARY})
message(STATUS ${CUDA_npp_LIBRARY})
message(STATUS ${OpenCV_LIBRARIES})
message(STATUS ${PROJECT_SOURCE_DIR}/include)

# OpenMP Parallel framework
if(WITH_OPENMP)
    find_package(OpenMP)
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")

    add_definitions(-DOPENMP)
endif()


if(UNIX)
    # add the executable
    if(WITH_CUDA)
	# c파일은 gcc로 컴파일, cu 파일은 nvcc로 컴파일
        # CUDA_INCLUDE_DIRS가 자동으로 include_directories()
        cuda_add_executable(${bin_name} ${SOURCE_FILES} ${HEADER_FILES} ${CUDA_FILES})
	
    else()
	add_executable(${bin_name} ${SOURCE_FILES} ${HEADER_FILES})
    endif()
    #cuda_add_library(YoloV3 OpenCV)
    # nvcc 영향 없음
    target_link_libraries(${bin_name} 
	#${OpenCV_LIBRARIES} 
	#${CUDA_LIBRARIES}
	#${CUDA_curand_LIBRARY} 
	#${CUDA_CUBLAS_LIBRARIES}
	#${CUDA_npp_LIBRARY}
	#/etc/alternatives/libcudnn.so
	pthread
	m
    )

    # Directory to the list of paths to search for include files
    # nvcc 영향 없음
    target_include_directories(${bin_name} PRIVATE
	#${OpenCV_INCLUDE_DIRS}
	${PROJECT_SOURCE_DIR}/include
 	#${CUDA_INCLUDE_DIRS}
	#/etc/alternatives/cudnn.h
    )

endif(UNIX)
