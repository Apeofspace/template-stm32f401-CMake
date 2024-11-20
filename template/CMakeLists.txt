cmake_minimum_required(VERSION 3.7...3.28)
if(${CMAKE_VERSION} VERSION_LESS 3.12)
    cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
endif()

#-----PROJECT NAME-----
# this hack puts dir name as project name
cmake_path(GET CMAKE_CURRENT_SOURCE_DIR FILENAME ProjectName)   
string(REPLACE " " "_" ProjectId ${ProjectName})
project(${ProjectName})

#-----MCU-----
set(MCPU cortex-m4)
set(MCU_MODEL DSTM32F401xC)

#----COMPILER SETTINGS----
enable_language(C CXX ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_VERSION 1)
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)
set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
set(CMAKE_AR arm-none-eabi-ar)
set(CMAKE_OBJCOPY arm-none-eabi-objcopy)
set(CMAKE_OBJDUMP arm-none-eabi-objdump)
set(SIZE arm-none-eabi-size)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

#----SOURCES, HEADERS, SCRIPTS----
file(GLOB_RECURSE C_SRC CONFIGURE_DEPENDS "Core/Src/*.c" )

file(GLOB_RECURSE C_LIBS CONFIGURE_DEPENDS
    ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/*.c
    ${CMAKE_CURRENT_SOURCE_DIR}/Middlewares/*.c
    ${CMAKE_CURRENT_SOURCE_DIR}/USB_DEVICE/*.c
    )

MACRO(HEADER_DIRECTORIES return_list)
    FILE(GLOB_RECURSE new_list CONFIGURE_DEPENDS *.h )
    SET(dir_list "")
    FOREACH(file_path ${new_list})
        GET_FILENAME_COMPONENT(dir_path ${file_path} PATH)
        SET(dir_list ${dir_list} ${dir_path})
    ENDFOREACH()
    LIST(REMOVE_DUPLICATES dir_list)
    SET(${return_list} ${dir_list})
ENDMACRO()

HEADER_DIRECTORIES(INCLUDE_DIRS)

set(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/STM32F401CCUx_FLASH.ld)
set(STARTUP_SCRIPT ${CMAKE_SOURCE_DIR}/startup_stm32f401xc.s)

#-----COMPILE OPTIONS-----
add_definitions(-DUSE_HAL_DRIVER -${MCU_MODEL})

add_compile_options(
    -mcpu=${MCPU}
    -mthumb
    -ffunction-sections
    -fdata-sections
    -fno-common
    -fmessage-length=0
    -Wno-unused-parameter
)

add_link_options(
    -mcpu=${MCPU}
    -mthumb
    -Wl,-Map=${PROJECT_BINARY_DIR}/${PROJECT_NAME}.map,--cref
    -T ${LINKER_SCRIPT}
    -lm -lnosys 
    --specs=nano.specs 
    -Wl,-gc-sections,--print-memory-usage 
    # -gc-sections requires -ffunction-sections and -fdata-sections
)

# Uncomment for hardware floating point
add_compile_definitions(ARM_MATH_CM4;ARM_MATH_MATRIX_CHECK;ARM_MATH_ROUNDING)
add_compile_options(-mfloat-abi=hard -mfpu=fpv4-sp-d16)
add_link_options(-mfloat-abi=hard -mfpu=fpv4-sp-d16)

# Uncomment for software floating point
# add_compile_options(-mfloat-abi=soft)

# Uncomment to mitigate c++17 absolute addresses warnings
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} 
# -Wno-register 
# -Wconversion # STM libraries!
# -Wno-volatile
# -Wold-style-cast
# -Wuseless-cast
# -Wsuggest-override")

#-----BUILD TYPE-----
if ("${CMAKE_BUILD_TYPE}" STREQUAL "Release")
    message(STATUS "High optimization for speed")
    add_compile_options(-O3)
    # add_compile_options(-Ofast)
elseif ("${CMAKE_BUILD_TYPE}" STREQUAL "RelWithDebInfo")
    message(STATUS "High optimization for speed, debug info included")
    add_compile_options(-O3 -g)
    # add_compile_options(-Ofast -g)
elseif ("${CMAKE_BUILD_TYPE}" STREQUAL "MinSizeRel")
    message(STATUS "Maximum optimization for size")
    add_compile_options(-Os)
else ()
    message(STATUS "Minimal optimization, debug info included")
    add_compile_options(-Og -g)
endif ()

#-----BUILD COMMANDS-----
set(OUTPUT_NAME output)
# set(OUTPUT_NAME ${PROJECT_NAME})
add_executable(${OUTPUT_NAME}.elf ${C_SRC} ${C_LIBS} ${STARTUP_SCRIPT} ${LINKER_SCRIPT})
set(HEX_FILE ${PROJECT_BINARY_DIR}/${OUTPUT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${OUTPUT_NAME}.bin)

target_include_directories(${OUTPUT_NAME}.elf PRIVATE ${INCLUDE_DIRS})
target_link_libraries(${OUTPUT_NAME}.elf m) # link math.h 
# target_link_libraries(${OUTPUT_NAME}.elf stdc++) # link c++ stdl

add_custom_command(TARGET ${OUTPUT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${OUTPUT_NAME}.elf> ${HEX_FILE}
    COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${OUTPUT_NAME}.elf> ${BIN_FILE}
    COMMENT "Building ${HEX_FILE}
    Building ${BIN_FILE}")
