cmake_minimum_required(VERSION 3.15)

PROJECT(sigar VERSION 1.6.2 LANGUAGES C CXX)

INCLUDE(CheckIncludeFiles)
INCLUDE(CheckFunctionExists)
INCLUDE(CheckTypeSize)
INCLUDE(CTest)
INCLUDE(InstallRequiredSystemLibraries)

ENABLE_TESTING()

SET(CPACK_PACKAGE_VERSION_MAJOR "1")
SET(CPACK_PACKAGE_VERSION_MINOR "6")
SET(CPACK_PACKAGE_VERSION_PATCH "2")
SET(CPACK_PACKAGE_VENDOR "Hyperic")
## SET(CPACK_*) before the INCLUDE(CPack)
INCLUDE(CPack)

include (GNUInstallDirs)
include (CMakePackageConfigHelpers)

set (SIGAR_INSTALL_NAME ${PROJECT_NAME})

IF(WIN32) 
	IF(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
		add_compile_options (
			/wd5103#关闭宏展开警告
			/wd4133#关闭类型转换警告
			/wd4293#关闭左移警告
			/wd4090#关闭const警告
			/wd4311#关闭指针截断警告

		)
	ENDIF(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")

	## make sure we only use the smallest set of 
	## headers on win32. Otherwise we get clashes 
	## between winsock2.h and winsock.h
	ADD_DEFINITIONS(-DWIN32_LEAN_AND_MEAN)

	# force the correct version for the redist manifest
	ADD_DEFINITIONS(-D_BIND_TO_CURRENT_MFC_VERSION=1 -D_BIND_TO_CURRENT_CRT_VERSION=1)
	# turn off security warnings for system calls
	ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS)
ENDIF(WIN32)

ADD_SUBDIRECTORY(src build-src)
ADD_SUBDIRECTORY(tests build-tests)
ADD_SUBDIRECTORY(include build-include)

set (SIGAR_CMAKE_CONFIG_INSTALL_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/${SIGAR_INSTALL_NAME})

if (MSVC)
	install (FILES $<TARGET_PDB_FILE:sigar> DESTINATION "${CMAKE_INSTALL_BINDIR}" OPTIONAL)
endif (MSVC)

write_basic_package_version_file (
	"${CMAKE_CURRENT_BINARY_DIR}/${SIGAR_INSTALL_NAME}-config-version.cmake"
	VERSION ${PROJECT_VERSION}
	COMPATIBILITY AnyNewerVersion
)

configure_package_config_file (
	"${CMAKE_CURRENT_LIST_DIR}/${SIGAR_INSTALL_NAME}-config.cmake.in"
	"${CMAKE_CURRENT_BINARY_DIR}/${SIGAR_INSTALL_NAME}-config.cmake"
	INSTALL_DESTINATION ${SIGAR_CMAKE_CONFIG_INSTALL_DIR}
	NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

install (FILES
	"${CMAKE_CURRENT_BINARY_DIR}/${SIGAR_INSTALL_NAME}-config.cmake"
	"${CMAKE_CURRENT_BINARY_DIR}/${SIGAR_INSTALL_NAME}-config-version.cmake"
	DESTINATION ${SIGAR_CMAKE_CONFIG_INSTALL_DIR}
)

install (EXPORT ${SIGAR_INSTALL_NAME}Targets
	FILE "${SIGAR_INSTALL_NAME}-targets.cmake"
	NAMESPACE ${SIGAR_INSTALL_NAME}::
	DESTINATION ${SIGAR_CMAKE_CONFIG_INSTALL_DIR}
)