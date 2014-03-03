cmake_minimum_required(VERSION 2.8)

## default architecture
set(ARCH "x86_64" CACHE STRING "Architecture to tell gcc to optimize for (-march)")

##
## Package-wide configuration
##
project(libserial)
set (pkg_NAME "libserial")
set(pkg_BRIEF "RoadNarrows serial library.")

set(pkg_VERSION_MAJ 1)
set(pkg_VERSION_MIN 4)
set(pkg_VERSION_REV 0)
set(pkg_VERSION "${pkg_VERSION_MAJ}.${pkg_VERSION_MIN}.${pkg_VERSION_REV}")

set(pkg_CREATED "2005")
execute_process(COMMAND date 
  OUTPUT_VARIABLE pkg_TIMESTAMP 
  OUTPUT_STRIP_TRAILING_WHITESPACE
)

set(pkg_FULL_NAME "${pkg_NAME}-${pkg_VERSION}")
set(pkg_AUTHORS "Robin Knight, Kim Wheeler, Daniel Packard")
set(pkg_OWNERS "RoadNarrows LLC")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DLOG -DLOGMOD='\"libserial\"'")

set(CMAKE_C_FLAGS "-DARCH_${ARCH} -DARCH='\"${ARCH}\"' ${CMAKE_C_FLAGS}")

## jenkins-specific configuration
if($ENV{JENKINS})
  set(CMAKE_INSTALL_PREFIX "$ENV{JENKINS_XINSTALL}/${ARCH}")
  include_directories(BEFORE ${CMAKE_INSTALL_PREFIX}/include)
  link_directories(${CMAKE_INSTALL_PREFIX}/lib/rnr)
endif($ENV{JENKINS})

## roadnarrows development-specific configuration
if($ENV{RNDEV})
  set(CMAKE_INSTALL_PREFIX "/prj/xinstall/${ARCH}")
  message(${CMAKE_INSTALL_PREFIX})
  include_directories(BEFORE ${CMAKE_INSTALL_PREFIX}/include)
  link_directories(${CMAKE_INSTALL_PREFIX}/lib/rnr)
endif($ENV{RNDEV})

## generate package headers
configure_file (
  "${PROJECT_SOURCE_DIR}/include/version.h.in"
  "${PROJECT_BINARY_DIR}/include/version.h"
)

## add project include directories
include_directories(
  ${PROJECT_SOURCE_DIR}/include
  ${PROJECT_BINARY_DIR}/include
)

## add libraries and other binaries
set(libserial_SOURCES 
  src/libserial/serdev.c
)

add_library(serial-static STATIC ${libserial_SOURCES})
set_target_properties(serial-static 
  PROPERTIES OUTPUT_NAME serial
  CLEAN_DIRECT_OUTPUT 1
)

add_library(serial-shared SHARED ${libserial_SOURCES})
set_target_properties(serial-shared 
  PROPERTIES OUTPUT_NAME serial
  CLEAN_DIRECT_OUTPUT 1
)

## add libraries/binaries/headers to installation
install(TARGETS serial-static serial-shared DESTINATION lib/rnr)
install(FILES  
    include/rnr/serdev.h
  DESTINATION 
    include/rnr
)

## configure debain package
SET(CPACK_GENERATOR "DEB")
SET(CPACK_DEBIAN_PACKAGE_NAME "libserial")
SET(CPACK_DEBIAN_PACKAGE_DESCRIPTION "RoadNarrows Robotics serial library")
SET(CPACK_DEBIAN_PACKAGE_DEPENDS "librnr-dev(>=1.12.0)")
SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "Daniel Packard <daniel@roadnarrows.com>")
SET(CPACK_PACKAGE_VERSION ${pkg_VERSION})
SET(CPACK_PACKAGE_DESCRIPTION_FILE ${PROJECT_SOURCE_DIR}/README.md) 
SET(CPACK_RESOURCE_FILE_LICENSE ${PROJECT_SOURCE_DIR}/LICENSE)
include(CPack)
