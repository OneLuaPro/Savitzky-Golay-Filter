# Define the source and include directories
set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(INCLUDE_DIR ${CMAKE_SOURCE_DIR}/include/iterative)

# List the source files
set(SOURCES
    ${SRC_DIR}/savgolFilter.c
)

# List the header files (optional, but good practice for IDEs)
set(HEADERS
    ${INCLUDE_DIR}/savgolFilter.h
)

# Create a static library
add_library(savgolFilter STATIC ${SOURCES} ${HEADERS})

# Enable AVX
if (CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(savgolFilter PRIVATE -mavx -mfma)
elseif (CMAKE_C_COMPILER_ID MATCHES "MSVC")
    target_compile_options(savgolFilter PRIVATE /arch:AVX)
    target_link_options(savgolFilter PRIVATE /NODEFAULTLIB:libcmt.lib)
endif()

# Set include directories for the library
target_include_directories(savgolFilter PUBLIC ${INCLUDE_DIR})

# Link the math library if needed
if (NOT CMAKE_C_COMPILER_ID MATCHES "MSVC")
    target_link_libraries(savgolFilter PRIVATE m)
endif()

# Ensure Cygwin runtime symbols are resolved
if (CYGWIN)
    target_link_libraries(savgolFilter PUBLIC c cygwin)
endif()

# Install
install(TARGETS savgolFilter
  LIBRARY DESTINATION ${INSTALL_LIBDIR} COMPONENT lib
)
install(
  FILES ${HEADERS} DESTINATION ${INSTALL_INCLUDEDIR}
)
