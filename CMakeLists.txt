cmake_minimum_required(VERSION 3.18)

# Project declaration with C++ and CUDA support
project(YOLOv11TRT LANGUAGES CXX CUDA)

# Set C++ standard to C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CUDA_COMPILER "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.8/bin/nvcc.exe")

# Define the path to TensorRT installation
set(TENSORRT_PATH "C:/Users/colin/Downloads/TensorRT-10.8.0.43.Windows.win10.cuda-12.8/TensorRT-10.8.0.43")  # Update this to the actual path for TensorRT

# Define the path to OpenCV installation

# set(OpenCV_INCLUDE_DIRS "C:/ws/vcpkg/installed/x64-windows/include/opencv4")

set(CMAKE_TOOLCHAIN_FILE "C:/ws/vcpkg/scripts/buildsystems/vcpkg.cmake" CACHE STRING "")

# Allow overriding TensorRT and OpenCV paths via command line
# e.g., cmake -DTENSORRT_PATH="path/to/TensorRT" -DOpenCV_DIR="path/to/OpenCV" ..
option(TENSORRT_PATH_OPTION "Path to TensorRT installation" ${TENSORRT_PATH})
set(TENSORRT_PATH ${TENSORRT_PATH_OPTION} CACHE PATH "Path to TensorRT installation")

set(CMAKE_PREFIX_PATH "C:/ws/vcpkg/installed/x64-windows")
set(Protobuf_DIR "C:/ws/vcpkg/installed/x64-windows/share/protobuf")
set(TIFF_DIR "C:/ws/vcpkg/installed/x64-windows/share/tiff")
set(quirc_DIR "C:/ws/vcpkg/installed/x64-windows/share/quirc")


# Set OpenCV path explicitly for vcpkg
set(OpenCV_DIR "C:/ws/vcpkg/installed/x64-windows/share/opencv")  # Adjust this path if needed

# Find OpenCV
find_package(OpenCV REQUIRED)
if(NOT OpenCV_FOUND)
    message(FATAL_ERROR "OpenCV not found. Please install OpenCV or set OpenCV_DIR.")
endif()

# Allow overriding TensorRT and OpenCV paths via command line
# Include directories
include_directories(${OpenCV_INCLUDE_DIRS})

# Find CUDA
find_package(CUDA REQUIRED)
if(NOT CUDA_FOUND)
    message(FATAL_ERROR "CUDA not found. Please install the CUDA Toolkit.")
endif()

# Include directories for TensorRT
include_directories(${TENSORRT_PATH}/include)

# Include directory for your project
include_directories(${CMAKE_SOURCE_DIR}/include)

# Define source files (including CUDA sources)
set(SOURCES
    main.cpp
    src/yolov11.cpp
    src/preprocess.cu
)

# Create executable (CMake handles CUDA sources automatically)
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

# Define API_EXPORTS macro
target_compile_definitions(${PROJECT_NAME} PRIVATE API_EXPORTS)

# Specify include directories (modern CMake approach)
target_include_directories(${PROJECT_NAME} PRIVATE
    src/
    ${OpenCV_INCLUDE_DIRS}
    ${CUDA_INCLUDE_DIRS}
    ${TENSORRT_PATH}/include
)

# Link TensorRT libraries
# Specify full paths to TensorRT libraries to avoid relying on link_directories
set(TENSORRT_LIBS
    "${TENSORRT_PATH}/lib/nvinfer.lib"
    "${TENSORRT_PATH}/lib/nvonnxparser.lib"
    "${TENSORRT_PATH}/lib/nvparsers.lib"
    "${TENSORRT_PATH}/lib/nvinfer_plugin.lib"
)

# Link libraries to the target
target_link_libraries(${PROJECT_NAME} PRIVATE
    ${OpenCV_LIBS}
    ${CUDA_LIBRARIES}
    ${TENSORRT_LIBS}
)

# Enable separable compilation for CUDA (optional but recommended)
set_target_properties(${PROJECT_NAME} PROPERTIES
    CUDA_SEPARABLE_COMPILATION ON
)

# (Optional) Specify CUDA architectures based on your GPU hardware
# set(CMAKE_CUDA_ARCHITECTURES 75)  # Example for Turing architecture

# (Optional) Set output directories for binaries
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
