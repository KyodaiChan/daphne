# CMakeLists.txt

# Set the minimum required version of CMake
cmake_minimum_required(VERSION 3.10)

# Project name
project(KM_Daphne_Xtreme)

# Default platform variable if not set
if(NOT DEFINED PLATFORM)
    execute_process(COMMAND uname -a OUTPUT_VARIABLE UNAME_RESULT OUTPUT_STRIP_TRAILING_WHITESPACE)
    if(NOT UNAME_RESULT STREQUAL "")
        if(UNAME_RESULT MATCHES "Darwin")
            set(PLATFORM "osx")
        elseif(UNAME_RESULT MATCHES "MINGW")
            set(PLATFORM "win")
        elseif(UNAME_RESULT MATCHES "win")
            set(PLATFORM "win")
        else()
            set(PLATFORM "unix")
        endif()
    else()
        set(PLATFORM "win")
    endif()
endif()

# Set up platform-specific variables
if(PLATFORM STREQUAL "unix")
    set(TARGET ${PROJECT_NAME}_libretro.so)
    set(FPIC -fPIC)
    set(SHARED -shared -Wl,--version-script=link.T -Wl,--no-undefined)
    set(LIBS pthread dl)
elseif(PLATFORM STREQUAL "osx")
    set(TARGET ${PROJECT_NAME}_libretro.dylib)
    set(FPIC -fPIC)
    set(SHARED -dynamiclib)
    add_definitions(-DOSX)
    set(ARCH "intel")
    execute_process(COMMAND uname -p OUTPUT_VARIABLE UNAME_P OUTPUT_STRIP_TRAILING_WHITESPACE)
    if(UNAME_P STREQUAL "powerpc")
        set(ARCH "ppc")
    endif()
    set(OSXVER "10.1")
    set(FPIC "${FPIC} -mmacosx-version-min=10.1")
elseif(PLATFORM STREQUAL "pi")
    set(TARGET ${PROJECT_NAME}_libretro.so)
    set(FPIC -fPIC)
    set(SHARED -shared -Wl,--version-script=link.T -Wl,--no-undefined)
    include_directories("/opt/vc/include/interface/vcos/pthreads" "/opt/vc/include/vmcs_host/linux")
    set(LIBS "/opt/vc/lib" pthread dl bcm_host vchostif)
elseif(PLATFORM STREQUAL "ios")
    set(TARGET ${PROJECT_NAME}_libretro_ios.dylib)
    set(FPIC -fPIC)
    set(SHARED -dynamiclib)
    add_definitions(-DIOS)
    set(IOSSDK "")
    if(NOT IOSSDK)
        execute_process(COMMAND xcodebuild -version -sdk iphoneos Path OUTPUT_VARIABLE IOSSDK OUTPUT_STRIP_TRAILING_WHITESPACE)
    endif()
    set(CC "cc -arch armv7 -isysroot ${IOSSDK}")
    set(CXX "c++ -arch armv7 -isysroot ${IOSSDK}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -miphoneos-version-min=8.0")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -miphoneos-version-min=8.0")
elseif(PLATFORM STREQUAL "qnx")
    set(TARGET ${PROJECT_NAME}_libretro_qnx.so)
    set(FPIC -fPIC)
    set(SHARED -shared -Wl,--version-script=link.T)
    set(CC qcc -Vgcc_ntoarmv7le)
elseif(PLATFORM STREQUAL "emscripten")
    set(TARGET ${PROJECT_NAME}_libretro_emscripten.bc)
else()
    set(TARGET ${PROJECT_NAME}_libretro.dll)
    set(SHARED -shared -static-libgcc -static-libstdc++ -s -Wl,--version-script=link.T -lwinmm -Wl,--no-undefined)
    set(LIBS winmm ws2_32)
endif()

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
if(PLATFORM STREQUAL "armv")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -I. -DARM")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -I. -DARM")
    if(PLATFORM MATCHES "cortexa8")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -marm -mcpu=cortex-a8")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -marm -mcpu=cortex-a8")
    elseif(PLATFORM MATCHES "cortexa9")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -marm -mcpu=cortex-a9")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -marm -mcpu=cortex-a9")
    endif()
    if(PLATFORM MATCHES "neon")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfpu=neon")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mfpu=neon")
    endif()
    if(PLATFORM MATCHES "softfloat")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfloat-abi=softfp")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mfloat-abi=softfp")
    elseif(PLATFORM MATCHES "hardfloat")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfloat-abi=hard")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mfloat-abi=hard")
    endif()
endif()

# Compiler and linker options
add_definitions(-D__LIBRETRO__)

# Create the target
add_library(${TARGET} SHARED ${OBJECTS})

# Link libraries
target_link_libraries(${TARGET} ${LIBS})

# Clean up
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY CLEAN_DIRECTORIES ${OBJECTS} ${TARGET})
