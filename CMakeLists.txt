##
##     This file is part of qpOASES.
##
##     qpOASES -- An Implementation of the Online Active Set Strategy.
##     Copyright (C) 2007-2017 by Hans Joachim Ferreau, Andreas Potschka,
##     Christian Kirches et al. All rights reserved.
##
##     qpOASES is free software; you can redistribute it and/or
##     modify it under the terms of the GNU Lesser General Public
##     License as published by the Free Software Foundation; either
##     version 2.1 of the License, or (at your option) any later version.
##
##     qpOASES is distributed in the hope that it will be useful,
##     but WITHOUT ANY WARRANTY; without even the implied warranty of
##     MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
##     See the GNU Lesser General Public License for more details.
##
##     You should have received a copy of the GNU Lesser General Public
##     License along with qpOASES; if not, write to the Free Software
##     Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
##



##
##     Filename:  CMakeLists.txt
##     Author:    Hans Joachim Ferreau (thanks to Milan Vukov)
##     Version:   3.2
##     Date:      2007-2017
##
## THIS FILE HAS BEEN MODIFIED FROM THE ORIGINAL SOURCE

cmake_minimum_required(VERSION 2.6)

PROJECT(qpOASES CXX)
SET(PACKAGE_NAME "qpOASES")
SET(PACKAGE_VERSION "3.2.0")
SET(PACKAGE_SO_VERSION "3.2")
SET(PACKAGE_DESCRIPTION "An implementation of the online active set strategy")
SET(PACKAGE_AUTHOR "Hans Joachim Ferreau, Andreas Potschka, Christian Kirches et al.")
SET(PACKAGE_MAINTAINER "Hans Joachim Ferreau, Andreas Potschka, Christian Kirches et al.")
SET(PACKAGE_URL "https://projects.coin-or.org/qpOASES")

SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/libs)

IF( NOT CMAKE_VERBOSE_MAKEFILE )
    SET( CMAKE_VERBOSE_MAKEFILE OFF )
ENDIF( NOT CMAKE_VERBOSE_MAKEFILE )

IF( NOT CMAKE_BUILD_TYPE )
    SET(CMAKE_BUILD_TYPE Release CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE
        )
ENDIF( NOT CMAKE_BUILD_TYPE )


option(QPOASES_BUILD_EXAMPLES "Build examples." ON)


############################################################
#################### compiler flags ########################
############################################################
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D__NO_COPYRIGHT__")
IF ( UNIX )
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic -Wfloat-equal -Wshadow -DLINUX")
    SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_DEBUG} -O0 -finline-functions")
ELSEIF( WINDOWS )
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -nologo -EHsc -DWIN32")
ENDIF()

SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -D__DEBUG__")

############################################################
######################## rpath #############################
############################################################
# use, i.e. don't skip the full RPATH for the build tree
set(CMAKE_SKIP_BUILD_RPATH  FALSE)

# when building, don't use the install RPATH already
# (but later on when installing)
set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)

set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib:${CMAKE_INSTALL_PREFIX}/lib/casadi")

# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

# the RPATH to be used when installing, but only if it's not a system directory
list(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/lib" isSystemDir)
if("${isSystemDir}" STREQUAL "-1")
   set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib:${CMAKE_INSTALL_PREFIX}/lib/casadi")
endif("${isSystemDir}" STREQUAL "-1")



############################################################
#################### build and install #####################
############################################################
INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/include)

find_package(LAPACK REQUIRED)
find_package(BLAS REQUIRED)

# compile qpOASES libraries
FILE(GLOB SRC src/*.cpp)
list(REMOVE_ITEM SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/LAPACKReplacement.cpp ${CMAKE_CURRENT_SOURCE_DIR}/src/BLASReplacement.cpp)

# library
ADD_LIBRARY(qpOASES SHARED ${SRC})

#############################
##########EDIT HERE##########
#############################
option(MA57 "Use MA57 sparse solver" OFF)
set(HSL_PATH "" CACHE STRING "Path to the coinhsl folder with the /.libs/libcoinhsl.so binary containing MA57")
set(GK_METIS_PATH "" CACHE STRING "Path to the directory containing the libmetis.a and libGKlib.a binaries that HSL requires")

if(MA57)
	ADD_LIBRARY(METIS STATIC IMPORTED)
	set_target_properties(METIS PROPERTIES IMPORTED_LOCATION ${GK_METIS_PATH}/libmetis.a)

	ADD_LIBRARY(GKLIB STATIC IMPORTED)
	set_target_properties(GKLIB PROPERTIES IMPORTED_LOCATION ${GK_METIS_PATH}/libGKlib.a)

	target_compile_options(qpOASES PUBLIC -DSOLVER_MA57)
	target_link_libraries(qpOASES PUBLIC ${HSL_PATH}/.libs/libcoinhsl.so METIS GKLIB LAPACK::LAPACK BLAS::BLAS dl)
endif()

option(MUMPS "Use MUMPS sparse solver" OFF)
set(MUMPS_PATH "" CACHE STRING "Path to the MUMPS build folder")

if(MUMPS)
	target_include_directories(qpOASES PUBLIC ${MUMPS_PATH}/include ${MUMPS_PATH}/libseq)
	target_compile_definitions(qpOASES PUBLIC SOLVER_MUMPS MUMPS_SEQ)
	target_link_libraries(qpOASES PUBLIC LAPACK::LAPACK BLAS::BLAS ${MUMPS_PATH}/lib/libdmumps.a ${MUMPS_PATH}/lib/libmumps_common.a ${MUMPS_PATH}/libpord.a ${MUMPS_PATH}/libmpiseq.a dl)
endif()


#END OF EDITS

INSTALL(TARGETS qpOASES
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION lib
)
SET_TARGET_PROPERTIES(
    qpOASES
    PROPERTIES
    SOVERSION ${PACKAGE_SO_VERSION}
    )

# headers /usr/lib/gcc/x86_64-linux-gnu/11/
INSTALL(FILES include/qpOASES.hpp
  DESTINATION include)
INSTALL(DIRECTORY include/qpOASES
  DESTINATION include
  FILES_MATCHING PATTERN "*.hpp"
  PATTERN "*.ipp"
  PATTERN ".svn" EXCLUDE)

############################################################
######################### examples #########################
############################################################

if (QPOASES_BUILD_EXAMPLES)
    # compile qpOASES examples
    SET(EXAMPLE_NAMES
        example1
        example1a
        example1b
        example2
        example3
        example3b
        example4
        example5
        exampleLP
        qrecipe
        qrecipeSchur
    )

    FOREACH(ELEMENT ${EXAMPLE_NAMES})
        ADD_EXECUTABLE(${ELEMENT} examples/${ELEMENT}.cpp)
        TARGET_LINK_LIBRARIES(${ELEMENT} qpOASES LAPACK::LAPACK BLAS::BLAS ${HSLPATH}/.libs/libcoinhsl.so dl METIS GKLIB)
    ENDFOREACH(ELEMENT ${EXAMPLE_NAMES})
endif(QPOASES_BUILD_EXAMPLES)

##
##   end of file
##
