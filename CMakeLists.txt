cmake_minimum_required(VERSION 3.28)

project(project)
enable_language(ASM)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

file(GLOB_RECURSE ${PROJECT_NAME}_sources CONFIGURE_DEPENDS
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.S
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cxx
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)

file(GLOB_RECURSE ${PROJECT_NAME}_modules CONFIGURE_DEPENDS
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cppm)

add_executable(${PROJECT_NAME}
    ${${PROJECT_NAME}_sources} ${${PROJECT_NAME}_modules})

target_include_directories(${PROJECT_NAME} PRIVATE inc include)

target_sources(${PROJECT_NAME} PRIVATE FILE_SET module_files TYPE CXX_MODULES BASE_DIRS
    ${CMAKE_CURRENT_SOURCE_DIR}/src FILES ${${PROJECT_NAME}_modules})

if (MSVC)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /O0 /DEBUG")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_DEBUG} /O2")
    target_compile_options(${PROJECT_NAME} PRIVATE /W4 /WX)
else()
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g -O0")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2")
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -pedantic -Werror)
endif()
