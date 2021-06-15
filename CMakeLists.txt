cmake_minimum_required(VERSION 3.18)

project(JDH8)

set(CMAKE_C_STANDARD 11)
add_definitions(-O0 -g -Wall -Wextra -Wpedantic -Wstrict-aliasing -Wno-pointer-arith -Wno-unused-parameter -Wno-gnu-zero-variadic-macro-arguments)

# https://discourse.cmake.org/t/support-for-embedding-data-in-a-manner-equivalent-to-xxd/2121/8
function(xxdi inFile outFile)
    # Read file.
    file(READ "${inFile}" text)

    # Convert to hex.
    string(HEX "${text}" hex)
    string(LENGTH "${text}" textLen)
    string(LENGTH ${hex} hexLen)
    math(EXPR hexLen "${hexLen} - 1")
    set(result "")
    foreach(idx RANGE 0 ${hexLen} 2)
        string(SUBSTRING ${hex} ${idx} 2 char)
        list(APPEND result "0x${char}")
    endforeach()
    list(JOIN result ", " result)

    # Variable name.
    string(REPLACE "/" "_" var "${inFile}")
    string(REPLACE "." "_" var "${var}")

    # Output C format.
    file(WRITE "${outFile}" "unsigned char ${var}[] = {${result}};\nunsigned int ${var}_len = ${textLen};\n")
endfunction()

xxdi(asm/macros.asm asm/builtin_macros.c)

find_package(SDL2 REQUIRED)

file(GLOB_RECURSE ASM asm/*.c)
file(GLOB_RECURSE EMU emu/*.c)
file(GLOB_RECURSE TEST test/*.c)

add_executable(asm ${ASM})
add_executable(emu ${EMU})

target_include_directories(emu
    PUBLIC ${SDL2_INCLUDE_DIRS}
)

target_link_libraries(emu
    PUBLIC ${SDL2_LIBRARIES}
    PUBLIC readline
)
