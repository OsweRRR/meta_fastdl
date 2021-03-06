project(meta_fastdl)
cmake_minimum_required(VERSION 2.8)

set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
enable_language(Rust)
include(CMakeCargo)

set(THIRD_PARTY "${CMAKE_SOURCE_DIR}/third_party")
set(SRC_DIR "${CMAKE_SOURCE_DIR}/cpp")
set(METAMOD "${THIRD_PARTY}/metamod-hl1/metamod")
set(HLSDK "${THIRD_PARTY}/hlsdk")

set(API_INCLUDE_PATH "${HLSDK}/common ${HLSDK}/dlls ${HLSDK}/engine ${HLSDK}/pm_shared ${HLSDK}/game_shared ${HLSDK}/public ${METAMOD}")

string(REPLACE " " ";" API_INCLUDE_PATH_LIST ${API_INCLUDE_PATH})

include_directories(${API_INCLUDE_PATH_LIST})
include_directories(${SRC_DIR}/helper)

add_subdirectory(rust)

file(GLOB_RECURSE SOURCE_FILES ${SRC_DIR}/*.h ${SRC_DIR}/*.cpp)
#${API_INCLUDE_PATH}/*.h ${API_INCLUDE_PATH}/*.cpp

add_library(${PROJECT_NAME} SHARED ${SOURCE_FILES} )

set(FLAGS "-m32 -O3 -mtune=generic -flto -fPIC -fvisibility=hidden")
set(PORTABLE_LINK_FLAGS "${FLAGS} -static-libgcc -static-libstdc++ -s  -Wl,--gc-sections -Wl,--version-script=${CMAKE_SOURCE_DIR}/version_script.lds")


set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_FLAGS "${FLAGS} -m32 -fdata-sections -ffunction-sections -fpermissive -fstack-protector-strong")
set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")

if(WIN32)
# TODO
#	target_link_libraries(${PROJECT_NAME} ${PROJECT_NAME}_rust Ws2_32 gcc Userenv Mswsock user32 Psapi Ncrypt Crypt32 Secur32 pthread)
#	set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "${PORTABLE_LINK_FLAGS} -Wl,-L@CMakeFiles/../rust/i686-pc-windows-gnu/debug")
#	set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX "_i386.dll")
else()
	target_link_libraries(${PROJECT_NAME} ${PROJECT_NAME}-rust pthread)
	set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "${PORTABLE_LINK_FLAGS} -m32 -fuse-ld=lld")
	set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX "_i386.so")
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wzero-as-null-pointer-constant -Wunknown-pragmas")

if (CMAKE_BUILD_TYPE STREQUAL "Debug")
	 add_definitions(-D_DEBUG)
endif()
