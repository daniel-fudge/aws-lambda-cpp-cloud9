cmake_minimum_required(VERSION 3.21)

set(CMAKE_CXX_STANDARD 11)
project(demo LANGUAGES CXX)

find_package(aws-lambda-runtime)
find_package(AWSSDK COMPONENTS core)

add_subdirectory(src)
