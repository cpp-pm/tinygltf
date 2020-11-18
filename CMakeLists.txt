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

if(HUNTER_ENABLED)
  include(GNUInstallDirs)
  set(PROJECT_TARGET tinygltf)
  add_library(${PROJECT_TARGET} INTERFACE)
  set(TINYGLTF_HEADERS tiny_gltf.h)

  set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
  set(VERSION_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
  set(PROJECT_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}Config.cmake")

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
    INCLUDES DESTINATION include
  )

  install(
    FILES ${TINYGLTF_HEADERS}
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
