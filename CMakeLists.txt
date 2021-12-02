cmake_minimum_required(VERSION 3.13)

project(wisp 
    LANGUAGES C CXX ASM 
    VERSION 0.1) 

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

add_definitions(-DUSE_STD -DHAS_LIBM)

add_executable(${CMAKE_PROJECT_NAME}
    wisp.cpp
 )
