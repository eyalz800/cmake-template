cmake_minimum_required(VERSION 3.28)

project(main)
enable_language(ASM)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_SCAN_FOR_MODULES ON)

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

if (CMAKE_CXX_COMPILER_FRONTEND_VARIANT STREQUAL "MSVC")
    target_compile_options(${PROJECT_NAME} PRIVATE /W4 /WX
        $<$<CONFIG:Debug>:/O0>
        $<$<CONFIG:Release>:/O2>)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -pedantic -Werror
        $<$<CONFIG:Debug>:-O0>
        $<$<CONFIG:Release>:-O2>)

    if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        if (NOT LIBCXX_BUILD)
            target_compile_options(${PROJECT_NAME} PRIVATE -stdlib=libc++)
            target_link_options(${PROJECT_NAME} PRIVATE -stdlib=libc++)
        else()
            include(FetchContent)
            FetchContent_Declare(
                std
                URL "file://${LIBCXX_BUILD}/modules/c++/v1"
                DOWNLOAD_EXTRACT_TIMESTAMP TRUE
                SYSTEM
                )
            FetchContent_MakeAvailable(std)
            target_link_libraries(std.compat std)

            target_compile_options(${PROJECT_NAME} PRIVATE $<$<COMPILE_LANGUAGE:CXX>:-nostdinc++>)
            target_compile_options(${PROJECT_NAME} PRIVATE $<$<COMPILE_LANGUAGE:CXX>:-isystem>)
            target_compile_options(${PROJECT_NAME} PRIVATE $<$<COMPILE_LANGUAGE:CXX>:${LIBCXX_BUILD}/include/c++/v1>)

            target_link_options(${PROJECT_NAME} PRIVATE $<$<COMPILE_LANGUAGE:CXX>:-nostdlib++>)
            target_link_options(${PROJECT_NAME} PRIVATE $<$<COMPILE_LANGUAGE:CXX>:-L${LIBCXX_BUILD}/lib>)
            target_link_options(${PROJECT_NAME} PRIVATE $<$<COMPILE_LANGUAGE:CXX>:-Wl,-rpath,${LIBCXX_BUILD}/lib>)

            target_link_options(${PROJECT_NAME} PRIVATE $<$<COMPILE_LANGUAGE:CXX>:-Wl,-no_warn_duplicate_libraries>)

            target_link_libraries(${PROJECT_NAME} PRIVATE std c++)
            target_link_libraries(${PROJECT_NAME} PRIVATE std.compat c++)
        endif()
    endif()

    add_custom_command(
        TARGET "${PROJECT_NAME}" POST_BUILD
        DEPENDS "${PROJECT_NAME}"
        COMMAND "$<$<CONFIG:Release>:${CMAKE_STRIP}>"
        ARGS "${PROJECT_NAME}")
endif()
