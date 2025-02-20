cmake_minimum_required(VERSION 3.0)
#set(CMAKE_VERBOSE_MAKEFILE ON)

# setup toolchain
set(CMAKE_SYSTEM_NAME "Generic")
set(DEVKITPRO $ENV{DEVKITPRO})
set(CMAKE_SYSTEM_PROCESSOR "armv8-a")
set(CMAKE_C_COMPILER "${DEVKITPRO}/devkitA64/bin/aarch64-none-elf-gcc")
set(CMAKE_CXX_COMPILER "${DEVKITPRO}/devkitA64/bin/aarch64-none-elf-g++")
set(CMAKE_ASM_COMPILER "${DEVKITPRO}/devkitA64/bin/aarch64-none-elf-as")
set(CMAKE_AR "${DEVKITPRO}/devkitA64/bin/aarch64-none-elf-gcc-ar" CACHE STRING "")
set(CMAKE_RANLIB "${DEVKITPRO}/devkitA64/bin/aarch64-none-elf-gcc-ranlib" CACHE STRING "")
set(C_FLAGS "-march=armv8-a -mtune=cortex-a57 -mtp=soft -fPIE -ffast-math -ffunction-sections -fdata-sections")
set(CMAKE_C_FLAGS "${C_FLAGS} -D__SWITCH__ -I${DEVKITPRO}/libnx/include -I${DEVKITPRO}/portlibs/switch/include" CACHE STRING "C flags")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -fpermissive -fno-rtti -fno-exceptions -std=gnu++11" CACHE STRING "C++ flags")
set(CMAKE_FIND_ROOT_PATH ${DEVKITPRO} ${DEVKITPRO}/devkitA64 ${DEVKITPRO}/libnx ${DEVKITPRO}/portlibs/switch)
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
set(BUILD_SHARED_LIBS OFF CACHE INTERNAL "Shared libs not available")

project(nx-cmake)

# flags
set(FLAGS -Wall -D_REENTRANT)

# portlibs (libraries ported to switch in devkitpro)
set(PORTLIBS_PREFIX "${DEVKITPRO}/portlibs/switch")
set(PORTLIBS_LIBDIR "${PORTLIBS_PREFIX}/lib")

# sdl2 specific
set(SDL2_INCLUDE_DIRS "${PORTLIBS_PREFIX}/include/SDL2")

# libnx
set(LIBNX_PREFIX "${DEVKITPRO}/libnx")
set(LIBNX_LIBDIR "${DEVKITPRO}/libnx/lib")

set(CMAKE_FIND_ROOT_PATH ${DEVKITPRO} ${DEVKITPRO}/devkitA64 ${DEVKITPRO}/libnx ${PORTLIBS_PREFIX})

add_executable(${CMAKE_PROJECT_NAME}.elf source/main.c)
target_include_directories(${CMAKE_PROJECT_NAME}.elf PRIVATE source ${SDL2_INCLUDE_DIRS})
target_link_libraries(${CMAKE_PROJECT_NAME}.elf
		-L${PORTLIBS_LIBDIR} -L${LIBNX_LIBDIR} SDL2 bz2 drm_nouveau EGL freetype glad z stdc++ drm_nouveaud glapi GLESv1_CM
		GLESv2 minizip png SDL2main nx m)

#####################
# NX (SWITCH) target
#####################
set_target_properties(${CMAKE_PROJECT_NAME}.elf PROPERTIES LINK_FLAGS "-specs=${DEVKITPRO}/libnx/switch.specs")
add_custom_target(${CMAKE_PROJECT_NAME}.nro
        DEPENDS ${CMAKE_PROJECT_NAME}.elf
        COMMAND elf2nro ${CMAKE_PROJECT_NAME}.elf ${CMAKE_PROJECT_NAME}.nro)
