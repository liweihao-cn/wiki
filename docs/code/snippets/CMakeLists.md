# CMakeLists

## 模板

```cmake
# CMake template file
# Use this file with
# mkdir build
# cd build
# cmake .. && make ..
# make -C build

cmake_minimum_required(VERSION 3.15.3)

# Optional: print out extra messages to see what is going on. Comment it to have less verbose messages
set(CMAKE_VERBOSE_MAKEFILE ON)

# Path to toolchain file. This one has to be before 'project()' below
SET(CROSS_COMPILE "/path-to-toolchain/bin/arm-none-eabi-")
SET(CMAKE_C_COMPILER ${CROSS_COMPILE}gcc)
SET(CMAKE_CXX_COMPILER ${CROSS_COMPILE}g++)

# Setup project, output and linker file
project(make_template)
set(EXECUTABLE ${PROJECT_NAME}.elf)
set(LINKER_FILE ${CMAKE_SOURCE_DIR}/link.ld)

enable_language(C ASM)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# List of source files
set(SRC_FILES
    main.c
    )

# Build the executable based on the source files
add_executable(${EXECUTABLE} ${SRC_FILES})

# List of compiler defines, prefix with -D compiler option
target_compile_definitions(${EXECUTABLE} PRIVATE
    -DDEBUG
    )

# List of includ directories
target_include_directories(${EXECUTABLE} PRIVATE
    . 
    driver
    )

# Compiler options
target_compile_options(${EXECUTABLE} PRIVATE
    -fdata-sections
    -ffunction-sections

    -Wall
    -O2
    )

# Linker options
target_link_options(${EXECUTABLE} PRIVATE
    -T${LINKER_FILE}
    -lm
    -Wl,-Map=${PROJECT_NAME}.map,--cref
    -Wl,--gc-sections
    -Xlinker -print-memory-usage -Xlinker
    )

# Linker libraries
#target_link_libraries(${EXECUTABLE} PRIVATE
#    m
#    )

# Optional: Print executable size as part of the post build process
add_custom_command(TARGET ${EXECUTABLE}
    POST_BUILD
    COMMAND ${CMAKE_SIZE_UTIL} ${EXECUTABLE_OUTPUT_PATH}/${EXECUTABLE})

# Optional: Create hex, bin and S-Record files after the build
add_custom_command(TARGET ${EXECUTABLE}
    POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O srec --srec-len=64 "${EXECUTABLE_OUTPUT_PATH}/${EXECUTABLE}" "${EXECUTABLE_OUTPUT_PATH}/${PROJECT_NAME}.s19"
    COMMAND ${CMAKE_OBJCOPY} -O ihex "${EXECUTABLE_OUTPUT_PATH}/${EXECUTABLE}" "${EXECUTABLE_OUTPUT_PATH}/${PROJECT_NAME}.hex"
    COMMAND ${CMAKE_OBJCOPY} -O binary "${EXECUTABLE_OUTPUT_PATH}/${EXECUTABLE}" "${EXECUTABLE_OUTPUT_PATH}/${PROJECT_NAME}.bin" 
    )

# Optional: flash device
add_custom_target(flash
    COMMAND openocd/pyocd commands
    DEPENDS ${EXECUTABLE}
    )

```
