#********************************************************************
#        _       _         _
#  _ __ | |_  _ | |  __ _ | |__   ___
# | '__|| __|(_)| | / _` || '_ \ / __|
# | |   | |_  _ | || (_| || |_) |\__ \
# |_|    \__|(_)|_| \__,_||_.__/ |___/
#
# www.rt-labs.com
# Copyright 2017 rt-labs AB, Sweden.
#
# This software is licensed under the terms of the BSD 3-clause
# license. See the file LICENSE distributed with this software for
# full license information.
#*******************************************************************/

cmake_minimum_required (VERSION 3.13)
set(CMAKE_MODULE_PATH "")
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/tools")
project (OSAL VERSION 0.1.0)

if (CMAKE_PROJECT_NAME STREQUAL OSAL)
  include(CTest)
endif()

# Set required standard level
set (CMAKE_C_STANDARD 99)
set (CMAKE_CXX_STANDARD 11)

# Always use standard .o suffix
set(CMAKE_C_OUTPUT_EXTENSION_REPLACE 1)
set(CMAKE_CXX_OUTPUT_EXTENSION_REPLACE 1)

# Generate version numbers
configure_file (
  version.h.in
  ${OSAL_BINARY_DIR}/src/version.h
  )

# Default to release build with debug info
if (NOT CMAKE_BUILD_TYPE)
  set (CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING
    "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
    FORCE)
endif (NOT CMAKE_BUILD_TYPE)
message(STATUS "Current build type is: ${CMAKE_BUILD_TYPE}")

# Default to installing in build directory
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX ${OSAL_BINARY_DIR}/install
    CACHE PATH "Default install path" FORCE)
endif()

# Add platform-dependent targets early, so they can be configured by
# platform
add_library(osal "")

if (CMAKE_PROJECT_NAME STREQUAL OSAL AND BUILD_TESTING)
  add_executable(osal_test "")
endif()

# Platform configuration
include(${CMAKE_SYSTEM_NAME})
message(STATUS "Building for ${CMAKE_SYSTEM_NAME}")

target_include_directories(osal
  PUBLIC
  $<BUILD_INTERFACE:${OSAL_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  PRIVATE
  src
  ${OSAL_BINARY_DIR}/src
  )

install(
  TARGETS osal
  EXPORT OsalConfig
  DESTINATION lib
  )

install(
  EXPORT OsalConfig
  DESTINATION cmake
  )

install(FILES
  include/osal.h
  include/osal_log.h
  DESTINATION include
  )

if (CMAKE_PROJECT_NAME STREQUAL OSAL AND BUILD_TESTING)
  add_subdirectory (test)
  include(AddGoogleTest)
  add_gtest(osal_test)
endif()

# Doxygen configuration
cmake_policy(SET CMP0057 NEW)
find_package(Doxygen)
if (CMAKE_PROJECT_NAME STREQUAL OSAL AND DOXYGEN_FOUND)
  set(DOXYGEN_OPTIMIZE_OUTPUT_FOR_C YES)
  set(DOXYGEN_TYPEDEF_HIDES_STRUCT YES)
  set(DOXYGEN_EXTRACT_STATIC YES)
  set(DOXYGEN_STRIP_CODE_COMMENTS NO)
  set(DOXYGEN_MACRO_EXPANSION YES)
  set(DOXYGEN_EXPAND_ONLY_PREDEF YES)
  set(DOXYGEN_PREDEFINED OSAL_EXPORT)
  set(DOXYGEN_EXPAND_AS_DEFINED OSAL_EXPORT)
  set(DOXYGEN_COLLABORATION_GRAPH NO)
  set(DOXYGEN_INCLUDE_GRAPH NO)
  set(DOXYGEN_INCLUDED_BY_GRAPH NO)
  set(DOXYGEN_RECURSIVE NO)
  set(DOXYGEN_USE_MDFILE_AS_MAINPAGE README.md)
  doxygen_add_docs(docs
    README.md
    include
    src
    )
endif()
