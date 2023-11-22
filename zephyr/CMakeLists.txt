# SPDX-License-Identifier: Apache-2.0
if (CONFIG_COMMON_INTERFACES)

zephyr_library_named(common_interfaces)

target_compile_definitions(common_interfaces PUBLIC
  )
target_compile_options(common_interfaces PRIVATE -Wno-unused-variable -Wno-implicit-function-declaration -Wno-int-conversion)

zephyr_include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}/..
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}/common_interfaces
  ${ZEPHYR_CURRENT_MODULE_DIR}/../rosidl/rosidl_runtime_c/include
  ${ZEPHYR_CURRENT_MODULE_DIR}/../rosidl/rosidl_typesupport_interface/include
  )

zephyr_library_include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}/..
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}/common_interfaces
  ${ZEPHYR_CURRENT_MODULE_DIR}/../rosidl/rosidl_runtime_c/include
  ${ZEPHYR_CURRENT_MODULE_DIR}/../rosidl/rosidl_typesupport_interface/include
  )

function(add_package package sources)
  set(new_sources)
  file(GLOB_RECURSE msgs
    RELATIVE ${ZEPHYR_CURRENT_MODULE_DIR}/${package}
    "${ZEPHYR_CURRENT_MODULE_DIR}/${package}/msg/*.msg"
  )
  message("msg: ${msgs}")
  foreach(msg ${msgs})
    get_filename_component(file ${msg} NAME ABSOLUTE)
    get_filename_component(dir1 ${msg} DIRECTORY ABSOLUTE)
    get_filename_component(name ${msg} NAME_WE)
    cmake_path(GET dir1 PARENT_PATH dir2)

    # snake case name
    string(REGEX REPLACE "([a-z0-9]+)([A-Z]+)" "\\1_\\2" snake_case_name "${name}")
    string(REGEX REPLACE "([A-Z])([A-Z])([a-z0-9]+)" "\\1_\\2\\3" snake_case_name "${snake_case_name}")
    string(TOLOWER "${snake_case_name}" snake_case_name)
    string(REGEX REPLACE "^_(.*)" "\\1" snake_case_name "${snake_case_name}")
    string(REGEX REPLACE "uint" "u_int" snake_case_name "${snake_case_name}")
    #string(REGEX REPLACE "([23])d" "//1_D" snake_case_name "${snake_case_name}")

    message(STATUS "snake_case_name: ${snake_case_name}")
    message(STATUS "package: ${package}")
    message(STATUS "name: ${name}")
    message(STATUS "file: ${file}")

    set(output ${CMAKE_CURRENT_BINARY_DIR}/${package}/msg/detail/${snake_case_name}__functions.c)
    message(STATUS "output: ${output}")

    list(APPEND new_sources ${output})
    add_custom_command(
      OUTPUT ${output}
      COMMAND rosidl generate -o ${CMAKE_CURRENT_BINARY_DIR}/${package} -t c ${package} ${msg}
      WORKING_DIRECTORY ${ZEPHYR_CURRENT_MODULE_DIR}/${package}
    )
  endforeach()
  message(STATUS "sources internal: ${new_sources}")
  set(${sources} ${new_sources} ${${sources}} PARENT_SCOPE)
endfunction()

set(sources)
add_package("std_msgs" std_msgs_sources)
add_package("geometry_msgs" geometry_msgs_sources)
add_package("sensor_msgs" sensor_msgs_sources)
add_package("nav_msgs" nav_msgs_sources)
add_package("trajectory_msgs" trajectory_msgs_sources)

zephyr_library_sources(
  ${std_msgs_sources}
  ${geometry_msgs_sources}
  ${sensor_msgs_sources}
  ${nav_msgs_sources}
  ${trajectory_msgs_sources}
)

include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}
)

add_dependencies(common_interfaces rcl_interfaces)
add_dependencies(app  common_interfaces)

endif()
