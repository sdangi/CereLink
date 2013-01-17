# CBSDK CMake Build System
# Author: griffin.milsap@gmail.com
# Version 1.0 Dec. 2012
#	Build information for
#		cbsdk Library
#		testcbsdk Test Binary

CMAKE_MINIMUM_REQUIRED( VERSION 2.8 )

PROJECT( CBSDK )

SET( CBSDK_VERSION_MAJOR 1 )
SET( CBSDK_VERSION_MINOR 0 )

# Handle 64 bit binaries
IF( CMAKE_SIZEOF_VOID_P EQUAL 4 )
	SET( LIB_SUFFIX "" )
ELSE( CMAKE_SIZEOF_VOID_P EQUAL 4 )
	SET( LIB_SUFFIX 64 )
ENDIF( CMAKE_SIZEOF_VOID_P EQUAL 4 )

SET( CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/../dist/lib${LIB_SUFFIX} )
SET( CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/../dist/lib${LIB_SUFFIX} )
SET( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/../dist/bin )

SET( LIB_NAME cbsdk )

FIND_PACKAGE( Qt4 REQUIRED )

INCLUDE_DIRECTORIES(
	../cbhwlib
	../Central
	../Central Common
)

SET( LIB_SOURCE
	../cbmex/cbsdk.cpp
	../cbhwlib/cbhwlib.cpp
	../cbhwlib/cbHwlibHi.cpp
	../cbhwlib/CCFUtils.cpp
	../cbhwlib/CCFUtilsBinary.cpp
	../cbhwlib/CCFUtilsConcurrent.cpp
	../cbhwlib/CCFUtilsXml.cpp
	../cbhwlib/CCFUtilsXmlItems.cpp
	../cbhwlib/InstNetwork.cpp
	../cbhwlib/XmlFile.cpp
	../Central/Instrument.cpp
	../Central/UDPsocket.cpp
)

# Only headers which need MOC'ing
SET( LIB_HEADERS
	../cbhwlib/InstNetwork.h
)

# Handle Qt
QT4_WRAP_CPP( LIB_HEADERS_MOC ${LIB_HEADERS} )
SET( QT_USE_QTXML TRUE )
INCLUDE( ${QT_USE_FILE} )
ADD_DEFINITIONS( ${QT_DEFINITIONS} -DQT_APP )

ADD_DEFINITIONS( -DCBSDK_EXPORTS )

# Perform an MFC compilation if running on Windows.
IF( WIN32 )
	ADD_DEFINITIONS( -DWIN32 -D_DEBUG -D_WINDOWS -DWINVER=0x0501 -DNO_AFX )
	SET( CMAKE_MFC_FLAG 1 )
ENDIF( WIN32 )

ADD_LIBRARY( ${LIB_NAME} SHARED ${LIB_SOURCE} ${LIB_HEADERS_MOC} )

TARGET_LINK_LIBRARIES( ${LIB_NAME} ${QT_LIBRARIES} )

IF( WIN32 )
	TARGET_LINK_LIBRARIES( ${LIB_NAME} ws2_32 winmm )
ENDIF( WIN32 )

ADD_CUSTOM_COMMAND(
	TARGET "${LIB_NAME}"
	POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy "${PROJECT_BINARY_DIR}/../cbmex/cbsdk.h" "${PROJECT_BINARY_DIR}/../dist/include/cbsdk.h"
	COMMAND ${CMAKE_COMMAND} -E copy "${PROJECT_BINARY_DIR}/../cbhwlib/cbhwlib.h" "${PROJECT_BINARY_DIR}/../dist/include/cbhwlib.h"
)

# Test executable
SET( TEST_NAME testcbsdk )
ADD_EXECUTABLE( ${TEST_NAME} ../cbmex/testcbsdk.cpp )
TARGET_LINK_LIBRARIES( ${TEST_NAME} ${LIB_NAME} )

# Install information
INSTALL( TARGETS ${TEST_NAME} ${LIB_NAME}
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib${LIB_SUFFIX}
	ARCHIVE DESTINATION lib${LIB_SUFFIX}
)
INSTALL( FILES ../cbmex/cbsdk.h ../cbhwlib/cbhwlib.h DESTINATION include )
