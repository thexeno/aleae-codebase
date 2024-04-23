# CMake template file

cmake_minimum_required(VERSION 3.15.3)

# Optional: print out extra messages to see what is going on. Comment it to have less verbose messages
set(CMAKE_VERBOSE_MAKEFILE ON)

# Path to toolchain file. This one has to be before 'project()' below
message(STATUS "CMAKE_CURR_SOURCE_DIR: ${CMAKE_CURR_SOURCE_DIR}")
message(STATUS "CMAKE_SOURCE_DIR: ${CMAKE_SOURCE_DIR}")

set(CMAKE_TOOLCHAIN_FILE "${CMAKE_SOURCE_DIR}/arm-none-eabi-gcc.cmake")

# Setup project, output and linker file
# Check if CMAKE_BUILD_TYPE is not set or is empty
if(NOT CMAKE_BUILD_TYPE OR CMAKE_BUILD_TYPE STREQUAL "")
    # Set a default build type if none was specified
    set(CMAKE_BUILD_TYPE "Debug" CACHE STRING "Default build type: Debug" FORCE)
endif()

# Now, you can use CMAKE_BUILD_TYPE in your CMakeLists.txt
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
set(BUILD_FOLDER "${CMAKE_SOURCE_DIR}/../build/${CMAKE_BUILD_TYPE}")

project(AleaExample)

set(EXECUTABLE "${PROJECT_NAME}.elf")
set(EXECUTABLE_OUTPUT_PATH "${CMAKE_SOURCE_DIR}/../build/${CMAKE_BUILD_TYPE}")
set(LINKER_FILE "${CMAKE_SOURCE_DIR}/../build/STM32F103C8TX_FLASH.ld")

enable_language(C ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)



include_directories("${CMAKE_SOURCE_DIR}/startup")
add_subdirectory("${CMAKE_SOURCE_DIR}/startup")


#List of main source file
set(SRC_FILES   
"main.c"
)

file(GLOB AleaExample ${SRC_FILES})
add_executable(${EXECUTABLE} ${AleaExample})

message(STATUS " °°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°°° Executable is: ${EXECUTABLE}")

unset(CMAKE_ASM_FLAGS CACHE)
set(CMAKE_ASM_FLAGS "-x assembler-with-cpp -mcpu=cortex-m3 -mthumb  -c" CACHE STRING "" FORCE)

# Compiler options
set(MY_COMPILE_FLAGS 
        -fdiagnostics-color 
        -mcpu=cortex-m3 
        -std=gnu11 
        -O0 
        -ffunction-sections 
        -fdata-sections 
        -Wall 
        -fstack-usage 
        -mfloat-abi=soft 
        -mthumb 
        -lc 
        -nodefaultlibs 
        -nostdlib
)

target_link_libraries(${EXECUTABLE} hstartup)
target_compile_options(hstartup PRIVATE ${MY_COMPILE_FLAGS})

message(STATUS " #################################################  Alea Example is: ${AleaExample}")

target_compile_options(${EXECUTABLE} PRIVATE ${MY_COMPILE_FLAGS})
message(STATUS " #################################################  MY_COMPILE_FLAGS is: ${MY_COMPILE_FLAGS}")

# Linker options
target_link_options(${EXECUTABLE} PRIVATE
        -T${LINKER_FILE}
        -fdiagnostics-color
        --specs=nosys.specs
        -nodefaultlibs
        -Wl,-Map=${EXECUTABLE_OUTPUT_PATH}/${PROJECT_NAME}.map
        -nodefaultlibs
        -nostdlib
        -Wl,--cref -Wl,--gc-sections -Wl,--verbose -static --specs=nano.specs -mfloat-abi=soft -mthumb -Wl,--start-group -lc -lm -Wl,--end-group
        )
        
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