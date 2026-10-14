cmake_minimum_required(VERSION 3.24)
project(CudaBilliardsSimulation LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
option(BUILD_CUDA "Build the CUDA OpenGL implementation" OFF)

find_package(OpenGL REQUIRED)
find_package(GLUT REQUIRED)
if(NOT TARGET OpenGL::GLU)
  message(FATAL_ERROR "GLU is required for gluPerspective and gluLookAt")
endif()

add_executable(billiards_serial src/serialCode.cpp)
target_link_libraries(billiards_serial PRIVATE OpenGL::GL OpenGL::GLU GLUT::GLUT)

if(BUILD_CUDA)
  enable_language(CUDA)
  find_package(CUDAToolkit REQUIRED)
  find_package(GLEW REQUIRED)
  add_executable(billiards_cuda src/CudaCode.cu)
  set_target_properties(billiards_cuda PROPERTIES CUDA_STANDARD 17 CUDA_STANDARD_REQUIRED ON)
  target_link_libraries(billiards_cuda PRIVATE CUDA::cudart OpenGL::GL OpenGL::GLU GLUT::GLUT GLEW::GLEW)
endif()
