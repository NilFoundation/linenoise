#---------------------------------------------------------------------------#
# Copyright (c) 2018-2020 Mikhail Komarov <nemo@nil.foundation>
#
# Distributed under the Boost Software License, Version 1.0
# See accompanying file LICENSE_1_0.txt or copy at
# http://www.boost.org/LICENSE_1_0.txt
#---------------------------------------------------------------------------#

cmake_minimum_required(VERSION 2.6)

cmake_policy(SET CMP0028 NEW)
cmake_policy(SET CMP0042 NEW)
cmake_policy(SET CMP0048 NEW)
cmake_policy(SET CMP0057 NEW)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake"
     "${CMAKE_CURRENT_LIST_DIR}/cmake/packages"
     "${CMAKE_CURRENT_LIST_DIR}/cmake/modules/share/modules/cmake")

include(CMConfig)
include(CMDeploy)
include(CMSetupVersion)

cm_project(linenoise)

if(UNIX AND BUILD_WITH_CCACHE)
    find_program(CCACHE_FOUND ccache)
    if(CCACHE_FOUND)
        set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
        set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
    endif(CCACHE_FOUND)
endif()

cm_setup_version(VERSION 0.1.0 PREFIX ${CURRENT_PROJECT_NAME})

# compiler options
if(CMAKE_COMPILER_IS_GNUCXX)
    set(${CURRENT_PROJECT_NAME}_BASE_COMPILER_OPTIONS "-std=c++17 -Wall -Wextra")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${BASE_COMPILER_OPTIONS}")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${BASE_COMPILER_OPTIONS} -O0 -g")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_MINSIZEREL "${CMAKE_CXX_FLAGS_MINSIZEREL} ${BASE_COMPILER_OPTIONS} -Os")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${BASE_COMPILER_OPTIONS} -O3 -fomit-frame-pointer")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO}
    ${BASE_COMPILER_OPTIONS} -O3 -g")

elseif(CMAKE_COMPILER_IS_CLANGCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    # using regular Clang or AppleClang
    set(${CURRENT_PROJECT_NAME}_BASE_COMPILER_OPTIONS "-std=c++17 -Wall -Wextra")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${BASE_COMPILER_OPTIONS}")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${BASE_COMPILER_OPTIONS} -O0 -g")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_MINSIZEREL "${CMAKE_CXX_FLAGS_MINSIZEREL} ${BASE_COMPILER_OPTIONS} -Os")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${BASE_COMPILER_OPTIONS} -O3 -fomit-frame-pointer")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} ${BASE_COMPILER_OPTIONS} -O3 -g")

elseif(MSVC)
    target_compile_definitions("-D_CRT_SECURE_NO_WARNINGS=1")

    foreach(flag_var
            ${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS
            ${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_DEBUG
            ${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_RELEASE
            ${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_MINSIZEREL
            ${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_RELWITHDEBINFO)
        if(flag_var MATCHES "DEBUG")
            set(${flag_var} "${${flag_var}} /MTd")
        else()
            set(${flag_var} "${${flag_var}} /MT")
        endif()
    endforeach()
    # https://msdn.microsoft.com/en-us/library/aa267384%28VS.60%29.aspx
    set(${CURRENT_PROJECT_NAME}_CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /INCREMENTAL:NO /SUBSYSTEM:CONSOLE /LTCG /ignore:4099 /NODEFAULTLIB:libc.lib /NODEFAULTLIB:libcmt.lib /NODEFAULTLIB:msvcrt.lib /NODEFAULTLIB:libcd.lib /NODEFAULTLIB:msvcrtd.lib")
    set(${CURRENT_PROJECT_NAME}_CMAKE_EXE_LINKER_FLAGS_MINSIZEREL "${CMAKE_EXE_LINKER_FLAGS_MINSIZEREL} /SUBSYSTEM:CONSOLE /ignore:4099 /NODEFAULTLIB:libc.lib /NODEFAULTLIB:libcmtd.lib /NODEFAULTLIB:msvcrt.lib /NODEFAULTLIB:libcd.lib /NODEFAULTLIB:msvcrtd.lib")
    set(${CURRENT_PROJECT_NAME}_CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /SUBSYSTEM:CONSOLE /ignore:4099 /NODEFAULTLIB:libc.lib /NODEFAULTLIB:libcmtd.lib /NODEFAULTLIB:msvcrt.lib /NODEFAULTLIB:libcd.lib /NODEFAULTLIB:msvcrtd.lib")
    set(${CURRENT_PROJECT_NAME}_CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO "${CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO} /SUBSYSTEM:CONSOLE /ignore:4099 /NODEFAULTLIB:libc.lib /NODEFAULTLIB:libcmtd.lib /NODEFAULTLIB:msvcrt.lib /NODEFAULTLIB:libcd.lib /NODEFAULTLIB:msvcrtd.lib")
else()
    set(${CURRENT_PROJECT_NAME}_BASE_COMPILER_OPTIONS "-std=c++17 -Wall -Wextra")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS "${BASE_COMPILER_OPTIONS}")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_DEBUG "${BASE_COMPILER_OPTIONS} -O0 -g")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_MINSIZEREL "${BASE_COMPILER_OPTIONS} -Os")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_RELEASE "${BASE_COMPILER_OPTIONS} -O3 -fomit-frame-pointer")
    set(${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_RELWITHDEBINFO "${BASE_COMPILER_OPTIONS} -O3 -g")
endif()

# build liblinenoise
add_library(${CURRENT_PROJECT_NAME}
            src/ConvertUTF.cpp
            src/linenoise.cpp
            src/wcwidth.cpp)

set_target_properties(${CURRENT_PROJECT_NAME} PROPERTIES
                      EXPORT_NAME ${CURRENT_PROJECT_NAME}
                      LANGUAGE C
                      CXX_STANDARD 17)

target_compile_options(${CURRENT_PROJECT_NAME} PUBLIC
                       $<$<STREQUAL:${CMAKE_BUILD_TYPE},"Debug">:"${${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_DEBUG}">
                       $<$<STREQUAL:${CMAKE_BUILD_TYPE},"MinSizeRel">:"${${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_MINSIZEREL}">
                       $<$<STREQUAL:${CMAKE_BUILD_TYPE},"Release">:"${${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_RELEASE}">
                       $<$<STREQUAL:${CMAKE_BUILD_TYPE},"RelWithDebInfo">:"${${CURRENT_PROJECT_NAME}_CMAKE_CXX_FLAGS_RELWITHDEBINFO}">)

target_link_libraries(${CURRENT_PROJECT_NAME} INTERFACE

                      ${Boost_LIBRARIES})

target_include_directories(${CURRENT_PROJECT_NAME} INTERFACE
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                           $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/include>

                           ${Boost_INCLUDE_DIRS})

cm_deploy(TARGETS ${CURRENT_PROJECT_NAME}
          INCLUDE include
          NAMESPACE "")

if(BUILD_TESTS)
    add_subdirectory(test)
endif()
