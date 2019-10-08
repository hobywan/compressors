#
# Copyright (c) 2019, Hoby Rakotoarivelo

cmake_minimum_required(VERSION 3.5)
cmake_policy(SET CMP0048 NEW)
project(fpzip  VERSION 1.2.0 LANGUAGES CXX C)

# set build-type dependent options
set(DEFAULT_BUILD_TYPE "Release")
if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
	message(STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified.")
	set(CMAKE_BUILD_TYPE "${DEFAULT_BUILD_TYPE}" CACHE STRING "Choose the type of build." FORCE)
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif ()

add_library(fpzip STATIC)

# Options
set(FPZIP_FP "FPZIP_FP_FAST")    # optimize for speed; may impact correctness and portability
#set(FPZIP_FP "FPZIP_FP_SAFE")   # use volatile accumulator
#set(FPZIP_FP "FPZIP_FP_EMUL")   # emulate floating-point arithmetic
#set(FPZIP_FP "FPZIP_FP_INT")    # reinterpret floating-point numbers as integers; most portable
set(FPZIP_BLOCK_SIZE "0x1000")   # output buffer size (ideally the disk block size)
#set(FPZIP_BLOCK_SIZE "1")
set(FPZIP_CONV "-DWITH_UNION")   # bitwise type conversion mechanisms (defaults to memcpy)
#set(FPZIP_CONV "-DWITH_REINTERPRET_CAST")

add_compile_options("-fPIC")

target_include_directories(fpzip PUBLIC
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

target_sources(fpzip PRIVATE
  src/error.cpp
  src/rcdecoder.cpp
	src/rcencoder.cpp
	src/rcqsmodel.cpp
	src/read.cpp
	src/write.cpp
)

target_compile_definitions(fpzip PRIVATE -DFPZIP_FP=${FPZIP_FP})
target_compile_definitions(fpzip PRIVATE -DFPZIP_BLOCK_SIZE=${FPZIP_BLOCK_SIZE})
target_compile_definitions(fpzip PRIVATE ${FPZIP_CONV})

add_executable(fpzip_example)
add_executable(fpzip_test)

target_sources(fpzip_example PRIVATE examples/fpzip.cpp)
target_include_directories(fpzip_example PRIVATE ${CMAKE_SOURCE_DIR}/include)
target_link_libraries(fpzip_example fpzip)

target_sources(fpzip_test PRIVATE examples/testfpzip.c)
target_include_directories(fpzip_test PRIVATE ${CMAKE_SOURCE_DIR}/include)
target_link_libraries(fpzip_test fpzip)

# export target from build/install trees
export(TARGETS fpzip FILE fpzip-config.cmake)
export(PACKAGE fpzip)

install(DIRECTORY "${CMAKE_SOURCE_DIR}/include"
        DESTINATION "${CMAKE_INSTALL_PREFIX}"
        FILES_MATCHING PATTERN "*.h" PATTERN "*.inl")

install(TARGETS fpzip
        EXPORT fpzip-targets
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_PREFIX}/include
        ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
        LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
        RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)

install(EXPORT fpzip-targets
        FILE fpzip-config.cmake
        DESTINATION ${CMAKE_INSTALL_PREFIX}/cmake)
