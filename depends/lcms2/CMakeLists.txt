cmake_minimum_required(VERSION 3.5.0)

project(lcms2)

include_directories ( include )
file ( GLOB LCMS_SRCS "src/*.c" )
add_library ( lcms2 STATIC ${LCMS_SRCS} )
