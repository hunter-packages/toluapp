cmake_minimum_required(VERSION 3.2)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.19.213.tar.gz"
    SHA1 "59b2b880178bdc1e9cce83f279f1d88feefa07a2"
    LOCAL
)

project(toluapp VERSION 1.0.93)

hunter_add_package(Lua)
find_package(Lua CONFIG REQUIRED)

add_library(
    toluapp
    src/lib/tolua_event.c
    src/lib/tolua_event.h
    src/lib/tolua_is.c
    src/lib/tolua_map.c
    src/lib/tolua_push.c
    src/lib/tolua_to.c
)

target_link_libraries(toluapp PUBLIC Lua::lua_lib)

if(CMAKE_CROSSCOMPILING)
  set(install_toluapp_exe "")
else()
  set(install_toluapp_exe tolua++)

  add_executable(
      tolua++
      src/bin/tolua.c
      src/bin/toluabind.c
  )

  target_link_libraries(tolua++ PUBLIC toluapp)

  if(APPLE)
    set_target_properties(
        tolua++
        PROPERTIES
        INSTALL_RPATH "@executable_path/../lib"
    )
  elseif(UNIX)
    # Linux (?)
    set_target_properties(
        tolua++
        PROPERTIES
        INSTALL_RPATH "\$ORIGIN/../lib"
    )
  endif()
endif()

set(generated_headers "${CMAKE_CURRENT_BINARY_DIR}/generated_headers")
set(toluapp_export "${generated_headers}/toluapp/TOLUAPP_EXPORT.h")
include(GenerateExportHeader)
generate_export_header(toluapp EXPORT_FILE_NAME ${toluapp_export})

target_include_directories(
    toluapp
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
    $<BUILD_INTERFACE:${generated_headers}>
)

# Installation (https://github.com/forexample/package-example) {

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Use:
#   * PROJECT_VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use variables:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    TARGETS toluapp ${install_toluapp_exe}
    EXPORT "${TARGETS_EXPORT_NAME}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    FILES "src/lib/tolua_event.h" "include/tolua++.h"
    DESTINATION "${include_install_dir}"
)

install(
    FILES "${toluapp_export}"
    DESTINATION "${include_install_dir}/${PROJECT_NAME}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

# }
