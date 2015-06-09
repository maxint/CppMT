cmake_minimum_required (VERSION 2.6)

# enable debug info in release mode
function(enable_release_debug)
  foreach(_target ${ARGN})
    get_target_property(_link_flags ${_target} LINK_FLAGS)
    get_target_property(_compile_flags ${_target} COMPILE_FLAGS)

    if(NOT _link_flags)
      set(_link_flags)
    endif()
    if(NOT _compile_flags)
      set(_compile_flags)
    endif()

    if(MSVC)
      set_target_properties(${_target} PROPERTIES
        LINK_FLAGS "${_link_flags} /DEBUG"
        COMPILE_FLAGS "${_compile_flags} /Z7"
        )
    else()
      # TODO:
    endif()
  endforeach()
endfunction()

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

project(CMT CXX)

find_package(OpenCV REQUIRED)

# CMT
file(GLOB CMT_SRCS_ALL *.h *.cpp
    fastcluster/fastcluster.h
    fastcluster/fastcluster.cpp
    logging/log.h
    )
set(CMT_SRCS ${CMT_SRCS_ALL})
list(REMOVE_ITEM CMT_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/trax.cpp)
if(WIN32)
    list(APPEND CMT_SRCS getopt/getopt.h getopt/getopt.cpp)
endif()

add_executable(cmt ${CMT_SRCS})
target_link_libraries(cmt ${OpenCV_LIBS})
enable_release_debug(cmt)


# TRAX Client
option(BUILD_TRAX_CLIENT "Build the trax client." OFF)

if(BUILD_TRAX_CLIENT)
    set(TRAX_DIR "" CACHE FILEPATH "Path to trax")
    include_directories(${TRAX_DIR}/include)
    add_executable(trax_client cmt ${CMT_SRCS_ALL})
    find_library(TRAX_LIBRARY NAMES trax HINTS "${TRAX_DIR}/lib")
    target_link_libraries(trax_client ${OpenCV_LIBS} ${TRAX_LIBRARY})
    enable_release_debug(trax_client)
endif()

# vim: et ts=4 sts=4 sw=4
