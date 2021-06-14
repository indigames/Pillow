cmake_minimum_required(VERSION 3.15.0)

set(TARGET_NAME pillow-static)

project(${TARGET_NAME})

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)

# Modules and install path (required by conan)
set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/install)
set(CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR} ${CMAKE_MODULE_PATH})
set(CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR} ${CMAKE_PREFIX_PATH})

if(DEFINED ANDROID_PLATFORM)
    set (ANDROID 1)
    if(ANDROID_ABI STREQUAL "armeabi-v7a")
        add_compile_options(-marm -march=armv7-a -mtune=cortex-a15 -mfpu=neon-fp16 -mfloat-abi=softfp)
    elseif(ANDROID_ABI STREQUAL "x86")
        add_compile_options(-march=atom -m32 -msse -msse2 -msse3 -mssse3 -mfpmath=sse -mstackrealign)
    elseif(ANDROID_ABI STREQUAL "arm64-v8a")
        add_compile_options(-march=armv8-a+simd+sve -mtune=cortex-a53 -D__ARM_NEON -D__ARM_NEON__)
    elseif(ANDROID_ABI STREQUAL "x86_64")
        add_compile_options(-march=atom -m64 -msse -msse2 -msse3 -mssse3 -msse4.1 -msse4.2 -mfpmath=sse)
    endif()
    if(CMAKE_BUILD_TYPE STREQUAL "Release")
        add_compile_options(-g0 -Ofast)
    else(CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(-Og)
    endif()
elseif(MSVC)
    if(${APP_STYLE} MATCHES "SHARED")
        set(CMAKE_CXX_FLAGS "/MD ${CMAKE_CXX_FLAGS}")
        set(CMAKE_CXX_FLAGS_RELEASE "/MD /O2 /Ob2 /Zc:inline")
        set(CMAKE_CXX_FLAGS_DEBUG "/MD /JMC /ZI /Od")
    endif()
    add_compile_options(/MP /std:c++17)
elseif(IOS)
    if(CMAKE_BUILD_TYPE STREQUAL "Release")
        add_compile_options(-g0 -Ofast)
    endif()
endif()

if(${APP_STYLE} MATCHES "SHARED")
    find_package (Python3 ${PYTHON_VERSION} EXACT REQUIRED COMPONENTS Development)
else()
    find_package(Python REQUIRED)
    set (Python3_INCLUDE_DIRS ${Python_INCLUDE_DIRS})
    set (Python3_LIBRARIES ${Python_LIBRARIES})
endif()

if(MSVC)
    find_package(zlib REQUIRED)
    set(ZLIB_FOUND ${zlib_FOUND})
    set(ZLIB_LIBRARIES ${zlib_LIBRARIES})
    set(ZLIB_INCLUDE_DIRS ${zlib_INCLUDE_DIRS})
endif()

set(PYTHON_INCLUDE_DIRS ${Python3_INCLUDE_DIRS})

set(PNG_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/depends/libpng")
set(HARFBUZZ_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/depends/harfbuzz")
set(HARFBUZZ_LIBRARIES "${CMAKE_CURRENT_SOURCE_DIR}/depends/harfbuzz")

file(GLOB_RECURSE SRC
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c"
)

set(INCLUDES
    "${CMAKE_CURRENT_SOURCE_DIR}"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/freetype"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/freetype/include"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/fribidi"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/fribidi/lib"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/harfbuzz"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/harfbuzz/src"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/harfbuzz/util"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/lcms2"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/lcms2/include"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/lcms2/src"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/libimagequant"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/libraqm"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/libraqm/src"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/zlib"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/libjpeg"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/libpng"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/libwebp"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/libwebp/src"
    "${CMAKE_CURRENT_SOURCE_DIR}/depends/libtiff"
    "${PYTHON_INCLUDE_DIRS}"
    "${zlib_INCLUDE_DIRS}"
)

add_library(${TARGET_NAME} STATIC ${SRC})

target_include_directories(${TARGET_NAME} PRIVATE ${INCLUDES})
target_compile_definitions(${TARGET_NAME} PUBLIC -DPILLOW_VERSION="8.2.0.dev0" -DHAVE_LIBJPEG=1 -DHAVE_LIBZ=1 -DHAVE_LIBTIFF=1)

set(SKIP_INSTALL_ALL TRUE)
add_subdirectory(depends/freetype)
add_subdirectory(depends/fribidi)
# add_subdirectory(depends/harfbuzz)
add_subdirectory(depends/lcms2)
add_subdirectory(depends/libimagequant)
add_subdirectory(depends/libraqm)
# add_subdirectory(depends/tiff)
# add_subdirectory(depends/zlib)
# add_subdirectory(depends/libjpeg-turbo)
# add_subdirectory(depends/libpng)


# Install Targets
install(TARGETS
    ${TARGET_NAME}
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    PUBLIC_HEADER DESTINATION include
)

file(GLOB_RECURSE PUBLIC_INCLUDE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/*.h*")
foreach(HEADER ${PUBLIC_INCLUDE_FILES})
    file(RELATIVE_PATH HEADER_RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}/" "${HEADER}")
    get_filename_component(DIR ${HEADER_RELATIVE} DIRECTORY)
    install(FILES ${HEADER} DESTINATION include/${DIR})
endforeach()

