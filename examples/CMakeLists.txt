# Find Open Source Computer Vision (OpenCV)
find_package(OpenCV QUIET)
if(USE_OPENCV AND OpenCV_FOUND)
    add_definitions(-DCNN_USE_OPENCV)
    include_directories(${OpenCV_INCLUDE_DIRS})
    set(REQUIRED_LIBRARIES ${REQUIRED_LIBRARIES} ${OpenCV_LIBS})
elseif(USE_OPENCV AND NOT OpenCV_FOUND)
    # OpenCV is not a real dependency. In case is not found,
    # USE_OPENCV flag will be kept to OFF.
    message(WARNING "OpenCV not found. Building examples without it.")
endif()

add_executable(benchmarks_all benchmarks/main.cpp)
target_link_libraries(benchmarks_all
    ${project_library_target_name} ${REQUIRED_LIBRARIES})

add_executable(example_mnist_train mnist/train.cpp)
target_link_libraries(example_mnist_train
    ${project_library_target_name} ${REQUIRED_LIBRARIES})

add_executable(example_deconv_train deconv/train.cpp)
target_link_libraries(example_deconv_train
    ${project_library_target_name} ${REQUIRED_LIBRARIES})

if(CNN_USE_OPENCV)
    add_executable(example_mnist_test mnist/test.cpp)
    target_link_libraries(example_mnist_test
        ${project_library_target_name} ${REQUIRED_LIBRARIES})

    add_executable(example_deconv_visual deconv/visual.cpp)
    target_link_libraries(example_deconv_visual
        ${project_library_target_name} ${REQUIRED_LIBRARIES})
else(CNN_USE_OPENCV)
    file(GLOB stb_image_hdrs mnist/stb_image/*.h)
    add_executable(example_mnist_test mnist/test_with_stb_image.cpp
                   ${stb_image_hdrs})
    target_link_libraries(example_mnist_test
        ${project_library_target_name} ${REQUIRED_LIBRARIES})
endif(CNN_USE_OPENCV)

add_executable(example_cifar_train cifar10/train.cpp)
target_link_libraries(example_cifar_train
    ${project_library_target_name} ${REQUIRED_LIBRARIES})

find_package(Protobuf QUIET)
if(CNN_USE_OPENCV AND PROTOBUF_FOUND)
    set(proto_file "${CMAKE_SOURCE_DIR}/tiny_dnn/io/caffe/caffe.pb.cc")
    if(EXISTS ${proto_file})
        add_executable(example_caffe_converter
                       caffe_converter/caffe_converter.cpp ${proto_file})
        target_link_libraries(example_caffe_converter
            ${project_library_target_name}
            ${PROTOBUF_LIBRARIES} ${REQUIRED_LIBRARIES})
# As of Ubuntu 14.04 protoc is no longer a part of libprotobuf-dev package
# and should be installed separately as in: sudo apt-get install protobuf-compiler
    elseif(EXISTS ${PROTOBUF_PROTOC_EXECUTABLE})
        message(STATUS "Found PROTOBUF Compiler: ${PROTOBUF_PROTOC_EXECUTABLE}")
        if(EXISTS ${CMAKE_SOURCE_DIR}/tiny_dnn/io/caffe/caffe.proto)
            PROTOBUF_GENERATE_CPP(PROTO_SRCS PROTO_HDRS
                ${CMAKE_SOURCE_DIR}/tiny_dnn/io/caffe/caffe.proto)
            if(EXISTS ${PROTO_HDRS})
                file(COPY ${PROTO_SRCS} DESTINATION "${CMAKE_SOURCE_DIR}/tiny_dnn/io/caffe")
                file(COPY ${PROTO_HDRS} DESTINATION "${CMAKE_SOURCE_DIR}/tiny_dnn/io/caffe")
            endif()
            add_executable(example_caffe_converter
                           caffe_converter/caffe_converter.cpp
                           ${PROTO_SRCS} ${PROTO_HDRS})
            target_link_libraries(example_caffe_converter
                ${project_library_target_name}
                ${PROTOBUF_LIBRARIES} ${REQUIRED_LIBRARIES})
        else()
            message(STATUS "Cannot generate C++ proto files, please provide Protobuf file.")
        endif()
    else()
        message(STATUS "Proto is not linked correctly, please make sure file exists.")
    endif()
else(CNN_USE_OPENCV AND PROTOBUF_FOUND)
    message(STATUS "Cannot generate Caffe Importer. Please install Protobuf.")
endif(CNN_USE_OPENCV AND PROTOBUF_FOUND)
