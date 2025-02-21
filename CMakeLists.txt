cmake_minimum_required(VERSION 3.31 FATAL_ERROR)
set(CMAKE_EXPERIMENTAL_CXX_IMPORT_STD "0e5b6991-d74f-4b3d-a41c-cf096e0b2508")
set(CMAKE_CXX_STANDARD 26)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_SCAN_FOR_MODULES ON)

project(main LANGUAGES CXX)
set(CMAKE_CXX_MODULE_STD 1)

file(GLOB_RECURSE ${PROJECT_NAME}_sources CONFIGURE_DEPENDS
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.S
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cxx
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)

file(GLOB_RECURSE ${PROJECT_NAME}_modules CONFIGURE_DEPENDS
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cppm)

add_executable(${PROJECT_NAME} ${${PROJECT_NAME}_sources} ${${PROJECT_NAME}_modules})
target_include_directories(${PROJECT_NAME} PRIVATE inc include)

target_sources(${PROJECT_NAME} PRIVATE FILE_SET module_files TYPE CXX_MODULES BASE_DIRS
    ${CMAKE_CURRENT_SOURCE_DIR}/src FILES ${${PROJECT_NAME}_modules})

if (CMAKE_CXX_COMPILER_FRONTEND_VARIANT STREQUAL "MSVC")
    target_compile_options(${PROJECT_NAME} PRIVATE /W4 /WX
        $<$<CONFIG:Debug>:/DEBUG /Zi /O0>
        $<$<CONFIG:Release>:/O2>)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -pedantic -Werror
        $<$<CONFIG:Debug>:-g -O0>
        $<$<CONFIG:Release>:-O2>)

    if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_23 INTERFACE cxx_std_23)
        target_compile_options(${PROJECT_NAME} PRIVATE -stdlib=libc++)
        target_link_options(${PROJECT_NAME} PRIVATE -stdlib=libc++)
    endif()

    add_custom_command(
        TARGET "${PROJECT_NAME}" POST_BUILD
        COMMAND "$<$<CONFIG:Release>:${CMAKE_STRIP}>"
        ARGS "${PROJECT_NAME}")
endif()
