cmake_minimum_required(VERSION 3.20)
project(LuaJITWrapper VERSION 2.1.0)

set(LUAJIT_PREFIX "/usr/local" CACHE PATH "LuaJIT installation prefix")
set(LUAJIT_DESTDIR "" CACHE PATH "DESTDIR")
find_program(GNU_MAKE_EXECUTABLE NAMES gmake make REQUIRED)

set(LUAJIT_LIB "${CMAKE_CURRENT_SOURCE_DIR}/src/libluajit.a")
set(LUAJIT_BIN "${CMAKE_CURRENT_SOURCE_DIR}/src/luajit")
set(LUAJIT_INCLUDE "${CMAKE_CURRENT_SOURCE_DIR}/src")

add_custom_command(
    OUTPUT "${LUAJIT_LIB}" "${LUAJIT_BIN}"
    COMMAND "${GNU_MAKE_EXECUTABLE}" -C "${CMAKE_CURRENT_SOURCE_DIR}" BUILDMODE=static
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/Makefile"
    COMMENT "Building LuaJIT static library..."
)

add_custom_target(luajit-build ALL DEPENDS "${LUAJIT_LIB}")

add_library(LuaJIT::LuaJIT STATIC IMPORTED GLOBAL)
set_target_properties(LuaJIT::LuaJIT PROPERTIES
    IMPORTED_LOCATION "${LUAJIT_LIB}"
    INTERFACE_INCLUDE_DIRECTORIES "${LUAJIT_INCLUDE}"
)
add_library(LuaJIT ALIAS LuaJIT::LuaJIT)
