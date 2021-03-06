cmake_minimum_required(VERSION 2.8)

project(heartbeats-simple)
set(PROJECT_VERSION 0.1.0)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-unknown-pragmas")
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)

include_directories(${PROJECT_SOURCE_DIR}/inc)

# Libraries

set(HBS_SRC src/hb.c src/hb-util.c)
set(HBS_ACC_SRC src/hb.c src/hb-util.c src/hb-acc-util.c)
set(HBS_ACC_PROP HEARTBEAT_MODE_ACC;HEARTBEAT_USE_ACC)
set(HBS_POW_SRC src/hb.c src/hb-util.c src/hb-pow-util.c)
set(HBS_POW_PROP HEARTBEAT_MODE_POW;HEARTBEAT_USE_POW)
set(HBS_ACC_POW_SRC src/hb.c src/hb-util.c src/hb-acc-util.c src/hb-pow-util.c)
set(HBS_ACC_POW_PROP HEARTBEAT_MODE_ACC_POW;HEARTBEAT_USE_ACC;HEARTBEAT_USE_POW)

add_library(hbs SHARED ${HBS_SRC})
add_library(hbs-static STATIC ${HBS_SRC})

add_library(hbs-acc SHARED ${HBS_ACC_SRC})
set_target_properties(hbs-acc PROPERTIES COMPILE_DEFINITIONS "${HBS_ACC_PROP}")
add_library(hbs-acc-static STATIC ${HBS_ACC_SRC})
set_target_properties(hbs-acc-static PROPERTIES COMPILE_DEFINITIONS "${HBS_ACC_PROP}")

add_library(hbs-pow SHARED ${HBS_POW_SRC})
set_target_properties(hbs-pow PROPERTIES COMPILE_DEFINITIONS "${HBS_POW_PROP}")
add_library(hbs-pow-static STATIC ${HBS_POW_SRC})
set_target_properties(hbs-pow-static PROPERTIES COMPILE_DEFINITIONS "${HBS_POW_PROP}")

add_library(hbs-acc-pow SHARED ${HBS_ACC_POW_SRC})
set_target_properties(hbs-acc-pow PROPERTIES COMPILE_DEFINITIONS "${HBS_ACC_POW_PROP}")
add_library(hbs-acc-pow-static STATIC ${HBS_ACC_POW_SRC})
set_target_properties(hbs-acc-pow-static PROPERTIES COMPILE_DEFINITIONS "${HBS_ACC_POW_PROP}")

# Required to force fPIC on static libs
# 'set (CMAKE_POSITION_INDEPENDENT_CODE TRUE)' not supported until cmake 2.8.9
IF(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
  SET_TARGET_PROPERTIES(hbs-static hbs-acc-static hbs-pow-static hbs-acc-pow-static PROPERTIES COMPILE_FLAGS "-fPIC")
ENDIF(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")

# Examples

add_executable(hb-pow-example example/hb-pow-example.c)
target_link_libraries(hb-pow-example hbs-pow)
find_library(LIBRT NAMES rt)
if(LIBRT)
  # no librt on Mac ("Darwin") or Android
  target_link_libraries(hb-pow-example rt)
endif()

# Tests

add_executable(hb-test test/hb-test.c)
target_link_libraries(hb-test hbs)
add_executable(hb-acc-test test/hb-acc-test.c)
target_link_libraries(hb-acc-test hbs-acc)
add_executable(hb-pow-test test/hb-pow-test.c)
target_link_libraries(hb-pow-test hbs-pow)
add_executable(hb-acc-pow-test test/hb-acc-pow-test.c)
target_link_libraries(hb-acc-pow-test hbs-acc-pow)

# pkg-config

set(PKG_CONFIG_EXEC_PREFIX "\${prefix}")
set(PKG_CONFIG_LIBDIR "\${prefix}/lib")
set(PKG_CONFIG_INCLUDEDIR "\${prefix}/include/heartbeats-simple")
set(PKG_CONFIG_CFLAGS "-I\${includedir}")

set(PKG_CONFIG_NAME "heartbeats-simple")
set(PKG_CONFIG_DESCRIPTION "Simple performance monitoring API")
set(PKG_CONFIG_LIBS "-L\${libdir} -lhbs")
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/pkgconfig.in"
  "${CMAKE_CURRENT_BINARY_DIR}/pkgconfig/hbs.pc"
)
set(PKG_CONFIG_NAME "heartbeats-simple-static")
set(PKG_CONFIG_LIBS "-L\${libdir} -lhbs-static")
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/pkgconfig.in"
  "${CMAKE_CURRENT_BINARY_DIR}/pkgconfig/hbs-static.pc"
)

set(PKG_CONFIG_NAME "heartbeats-simple-acc")
set(PKG_CONFIG_DESCRIPTION "Simple performance monitoring API with accuracy tracking")
set(PKG_CONFIG_LIBS "-L\${libdir} -lhbs-acc")
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/pkgconfig.in"
  "${CMAKE_CURRENT_BINARY_DIR}/pkgconfig/hbs-acc.pc"
)
set(PKG_CONFIG_NAME "heartbeats-simple-acc-static")
set(PKG_CONFIG_LIBS "-L\${libdir} -lhbs-acc-static")
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/pkgconfig.in"
  "${CMAKE_CURRENT_BINARY_DIR}/pkgconfig/hbs-acc-static.pc"
)

set(PKG_CONFIG_NAME "heartbeats-simple-pow")
set(PKG_CONFIG_DESCRIPTION "Simple performance monitoring API with power/energy tracking")
set(PKG_CONFIG_LIBS "-L\${libdir} -lhbs-pow")
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/pkgconfig.in"
  "${CMAKE_CURRENT_BINARY_DIR}/pkgconfig/hbs-pow.pc"
)
set(PKG_CONFIG_NAME "heartbeats-simple-pow-static")
set(PKG_CONFIG_LIBS "-L\${libdir} -lhbs-pow-static")
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/pkgconfig.in"
  "${CMAKE_CURRENT_BINARY_DIR}/pkgconfig/hbs-pow-static.pc"
)

set(PKG_CONFIG_NAME "heartbeats-simple-acc-pow")
set(PKG_CONFIG_DESCRIPTION "Simple performance monitoring API with accuracy and power/energy tracking")
set(PKG_CONFIG_LIBS "-L\${libdir} -lhbs-acc-pow")
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/pkgconfig.in"
  "${CMAKE_CURRENT_BINARY_DIR}/pkgconfig/hbs-acc-pow.pc"
)
set(PKG_CONFIG_NAME "heartbeats-simple-acc-pow-static")
set(PKG_CONFIG_LIBS "-L\${libdir} -lhbs-acc-pow-static")
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/pkgconfig.in"
  "${CMAKE_CURRENT_BINARY_DIR}/pkgconfig/hbs-acc-pow-static.pc"
)

# Install

install(TARGETS hbs hbs-acc hbs-pow hbs-acc-pow DESTINATION lib)
install(TARGETS hbs-static hbs-acc-static hbs-pow-static hbs-acc-pow-static DESTINATION lib)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/inc/ DESTINATION include/${PROJECT_NAME})
install(DIRECTORY ${CMAKE_BINARY_DIR}/pkgconfig/ DESTINATION lib/pkgconfig)

# Uninstall

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
