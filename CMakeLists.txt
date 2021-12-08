cmake_minimum_required(VERSION 3.17)
project(idfexe)

execute_process(COMMAND git describe --always --abbrev=8
                OUTPUT_VARIABLE VERSION
                ERROR_QUIET)
string(STRIP "${VERSION}" VERSION)
if(NOT VERSION)
    message(FATAL_ERROR "Failed to determine version from git")    
endif()

set(ARCHIVE_NAME idf-exe-${VERSION}.zip)

add_executable(idf idf_main.c)
target_compile_definitions(idf PRIVATE -DVERSION=\"${VERSION}\")
set_target_properties(idf PROPERTIES C_STANDARD 99)
target_link_libraries(idf "shlwapi")

set_property(TARGET idf PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

if(CMAKE_BUILD_TYPE STREQUAL Release AND CMAKE_STRIP)
    add_custom_command(TARGET idf
                        POST_BUILD
                        COMMAND ${CMAKE_STRIP} idf.exe)
endif()

add_custom_target(dist ALL DEPENDS idf)

add_custom_command(
    TARGET dist
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} ARGS -E copy "${CMAKE_CURRENT_BINARY_DIR}/idf.exe" "${CMAKE_CURRENT_BINARY_DIR}/idf.py.exe"
    COMMAND ${CMAKE_COMMAND} ARGS -E tar cfv ${ARCHIVE_NAME} --format=zip
       "${CMAKE_CURRENT_BINARY_DIR}/idf.py.exe"
    )
