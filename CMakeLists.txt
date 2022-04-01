cmake_minimum_required(VERSION 3.21)

set(CMAKE_CXX_STANDARD 20)
project(demo)

find_package(aws-lambda-runtime REQUIRED)
find_package(AWSSDK COMPONENTS core)

add_subdirectory(src)
