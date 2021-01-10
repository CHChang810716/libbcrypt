###################################################################################
#
#  Copyright (c) 2014, webvariants GmbH, http://www.webvariants.de
#
#  This file is released under the terms of the MIT license. You can find the
#  complete text in the attached LICENSE file or online at:
#
#  http://www.opensource.org/licenses/mit-license.php
#
#  @author: Tino Rusch (tino.rusch@webvariants.de)
#
###################################################################################

cmake_minimum_required(VERSION 3.10 FATAL_ERROR)

include(cmake/HunterGate.cmake)
set(
    HUNTER_CACHE_SERVERS
    "https://github.com/CHChang810716/hunter-binary-cache"
    CACHE
    STRING
    "Hunter cache servers"
)
HunterGate(
    URL "https://github.com/CHChang810716/hunter.git"
    SHA1 001f05eefbd1b3f0edb3c73d9c50f3f20190a1fb
)

project(bcrypt 
  LANGUAGES C CXX ASM
  VERSION 0.1.0.0
)
if (CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)|(amd64)|(AMD64)")
  message(STATUS "Processor support x86")
else()
  message(FATAL "Package uses x86 ASM, processor must support x86")
endif()
option(ENABLE_TEST "enable build test case" OFF)
option(BUILD_SHARED_LIBS "build shared library instead of static archive" OFF)

if (NOT BUILD_SHARED_LIBS)
  message (STATUS "Building a static library")
else ()
  message (STATUS "Building a shared library")
endif ()

if(GNU)
  set( CMAKE_ASM_FLAGS "${CXXFLAGS} -x assembler-with-cpp")
endif()

set( SRCFILES
  ${CMAKE_CURRENT_SOURCE_DIR}/src/bcrypt.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/crypt_blowfish.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/crypt_gensalt.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/wrapper.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/x86.S
)

add_library(
  ${PROJECT_NAME}
  ${SRCFILES}
)
target_include_directories(${PROJECT_NAME} PUBLIC 
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_11)
if (NOT WIN32)
  target_include_directories(${PROJECT_NAME} PRIVATE 
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src>
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include/bcrypt>
  )
endif()
list(APPEND inst_targets ${PROJECT_NAME})

if(ENABLE_TEST)
  add_executable( ${PROJECT_NAME}_test ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp)
  target_link_libraries( ${PROJECT_NAME}_test ${PROJECT_NAME})
endif()

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)
set(AKT_GEN_DIR             "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(AKT_CONFIG_INSTALL_DIR  "lib/cmake/${PROJECT_NAME}")
set(AKT_CONFIG_VERSION      "${AKT_GEN_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(AKT_PROJECT_CONFIG      "${AKT_GEN_DIR}/${PROJECT_NAME}Config.cmake")
set(AKT_TARGETS_EXPORT      "${PROJECT_NAME}Targets")
set(AKT_NAMESPACE           "${PROJECT_NAME}::")
configure_package_config_file(
    cmake/Config.cmake.in
    ${AKT_PROJECT_CONFIG}
    INSTALL_DESTINATION ${AKT_CONFIG_INSTALL_DIR}
)
write_basic_package_version_file(
    ${AKT_CONFIG_VERSION} COMPATIBILITY SameMajorVersion
)
install(
    FILES ${AKT_PROJECT_CONFIG}
    DESTINATION ${AKT_CONFIG_INSTALL_DIR}
)
install(
    TARGETS ${inst_targets}
    EXPORT ${AKT_TARGETS_EXPORT}
    BUNDLE DESTINATION . COMPONENT Runtime
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT Runtime
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Library
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Develop
)
install(
    EXPORT ${AKT_TARGETS_EXPORT}
    NAMESPACE ${AKT_NAMESPACE}
    DESTINATION ${AKT_CONFIG_INSTALL_DIR}
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  FILES_MATCHING PATTERN "*.h")