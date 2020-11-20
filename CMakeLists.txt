cmake_minimum_required(VERSION 3.6)

include("cmake/HunterGate.cmake")
HunterGate(
  URL "https://github.com/cpp-pm/hunter/archive/v0.23.284.tar.gz"
  SHA1 "b15e14b31d96ee2e2be32187d21e2390d0b974c8"
)

PROJECT (tinygltf VERSION 2.5.0)

if(NOT HUNTER_ENABLED)
  SET(CMAKE_CXX_STANDARD 11)
endif()

option(TINYGLTF_BUILD_LOADER_EXAMPLE "Build loader_example" ON)
option(TINYGLTF_BUILD_GL_EXAMPLES "Build GL exampels(requires glfw, OpenGL, etc)" OFF)
option(TINYGLTF_BUILD_VALIDATOR_EXAMPLE "Build validator exampe" OFF)

if (TINYGLTF_BUILD_LOADER_EXAMPLE)
  ADD_EXECUTABLE ( loader_example
    loader_example.cc
    )
endif (TINYGLTF_BUILD_LOADER_EXAMPLE)

if (TINYGLTF_BUILD_GL_EXAMPLES)
  ADD_SUBDIRECTORY ( examples/gltfutil )
  ADD_SUBDIRECTORY ( examples/glview )
endif (TINYGLTF_BUILD_GL_EXAMPLES)

if (TINYGLTF_BUILD_VALIDATOR_EXAMPLE)
  ADD_SUBDIRECTORY ( examples/validator )
endif (TINYGLTF_BUILD_VALIDATOR_EXAMPLE)
#
# TinuGLTF is a header-only library, so no library build. just install header files.
#

set(TINYGLTF_HEADER ${PROJECT_SOURCE_DIR}/tiny_gltf.h)
set(TINYGLTF_SOURCE ${PROJECT_SOURCE_DIR}/tiny_gltf.cc)

if(HUNTER_ENABLED)
  include(GNUInstallDirs)

  hunter_add_package(stb)
  find_package(stb CONFIG REQUIRED)

  hunter_add_package(RapidJSON)
  find_package(RapidJSON CONFIG REQUIRED)

  set(PROJECT_TARGET tinygltf)
  add_library(${PROJECT_TARGET} ${TINYGLTF_SOURCE})

  target_include_directories(${PROJECT_TARGET} PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}>)

  target_link_libraries(${PROJECT_TARGET} PUBLIC stb::stb RapidJSON::rapidjson)

  target_compile_definitions(${PROJECT_TARGET} PUBLIC TINYGLTF_USE_HUNTER)

  set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
  set(VERSION_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
  set(PROJECT_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}Config.cmake")
  set(GENERATED_HEADER "${GENERATED_DIR}/tiny_gltf.h")

  set(CONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

  set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
  set(NAMESPACE "${PROJECT_NAME}::")

  include(CMakePackageConfigHelpers)
  write_basic_package_version_file(
    "${VERSION_CONFIG}" COMPATIBILITY ExactVersion
  )
  configure_package_config_file(
    "cmake/Config.cmake.in"
    "${PROJECT_CONFIG}"
     INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
  )

  install(
    TARGETS ${PROJECT_TARGET} 
    EXPORT "${TARGETS_EXPORT_NAME}"
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  )

  file(READ ${TINYGLTF_HEADER} TINY_GLTF_HEADER_STRING)
  configure_file(${PROJECT_SOURCE_DIR}/tiny_gltf.h.in ${GENERATED_HEADER})

  install(
    FILES ${GENERATED_HEADER}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
  )
  install(
    FILES "${PROJECT_CONFIG}" "${VERSION_CONFIG}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
  )
  install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${NAMESPACE}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
  )
else()
  INSTALL ( FILES
    json.hpp
    stb_image.h
    stb_image_write.h
    tiny_gltf.h
    DESTINATION
    include
  )
  INSTALL ( FILES
    cmake/TinyGLTFConfig.cmake
    DESTINATION
    cmake
  )
endif()
