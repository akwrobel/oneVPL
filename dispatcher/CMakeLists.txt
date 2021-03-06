# ##############################################################################
# Copyright (C) 2020 Intel Corporation
#
# SPDX-License-Identifier: MIT
# ##############################################################################
cmake_minimum_required(VERSION 3.10.2)

set(TARGET VPL)

if(WIN32)
  set(SOURCES
      windows/main.cpp
      windows/mfx_critical_section.cpp
      windows/mfx_dispatcher.cpp
      windows/mfx_dispatcher_log.cpp
      windows/mfx_dxva2_device.cpp
      windows/mfx_function_table.cpp
      windows/mfx_library_iterator.cpp
      windows/mfx_load_dll.cpp
      windows/mfx_win_reg_key.cpp
      windows/libmfx.def)
endif()

if(UNIX)
  set(SOURCES linux/mfxloader.cpp)

  if(NOT DEFINED MFX_MODULES_DIR)
    set(MFX_MODULES_DIR ${CMAKE_INSTALL_FULL_LIBDIR})
  endif()
  add_definitions(-DMFX_MODULES_DIR="${MFX_MODULES_DIR}")
  message(STATUS "MFX_MODULES_DIR=${MFX_MODULES_DIR}")
endif()

list(APPEND SOURCES vpl/mfx_dispatcher_vpl.cpp
     vpl/mfx_dispatcher_vpl_loader.cpp vpl/mfx_dispatcher_vpl_config.cpp)

add_library(${TARGET} SHARED "")

if(WIN32)
  # force correct name on Windows
  set_target_properties(${TARGET} PROPERTIES PREFIX "lib")
  set_target_properties(${TARGET} PROPERTIES IMPORT_PREFIX "lib")
endif()
set_target_properties(${TARGET} PROPERTIES OUTPUT_NAME "mfx")
set_target_properties(${TARGET} PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(${TARGET} PROPERTIES SOVERSION ${API_VERSION_MAJOR})

target_sources(${TARGET} PRIVATE ${SOURCES})

target_link_libraries(
  ${TARGET}
  PUBLIC vpl-api
  PRIVATE ${CMAKE_DL_LIBS})

target_include_directories(${TARGET} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}
                                             ${CMAKE_CURRENT_BINARY_DIR})

install(
  TARGETS ${TARGET}
  # EXPORT mfxTargets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT runtime
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT dev)

# export(PACKAGE VPL)

configure_file(${PROJECT_SOURCE_DIR}/cmake/templates/VPLConfig.cmake.in
               ${PROJECT_BINARY_DIR}/VPLConfig.cmake @ONLY)
configure_file(${PROJECT_SOURCE_DIR}/cmake/templates/VPLConfigVersion.cmake.in
               ${PROJECT_BINARY_DIR}/VPLConfigVersion.cmake @ONLY)

install(
  FILES "${PROJECT_BINARY_DIR}/VPLConfig.cmake"
        "${PROJECT_BINARY_DIR}/VPLConfigVersion.cmake"
  DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/vpl"
  COMPONENT dev)
