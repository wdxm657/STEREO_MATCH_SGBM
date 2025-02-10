cmake_minimum_required( VERSION 2.8 )
project(SemiGlobalMatching)

set(CMAKE_BUILD_TYPE "Release")

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
#if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++11.")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pthread  -ggdb  -O0 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread  -ggdb   -O0 -march=native ")

set(OpenCV_DIR "/usr/local/include/opencv-4.11.0/share/opencv4")
find_package( OpenCV 4.11 REQUIRED )
find_package(PCL REQUIRED)

include(FetchContent)
FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG        11.1.2  # 或者选择特定的版本，如 v8.0.1
)
FetchContent_MakeAvailable(fmt)

message( ${OpenCV_VERSION} )
message( ${OpenCV_INSTALL_PATH} )

message(${PCL_VERSION})

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

include_directories( 
    ${PROJECT_SOURCE_DIR}
    ${PROJECT_SOURCE_DIR}/include
    ${OpenCV_INCLUDE_DIRS}
    ${PCL_INCLUDE_DIRS}
)

message( ${OpenCV_INCLUDE_DIRS} )

add_library(semig STATIC
    src/semig/SemiGlobalMatching.cpp
    src/semig/sgm_util.cpp
)
target_link_libraries(semig PRIVATE fmt::fmt)

add_executable(semi_global_matching src/main.cpp)
target_link_libraries(semi_global_matching ${OpenCV_LIBS} ${PCL_LIBRARIES} semig fmt::fmt)
# 可将cpp仿真出的cost去掉每个像素位之间的空格，方便进入verilog仿真
# add_executable(rw_txt src/simulation/rw_txt.cpp)
# 读入verilog仿真生成的census l和r的txt文件，生成视差图，方便verilog cencus的仿真验证
# add_executable(cencus_in src/simulation/cencus_in.cpp)
# target_link_libraries(cencus_in ${OpenCV_LIBS} ${PCL_LIBRARIES})
# 读入verilog仿真生成基于Hamming-Distance的原始代价的txt文件，生成视差图，方便verilog hamming_distance的仿真验证
# add_executable(cost_init_in src/simulation/cost_init_in.cpp)

add_subdirectory(Stereo-Vision)
message("PROJECT_LIB_DIR : ${PROJECT_LIB_DIR}")
