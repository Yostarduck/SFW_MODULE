cmake_minimum_required(VERSION 4.0.0)

set(APP_VERSION_MAJOR 1)
set(APP_VERSION_MINOR 0)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/SFWConfig.h.in
               ${CMAKE_CURRENT_BINARY_DIR}/include/SFWConfig.h)

option(SFW_BUILD_SHARED "Build SFW as a shared library (ON) or static library (OFF)" OFF)

set(SFW_LIBRARY_TYPE STATIC)
if(SFW_BUILD_SHARED)
    set(SFW_LIBRARY_TYPE SHARED)
endif()

file(GLOB_RECURSE SFW_SOURCES CONFIGURE_DEPENDS
     ${CMAKE_CURRENT_SOURCE_DIR}/source/*.cpp
     ${CMAKE_CURRENT_SOURCE_DIR}/source/*.cc
     ${CMAKE_CURRENT_SOURCE_DIR}/source/*.cxx)

if(NOT SFW_SOURCES)
    set(SFW_DUMMY_SOURCE ${CMAKE_CURRENT_BINARY_DIR}/SFW_dummy.cpp)
    file(WRITE ${SFW_DUMMY_SOURCE} "namespace SFW { void __sfw_dummy_translation_unit() {} }\n")
    list(APPEND SFW_SOURCES ${SFW_DUMMY_SOURCE})
endif()

add_library(SFW ${SFW_LIBRARY_TYPE}
            ${SFW_SOURCES})

target_include_directories(SFW PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include
                                       ${CMAKE_CURRENT_BINARY_DIR}/include)

if(WIN32)
    #
elseif(LINUX)
    # Only one backend can be used at a time on Linux
    # Default to XCB if none are set
    if (NOT SFW_USE_XCB AND NOT SFW_USE_WAYLAND AND NOT SFW_USE_X11)
      set(SFW_USE_XCB ON)
    endif()

    if(SFW_USE_XCB)
        find_package(PkgConfig REQUIRED)
        pkg_check_modules(XCB REQUIRED xcb)

        target_compile_definitions(SFW PRIVATE SFW_USE_XCB)
        target_include_directories(SFW PRIVATE ${XCB_INCLUDE_DIRS})
        target_link_libraries(SFW PRIVATE ${XCB_LIBRARIES})
    elseif(SFW_USE_WAYLAND)
        find_package(PkgConfig REQUIRED)
        pkg_check_modules(WAYLAND_CLIENT REQUIRED wayland-client)

        target_compile_definitions(SFW PRIVATE SFW_USE_WAYLAND)
        target_include_directories(SFW PRIVATE ${WAYLAND_CLIENT_INCLUDE_DIRS})
        target_link_libraries(SFW PRIVATE ${WAYLAND_CLIENT_LIBRARIES})
    elseif(SFW_USE_X11)
        find_package(X11 REQUIRED)

        target_compile_definitions(SFW PRIVATE SFW_USE_X11)
        target_include_directories(SFW PRIVATE ${X11_INCLUDE_DIR})
        target_link_libraries(SFW PRIVATE ${X11_LIBRARIES})
    else()
        message(FATAL_ERROR "On Linux, set one backend: -DSFW_USE_WAYLAND=ON, -DSFW_USE_XCB=ON, or -DSFW_USE_X11=ON")
    endif()
elseif(APPLE)
    #
endif()

install(TARGETS SFW
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/include/SFWConfig.h DESTINATION include)