cmake_minimum_required(VERSION 3.2)

project(obs-transition-matrix)

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
	set(ARCH_NAME "64bit")
	set(OBS_BUILDDIR_ARCH "build64")
	set(_lib_suffix 64)
else()
	set(ARCH_NAME "32bit")
	set(OBS_BUILDDIR_ARCH "build32")
	set(_lib_suffix 32)
endif()

##########################################
# find libobs                            #
##########################################
include(external/FindLibObs.cmake)
find_package(LibObs REQUIRED)

# Find includes in corresponding build directories
if(DEFINED QTDIR${_lib_suffix})
	list(APPEND CMAKE_PREFIX_PATH "${QTDIR${_lib_suffix}}")
elseif(DEFINED QTDIR)
	list(APPEND CMAKE_PREFIX_PATH "${QTDIR}")
elseif(DEFINED ENV{QTDIR${_lib_suffix}})
	list(APPEND CMAKE_PREFIX_PATH "$ENV{QTDIR${_lib_suffix}}")
elseif(DEFINED ENV{QTDIR})
	list(APPEND CMAKE_PREFIX_PATH "$ENV{QTDIR}")
endif()

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)

##########################################
# QT support                             #
##########################################
find_package(Qt5Core REQUIRED)
find_package(Qt5Widgets REQUIRED)

set(obs-transition-matrix_HEADERS
	obs-transition-matrix.hpp
	obs-transition-matrix-dialog.hpp
	obs-transition-matrix-model.hpp
	)

set(obs-transition-matrix_SOURCES
	obs-transition-matrix.cpp
	obs-transition-matrix-dialog.cpp
	obs-transition-matrix-model.cpp
	)

include_directories(
	${LIBOBS_INCLUDE_DIR}
	"${LIBOBS_INCLUDE_DIR}/../UI/obs-frontend-api"
	${Qt5Core_INCLUDES}
	${Qt5Widgets_INCLUDES}
)

add_library(obs-transition-matrix MODULE
	${obs-transition-matrix_HEADERS}
	${obs-transition-matrix_SOURCES}
	)

target_link_libraries(obs-transition-matrix
	${OBS_FRONTEND_LIB}
	${LIBOBS_LIB}
	Qt5::Core
	Qt5::Widgets
	)

# --- Windows-specific build settings and tasks ---
if(WIN32)
	# --- Release package helper ---
	# The "release" folder has a structure similar OBS' one on Windows
	set(RELEASE_DIR "${PROJECT_SOURCE_DIR}/release")

	add_custom_command(TARGET obs-transition-matrix POST_BUILD
		# If config is Release, package release files
		COMMAND if $<CONFIG:Release>==1 (
		"${CMAKE_COMMAND}" -E make_directory
		"${RELEASE_DIR}/data/obs-plugins/obs-transition-matrix"
		"${RELEASE_DIR}/obs-plugins/${ARCH_NAME}")

		# If config is RelWithDebInfo, package release files
		COMMAND if $<CONFIG:RelWithDebInfo>==1 (
		"${CMAKE_COMMAND}" -E make_directory
		"${RELEASE_DIR}/data/obs-plugins/obs-transition-matrix"
		"${RELEASE_DIR}/obs-plugins/${ARCH_NAME}")

		COMMAND if $<CONFIG:Release>==1 ("${CMAKE_COMMAND}" -E copy_directory
		"${PROJECT_SOURCE_DIR}/data"
		"${RELEASE_DIR}/data/obs-plugins/obs-transition-matrix")

		COMMAND if $<CONFIG:RelWithDebInfo>==1 ("${CMAKE_COMMAND}" -E copy_directory
		"${PROJECT_SOURCE_DIR}/data"
		"${RELEASE_DIR}/data/obs-plugins/obs-transition-matrix")

		COMMAND if $<CONFIG:RelWithDebInfo>==1 ("${CMAKE_COMMAND}" -E copy
		"$<TARGET_PDB_FILE:obs-transition-matrix>"
		"${RELEASE_DIR}/obs-plugins/${ARCH_NAME}")

		COMMAND (
		"${CMAKE_COMMAND}" -E copy
		"$<TARGET_FILE:obs-transition-matrix>"
		"${RELEASE_DIR}/obs-plugins/${ARCH_NAME}")

		# Copy to obs-studio environment for immediate testing
		COMMAND (
		"${CMAKE_COMMAND}" -E copy
		"$<TARGET_FILE:obs-transition-matrix>"
		"${LIBOBS_INCLUDE_DIR}/../${OBS_BUILDDIR_ARCH}/rundir/$<CONFIG>/obs-plugins/${ARCH_NAME}")

		COMMAND if $<CONFIG:Debug>==1 (
		"${CMAKE_COMMAND}" -E copy
		"$<TARGET_PDB_FILE:obs-transition-matrix>"
		"${LIBOBS_INCLUDE_DIR}/../${OBS_BUILDDIR_ARCH}/rundir/$<CONFIG>/obs-plugins/${ARCH_NAME}")

		COMMAND (
		"${CMAKE_COMMAND}" -E make_directory
		"${LIBOBS_INCLUDE_DIR}/../${OBS_BUILDDIR_ARCH}/rundir/$<CONFIG>/data/obs-plugins/obs-transition-matrix")

		COMMAND (
		"${CMAKE_COMMAND}" -E copy_directory
		"${PROJECT_SOURCE_DIR}/data"
		"${LIBOBS_INCLUDE_DIR}/../${OBS_BUILDDIR_ARCH}/rundir/$<CONFIG>/data/obs-plugins/obs-transition-matrix")
		)
	# --- End of sub-section ---
endif()
# --- End of section ---

# --- Linux-specific build settings and tasks ---
if(UNIX AND NOT APPLE)
	set(CMAKE_CXX_FLAGS "-Wall -Wextra -Wno-unused-function -Werror-implicit-function-declaration -Wno-missing-field-initializers ${CMAKE_CXX_FLAGS} -std=c++11 -fno-strict-aliasing")
	set(CMAKE_C_FLAGS "-Wall -Wextra -Wno-unused-function -Werror-implicit-function-declaration -Wno-missing-braces -Wno-missing-field-initializers ${CMAKE_C_FLAGS} -std=gnu99 -fno-strict-aliasing")

	set_target_properties(obs-transition-matrix PROPERTIES PREFIX "")

	file(GLOB locale_files data/locale/*.ini)

	if(CUSTOM)
		install(TARGETS obs-transition-matrix LIBRARY DESTINATION
			"${CMAKE_INSTALL_PREFIX}/rundir/${CMAKE_BUILD_TYPE}/obs-plugins/${ARCH_NAME}")
		install(FILES ${locale_files} DESTINATION
			"${CMAKE_INSTALL_PREFIX}/rundir/${CMAKE_BUILD_TYPE}/data/obs-plugins/obs-transition-matrix/locale")
	else()
		install(TARGETS obs-transition-matrix LIBRARY DESTINATION
			"${CMAKE_INSTALL_PREFIX}/lib/obs-plugins")
		install(FILES ${locale_files} DESTINATION
			"${CMAKE_INSTALL_PREFIX}/share/obs/obs-plugins/obs-transition-matrix/locale")
	endif()
endif()
# --- End of section ---

# -- OS X specific build settings and tasks --
# -- Help with OS X build is need as i don't --
# -- have any hardware to test. --
if(APPLE)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++ -fvisibility=default")

	set_target_properties(obs-transition-matrix PROPERTIES PREFIX "")
endif()
# -- End of section --
