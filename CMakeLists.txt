set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_VERSION 1)
set(CMAKE_TRY_COMPILE_TARGET_TYPE "STATIC_LIBRARY")
cmake_minimum_required(VERSION 3.20)

# 工具链设置
set(TOOLPATH  D:/0wch/toolchain/RISCVEmbeddedGCC/bin/riscv-none-embed-)

if (WIN32)
	MESSAGE(STATUS "Now is windows!")
    set(CMAKE_C_COMPILER ${TOOLPATH}gcc.exe)
    set(CMAKE_CXX_COMPILER ${TOOLPATH}g++.exe)
    set(CMAKE_ASM_COMPILER ${TOOLPATH}gcc.exe)
    set(CMAKE_AR ${TOOLPATH}ar.exe)
    set(CMAKE_OBJCOPY ${TOOLPATH}objcopy.exe)
    set(CMAKE_OBJDUMP ${TOOLPATH}objdump.exe)
    set(SIZE ${TOOLPATH}size.exe)
elseif (UNIX)
	MESSAGE(STATUS "Now is UNIX-like OS!")
    set(CMAKE_C_COMPILER ${TOOLPATH}gcc)
    set(CMAKE_CXX_COMPILER ${TOOLPATH}g++)
    set(CMAKE_ASM_COMPILER ${TOOLPATH}gcc)
    set(CMAKE_AR ${TOOLPATH}ar)
    set(CMAKE_OBJCOPY ${TOOLPATH}objcopy)
    set(CMAKE_OBJDUMP ${TOOLPATH}objdump)
    set(SIZE ${TOOLPATH}size)
else ()
    MESSAGE(STATUS "Unsupported system!")
endif ()

# 项目设置
project(ch32v307-ninja C CXX ASM)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_C_STANDARD 99)

# 编译参数 一般不用改
add_compile_options(-march=rv32imacxw -mabi=ilp32 -mcmodel=medany -msmall-data-limit=8 -mno-save-restore)
add_compile_options(-fmessage-length=0 -fsigned-char -ffunction-sections -fdata-sections -fno-common)

# 编译等级
add_compile_options(-O3)

# 编译信息等级
add_compile_options(-Wall)

# 头文件路径
include_directories(APP
                    Core
                    Debug 
                    Peripheral/inc)

# 宏定义
# add_definitions(-DDEBUG=1)

# 源码文件
file(GLOB_RECURSE SOURCES 
                "APP/*.c"
                "Core/core_riscv.c" 
                "Debug/debug.c"
                "Peripheral/src/*.c" 
                "Startup/startup_ch32v30x_D8C.S"
                )

# 链接参数
set(LINKER_SCRIPT  ${CMAKE_SOURCE_DIR}/Ld/Link.ld)
add_link_options(-nostartfiles 
                -Xlinker --gc-sections  
                -Wl,--print-memory-usage
                -Wl,-Map,${PROJECT_NAME}.map 
                --specs=nano.specs 
                --specs=nosys.specs)
add_link_options(-T ${LINKER_SCRIPT})

# 编译可执行文件
add_executable(${PROJECT_NAME}.elf ${SOURCES} ${LINKER_SCRIPT})

# 链接静态库
target_link_libraries(${PROJECT_NAME}.elf printfloat)

# 输出hex和bin
set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)
set(LST_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.lst)
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE}
        COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE}
        COMMAND ${CMAKE_OBJDUMP} --all-headers --demangle --disassemble $<TARGET_FILE:${PROJECT_NAME}.elf> > ${LST_FILE}
        COMMAND ${SIZE} --format=berkeley $<TARGET_FILE:${PROJECT_NAME}.elf>
)
