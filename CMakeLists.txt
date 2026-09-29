#
# Copyright (c) 2016 Alibek Omarov
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
#

cmake_minimum_required(VERSION 3.10)
project(tf15-client)

set(CMAKE_CXX_STANDARD 11)

# Install custom module path
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
set(EXTRAS_DIR "${CMAKE_CURRENT_BINARY_DIR}/extras")

# Force XP toolchain for Visual Studio
include(VSForceXPToolchain)

# for extras packaging
find_package(Python COMPONENTS Interpreter REQUIRED)

if (Python_FOUND)
	message(STATUS "Python interpreter found: ${Python_EXECUTABLE}")
else()
	message(FATAL_ERROR "Python not found. Please install Python.")
endif()

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

#--------------
# USER DEFINES \
################\
option(USE_VOICEMGR "Enable VOICE MANAGER." OFF)

option(BUILD_CLIENT "Build client dll" ON)
option(BUILD_SERVER "Build server dll" OFF)
option(BUILD_MENU "Build menu dll" ON)
option(BUILD_PMAN "Build particleman" OFF)

option(GOLDSOURCE_SUPPORT "Build goldsource compatible client library" OFF)
option(GOLDSOURCE_DEFAULT_FLAGS "Build with default flags from Valve's Makefile" OFF)

set(GAME_DIR "tfc" CACHE STRING "Gamedir path")
set(SERVER_INSTALL_DIR "dlls" CACHE STRING "Where put server dll")
set(CLIENT_INSTALL_DIR "cl_dlls" CACHE STRING "Where put client dll")
set(SERVER_LIBRARY_NAME "tfc" CACHE STRING "Library name for Linux/MacOS/Windows")

#-----------------
# MAIN BUILD CODE \
###################\

# Xash3D FWGS Library Naming Scheme compliance
# see documentation: https://github.com/FWGS/xash3d-fwgs/blob/master/Documentation/library-naming.md
include(LibraryNaming)

# engine is obligated to provide VGUI interface in 32-bit builds on Windows/Linux as shared library
# on platforms supported only by Xash3D FWGS, FreeVGUI is linked statically into client library
string(TOLOWER "${CMAKE_SYSTEM_PROCESSOR}" SYSTEM_PROCESSOR_LOWER)
if((WIN32 OR CMAKE_SYSTEM_NAME STREQUAL "Linux")
	AND CMAKE_SIZEOF_VOID_P EQUAL 4
	AND SYSTEM_PROCESSOR_LOWER MATCHES "^(x86|i[3-6]86|x86_64|amd64|em64t)$")
	set(USE_STATIC_FREEVGUI OFF)
else()
	set(USE_STATIC_FREEVGUI ON)
endif()

if(MINGW)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static-libstdc++ -static-libgcc")
	set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--add-stdcall-alias")
endif()

# add_compile_options for older cmake versions
if(${CMAKE_VERSION} VERSION_LESS "3.0.2")
	macro(add_compile_options)
		set(list_var "${ARGV}")
		foreach(arg IN LISTS list_var)
			set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${arg}")
		endforeach()
	endmacro()
endif()

if(NOT MSVC)
	#add_compile_options(-Wempty-body) # GCC/Clang flag
	add_compile_options(-Wreturn-type) # GCC/Clang flag
else()
	add_definitions(-D_CRT_SILENCE_NONCONFORMING_TGMATH_H -D_CRT_SECURE_NO_WARNINGS)
endif()

if(BUILD_CLIENT)
	add_subdirectory(cl_dll)
endif()

if(BUILD_SERVER)
	add_subdirectory(dlls)
endif()

if(BUILD_MENU)
	add_subdirectory(3rdparty/mainui_cpp)
endif()

if(BUILD_CLIENT)
	# FreeVGUI isn't installed: on 32-bit x86 Windows and Linux the engine ships vgui library, the client only links against it
	add_subdirectory(3rdparty/freevgui)
	if(USE_STATIC_FREEVGUI)
		target_include_directories(vgui PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/engine")
	endif()
endif()

if(BUILD_PMAN)
	add_definitions(-DUSE_PARTICLEMAN)
	add_subdirectory(3rdparty/particleman)
endif()

if(NOT ANDROID)
	set(EXTRAS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/tf15client-extras")
	file(GLOB_RECURSE EXTRAS_FILES "${EXTRAS_DIR}/*")
	
	set(EXTRAS_OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/extras.pk3")
	
	add_custom_command(
		OUTPUT ${EXTRAS_OUTPUT}
		COMMAND ${Python_EXECUTABLE}
				${CMAKE_CURRENT_SOURCE_DIR}/scripts/pack_extras.py
				${EXTRAS_DIR}
				${EXTRAS_OUTPUT}
		DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/scripts/pack_extras.py
				${EXTRAS_FILES}
		COMMENT "Generating extras.pk3..."
	)
	
	add_custom_target(generate_extras ALL
		DEPENDS ${EXTRAS_OUTPUT}
	)
	
	install(
		FILES "${EXTRAS_OUTPUT}"
		DESTINATION ${GAME_DIR}
	)
endif()

if(WIN32)
	set(CPACK_GENERATOR "ZIP")
	set(CPACK_PACKAGE_FILE_NAME "TF15Client-${CMAKE_SYSTEM_NAME}-${CMAKE_CXX_COMPILER_ARCHITECTURE_ID}")
else()
	set(CPACK_GENERATOR "TGZ")
	set(CPACK_PACKAGE_FILE_NAME "TF15Client-${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")
endif()

set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY OFF)

include(CPack)
