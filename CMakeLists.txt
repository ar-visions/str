# CMake minimum version
cmake_minimum_required(VERSION 3.10)

# Project name
project(str_library VERSION 1.0 LANGUAGES C)

# Options
option(BUILD_SHARED_LIBS "Build shared library" OFF)
option(NO_ASAN "Disable AddressSanitizer (ASAN)" OFF)

# Set warning and extra flags
set(CC_WARN "-Wall -Wextra -Werror=implicit-function-declaration -Wformat -Werror=format-security")

if(CMAKE_C_COMPILER MATCHES "musl-gcc")
    # Define for musl-gcc builds
    set(CC_EXTRA "-D__STDC_ISO_10646__=201706L")
    set(CC_SAN "")  # No sanitizers for musl builds
else()
    # Sanitizers for non-musl builds
    if(NO_ASAN)
        set(CC_SAN "")
    else()
        set(CC_SAN "-fsanitize=address -fsanitize=leak -fsanitize=undefined -fsanitize-address-use-after-scope")
    endif()
endif()

# Common compile flags
set(COMMON_CFLAGS "-std=c11 -pipe ${CC_WARN} ${CC_EXTRA} ${CC_SAN}")

# Library source files
set(SRC_FILES str.c)

# Add the library
add_library(str ${SRC_FILES})

# Include directories
target_include_directories(str PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

# Set compile flags for the library
target_compile_options(str PRIVATE ${COMMON_CFLAGS})

# Installation
install(TARGETS str
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin)
install(FILES str.h DESTINATION include)

# Tools
add_executable(gen-char-class tools/gen_char_class.c)
target_compile_options(gen-char-class PRIVATE ${COMMON_CFLAGS})
install(TARGETS gen-char-class RUNTIME DESTINATION bin)

# Test executables
add_executable(test_str str_test.c)
target_compile_options(test_str PRIVATE -ggdb -fno-omit-frame-pointer ${COMMON_CFLAGS})
target_link_libraries(test_str str)

add_executable(flto_test_str str_test.c)
target_compile_options(flto_test_str PRIVATE -s -O2 -flto -march=native -mtune=native ${COMMON_CFLAGS})
target_link_libraries(flto_test_str str)

# Add test targets
add_custom_target(test
    COMMAND ./test_str
    DEPENDS test_str)

add_custom_target(flto-test
    COMMAND ./flto_test_str
    DEPENDS flto_test_str)

