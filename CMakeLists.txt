cmake_minimum_required(VERSION 4.0.0)

set(CMAKE_EXPERIMENTAL_CXX_IMPORT_STD
    "451f2fe2-a8a2-47c3-bc32-94786d8fc91b")

set(CMAKE_CXX_MODULE_STD ON)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

project(SFW VERSION 1.0)

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(ARCH_DIR "x64")
else()
    set(ARCH_DIR "x86")
endif()

if(WIN32)
    # 
elseif(LINUX)
    # Only one backend can be used at a time on Linux, so default to XCB if none are set
    set(SFW_USE_XCB     ON  CACHE BOOL "Use XCB backend on Linux")
    set(SFW_USE_WAYLAND OFF CACHE BOOL "Use Wayland backend on Linux")
    set(SFW_USE_X11     OFF CACHE BOOL "Use X11 backend on Linux")
elseif(APPLE)
    #
else()
    message(FATAL_ERROR "Unsupported platform")
endif()

# Normalize common config names to "Capitalized" form
set(CONFIG_DIR "$<IF:$<CONFIG:Debug>,Debug,$<IF:$<CONFIG:Release>,Release,$<IF:$<CONFIG:RelWithDebInfo>,Relwithdebinfo,$<IF:$<CONFIG:MinSizeRel>,Minsizerel,$<CONFIG>>>>>")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${ARCH_DIR}/${CONFIG_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${ARCH_DIR}/${CONFIG_DIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${ARCH_DIR}/${CONFIG_DIR})

foreach(OUTPUTCONFIG Debug Release RelWithDebInfo MinSizeRel)
    foreach(OUTPUTTYPE RUNTIME LIBRARY ARCHIVE)
        set(CMAKE_${OUTPUTTYPE}_OUTPUT_DIRECTORY_${OUTPUTCONFIG}
            ${CMAKE_BINARY_DIR}/${ARCH_DIR}/${OUTPUTCONFIG})
    endforeach()
endforeach()

if(WIN32)
    add_compile_definitions(SFW_WINDOWS)
elseif(LINUX)
    add_compile_definitions(SFW_LINUX)
elseif(APPLE)
    add_compile_definitions(SFW_MACOS)
endif()

add_compile_definitions($<$<CONFIG:Debug>:DEBUG>
                        $<$<CONFIG:Release>:RELEASE>)

# SFW
add_subdirectory(SFW)

# Example
add_subdirectory(Example)