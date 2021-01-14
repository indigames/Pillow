cmake_minimum_required(VERSION 3.4.1)

project(libraqm)

# find_library(fribidi NAMES fribidi)
# find_library(harfbuzz NAMES harfbuzz)
# find_library(freetype NAMES freetype)

add_definitions(-DFRIBIDI_LIB_STATIC)


function(raqm_conf)
	file(READ configure.ac RAQM_CONF)
	string(REGEX MATCH "\\[([0-9]+)\\.([0-9]+)\\.([0-9]+)\\]," _ "${RAQM_CONF}")
	set(RAQM_VERSION_MAJOR "${CMAKE_MATCH_1}")
	set(RAQM_VERSION_MINOR "${CMAKE_MATCH_2}")
	set(RAQM_VERSION_MICRO "${CMAKE_MATCH_3}")
	set(RAQM_VERSION "${RAQM_VERSION_MAJOR}.${RAQM_VERSION_MINOR}.${RAQM_VERSION_MICRO}")
	message("detected libraqm version ${RAQM_VERSION}")
	configure_file(src/raqm-version.h.in src/raqm-version.h @ONLY)
endfunction()
raqm_conf()


# set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
set(RAQM_SOURCES
	src/raqm.c)

set(RAQM_HEADERS
	src/raqm.h
	src/raqm-version.h)

add_library(libraqm STATIC
	${RAQM_SOURCES}
	${RAQM_HEADERS})

# target_link_libraries(libraqm
	# ${fribidi}
	# ${harfbuzz}
	# ${freetype})

target_include_directories(libraqm PRIVATE 
	"${CMAKE_CURRENT_SOURCE_DIR}/src"
	"${CMAKE_CURRENT_SOURCE_DIR}/../fribidi"
	"${CMAKE_CURRENT_SOURCE_DIR}/../fribidi/lib"
	"${CMAKE_CURRENT_SOURCE_DIR}/../harfbuzz"
    "${CMAKE_CURRENT_SOURCE_DIR}/../harfbuzz/src"
    "${CMAKE_CURRENT_SOURCE_DIR}/../harfbuzz/util"
	"${CMAKE_CURRENT_SOURCE_DIR}/../freetype"
    "${CMAKE_CURRENT_SOURCE_DIR}/../freetype/include"
)
