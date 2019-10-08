#
# Copyright (c) 2019, Hoby Rakotoarivelo

cmake_minimum_required(VERSION 3.5)
cmake_policy(SET CMP0048 NEW)
project(isabela  VERSION 0.2.1 LANGUAGES CXX)

# set build-type dependent options
set(DEFAULT_BUILD_TYPE "Release")
if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
	message(STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified.")
	set(CMAKE_BUILD_TYPE "${DEFAULT_BUILD_TYPE}" CACHE STRING "Choose the type of build." FORCE)
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif ()

add_library(isabela STATIC)

# manage dependencies
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

set(OPENSSL_ROOT_DIR "/usr/local/opt/openssl")
find_package(zlib REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(GSL REQUIRED)

target_include_directories(isabela PUBLIC
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

target_include_directories(isabela PUBLIC ${GSL_INCLUDE_DIRS})
target_include_directories(isabela PUBLIC ${OPENSSL_INCLUDE_DIR})
target_include_directories(isabela PUBLIC ${ZLIB_INCLUDE_DIRS})
target_link_libraries(isabela PUBLIC ${ZLIB_LIBRARIES})
target_link_libraries(isabela PUBLIC GSL::gsl GSL::gslcblas)
target_link_libraries(isabela PUBLIC ${OPENSSL_LIBRARIES})

target_sources(isabela PRIVATE
	src/bspline/BSplineLib.cpp
	src/bspline/ReconstructClass.cpp
	src/bspline/spline_fit.cpp
	src/wavelets/haar.cpp
	src/wavelets/wavelets.cpp
	src/bitstream.cpp
	src/compressor_zlib_routines.cpp
	src/error.cpp
	src/isabela.cpp
	src/utils.cpp)

add_executable(isabela_file_compress)
target_sources(isabela_file_compress PRIVATE apps/file_compress.cpp)
target_include_directories(isabela_file_compress PRIVATE ${CMAKE_SOURCE_DIR}/include)
target_compile_definitions(isabela_file_compress PRIVATE -DCOMPRESS=1)
target_link_libraries(isabela_file_compress isabela)

add_executable(isabela_file_decompress)
target_sources(isabela_file_decompress PRIVATE apps/file_compress.cpp)
target_include_directories(isabela_file_decompress PRIVATE ${CMAKE_SOURCE_DIR}/include)
target_link_libraries(isabela_file_decompress isabela)

# export target from build/install trees
export(TARGETS isabela FILE isabela-config.cmake)
export(PACKAGE isabela)

install(DIRECTORY "${CMAKE_SOURCE_DIR}/include"
        DESTINATION "${CMAKE_INSTALL_PREFIX}"
			FILES_MATCHING PATTERN "*.h" PATTERN "*.inl")

install(TARGETS isabela
        EXPORT isabela-targets
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_PREFIX}/include
        ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
        LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
        RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)

install(EXPORT isabela-targets
        FILE isabela-config.cmake
        DESTINATION ${CMAKE_INSTALL_PREFIX}/cmake)
