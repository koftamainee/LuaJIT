cmake_minimum_required(VERSION 3.20)

project(LuaJITWrapper VERSION 2.1.0 LANGUAGES NONE)

include(GNUInstallDirs)

set(LUAJIT_PREFIX "/usr/local" CACHE PATH "LuaJIT installation prefix passed to make install")
set(LUAJIT_MULTILIB "lib" CACHE STRING "LuaJIT multilib directory passed to make install")
set(LUAJIT_DESTDIR "" CACHE PATH "DESTDIR passed to make install/uninstall")

set(LUAJIT_ENABLE_AMALGAMATION OFF CACHE BOOL "Add amalgamation build target")
set(LUAJIT_ENABLE_INSTALL_TARGETS ON CACHE BOOL "Enable install/uninstall targets")

find_program(GNU_MAKE_EXECUTABLE
    NAMES gmake make
    REQUIRED
    DOC "GNU Make executable used to build LuaJIT"
)

if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/Makefile")
    message(FATAL_ERROR "LuaJIT Makefile not found in: ${CMAKE_CURRENT_SOURCE_DIR}")
endif()

set(LUAJIT_LIB "${CMAKE_CURRENT_SOURCE_DIR}/src/libluajit.so")
set(LUAJIT_BIN "${CMAKE_CURRENT_SOURCE_DIR}/src/luajit")
set(LUAJIT_INCLUDE "${CMAKE_CURRENT_SOURCE_DIR}/src")

add_custom_command(
    OUTPUT "${LUAJIT_LIB}" "${LUAJIT_BIN}"
    COMMAND "${GNU_MAKE_EXECUTABLE}" -C "${CMAKE_CURRENT_SOURCE_DIR}"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    DEPENDS
        "${CMAKE_CURRENT_SOURCE_DIR}/Makefile"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/Makefile"
    COMMENT "Building LuaJIT..."
    VERBATIM
)

add_custom_target(luajit-build ALL
    DEPENDS "${LUAJIT_LIB}" "${LUAJIT_BIN}"
)

if(LUAJIT_ENABLE_AMALGAMATION)
    add_custom_target(luajit-amalg
        COMMAND "${GNU_MAKE_EXECUTABLE}" -C "${CMAKE_CURRENT_SOURCE_DIR}" amalg
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        VERBATIM
    )
endif()

function(luajit_make_install target_name make_goal)
    set(cmd "${GNU_MAKE_EXECUTABLE}" -C "${CMAKE_CURRENT_SOURCE_DIR}" "${make_goal}"
        PREFIX="${LUAJIT_PREFIX}"
        MULTILIB="${LUAJIT_MULTILIB}"
    )

    if(NOT "${LUAJIT_DESTDIR}" STREQUAL "")
        list(APPEND cmd DESTDIR="${LUAJIT_DESTDIR}")
    endif()

    add_custom_target(${target_name}
        COMMAND ${cmd}
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        VERBATIM
    )
endfunction()

if(LUAJIT_ENABLE_INSTALL_TARGETS)
    luajit_make_install(luajit-install install)
    luajit_make_install(luajit-uninstall uninstall)
endif()

add_library(LuaJIT::LuaJIT SHARED IMPORTED GLOBAL)

set_target_properties(LuaJIT::LuaJIT PROPERTIES
    IMPORTED_LOCATION "${LUAJIT_LIB}"
    INTERFACE_INCLUDE_DIRECTORIES "${LUAJIT_INCLUDE}"
)

add_library(LuaJIT ALIAS LuaJIT::LuaJIT)

install(CODE [[
    message(STATUS "LuaJIT is built via Makefile; use luajit-install target for system install.")
]])
