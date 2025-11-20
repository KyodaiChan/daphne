# CMakeLists.txt

# Set the minimum required version of CMake
cmake_minimum_required(VERSION 3.10)

# Project name
project(KM_Daphne_Xtreme)

# Default platform variable if not set

set(PLATFORM "win")


# Set up platform-specific variables

set(TARGET ${PROJECT_NAME}_libretro.dll)
set(SHARED -shared -static-libgcc -static-libstdc++ -s -Wl,--version-script=link.T -lwinmm -Wl,--no-undefined)
set(LIBS winmm ws2_32)


# Add debugging flags if DEBUG is set
if(DEFINED DEBUG AND DEBUG EQUAL 1)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0 -g")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0 -g")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -g")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -DNDEBUG")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2 -DNDEBUG")
endif()

# Set common CXX and C flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${FPIC}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${FPIC}")

# Include directories and libraries
include_directories(${CMAKE_SOURCE_DIR})

# Set up the object files
file(GLOB SOURCES_C "${CMAKE_SOURCE_DIR}/*.c")
file(GLOB SOURCES_CXX "${CMAKE_SOURCE_DIR}/*.cpp")
set(OBJECTS ${SOURCES_C} ${SOURCES_CXX})

# Platform-specific ARM flags


# Compiler and linker options
add_definitions(-D__LIBRETRO__)

# Create the target
add_library(${TARGET} SHARED ${OBJECTS})

# Link libraries
target_link_libraries(${TARGET} ${LIBS})

# Clean up
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY CLEAN_DIRECTORIES ${OBJECTS} ${TARGET})
