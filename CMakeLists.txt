# Minimum CMake version required
cmake_minimum_required(VERSION 3.10)

# Project name and languages
project(SavitzkyGolayFilter LANGUAGES C CXX)

# Set C and C++ standards
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# =============================================================================
# BUILD OPTIONS
# =============================================================================
include (GNUInstallDirs)
set(INSTALL_BINDIR
  ${CMAKE_INSTALL_BINDIR} CACHE PATH "Installation directory for binaries")
set(INSTALL_LIBDIR
  ${CMAKE_INSTALL_LIBDIR} CACHE PATH "Installation directory for libraries")
set(INSTALL_INCLUDEDIR
  ${CMAKE_INSTALL_INCLUDEDIR} CACHE PATH "Installation directory for header files")
option(USE_PARALLEL_SAVGOL "Build parallel (OpenMP) version of Savitzky-Golay filter" OFF)

# Print selected build mode
if(USE_PARALLEL_SAVGOL)
    message(STATUS "Building PARALLEL Savitzky-Golay implementation (with OpenMP)")
else()
    message(STATUS "Building SEQUENTIAL Savitzky-Golay implementation")
endif()

# Ensure Cygwin runtime linkage for all targets
if (CYGWIN)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -lc -lcygwin")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -lc -lcygwin")
    set(CMAKE_STATIC_LINKER_FLAGS "${CMAKE_STATIC_LINKER_FLAGS} -lc -lcygwin")
endif()

# Fetch GoogleTest
include(FetchContent)
# Forces GoogleTest to link against the Shared C Runtime (MSVC /MD), ensuring
# consistency with the project's runtime settings and preventing linker conflicts.
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG release-1.12.1
)
FetchContent_MakeAvailable(googletest)

# Suppress specific IntelLLVM compiler warnings for GoogleTest targets to prevent
# build failures when "Warnings as Errors" is enabled, specifically addressing
# char16_t to char32_t conversions.
if (CMAKE_CXX_COMPILER_ID STREQUAL "IntelLLVM")
  foreach(_target gtest gtest_main gmock gmock_main)
    if(TARGET ${_target})
      target_compile_options(${_target} PRIVATE "-W0" "-Wno-error=character-conversion")
    endif()
  endforeach()
endif()

# Fetch FFF (Fake Function Framework)
FetchContent_Declare(
    fff
    GIT_REPOSITORY https://github.com/meekrosoft/fff.git
    GIT_TAG master
)
FetchContent_MakeAvailable(fff)

# =============================================================================
# INCLUDE SUBDIRECTORIES
# =============================================================================
# Conditionally include the appropriate implementation
if(USE_PARALLEL_SAVGOL)
    #add_subdirectory(src/parallel)
else()
    add_subdirectory(src)
endif()

# Always include tests (they should work with either implementation)
add_subdirectory(test/iterative)

# Enable testing
enable_testing()
