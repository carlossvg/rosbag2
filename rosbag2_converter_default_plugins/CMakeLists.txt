cmake_minimum_required(VERSION 3.5)
project(rosbag2_converter_default_plugins)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Windows supplies macros for min and max by default. We should only use min and max from stl
if(WIN32)
  add_definitions(-DNOMINMAX)
endif()

find_package(ament_cmake REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(poco_vendor REQUIRED)
find_package(Poco COMPONENTS Foundation)
find_package(pluginlib REQUIRED)
find_package(rcutils REQUIRED)
find_package(rmw REQUIRED)
find_package(rmw_fastrtps_cpp QUIET)
find_package(rosbag2_cpp REQUIRED)
find_package(rosidl_generator_cpp REQUIRED)

if(rmw_fastrtps_cpp_FOUND)
  add_library(${PROJECT_NAME} SHARED
    src/rosbag2_converter_default_plugins/cdr/cdr_converter.cpp)

  ament_target_dependencies(${PROJECT_NAME}
    ament_index_cpp
    pluginlib
    Poco
    rcutils
    rmw
    rosbag2_cpp
    rosidl_generator_cpp)

  target_include_directories(${PROJECT_NAME}
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)

  pluginlib_export_plugin_description_file(rosbag2_cpp plugin_description.xml)

  install(
    TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin)

  if(BUILD_TESTING)
    find_package(ament_cmake_gmock REQUIRED)
    find_package(ament_lint_auto REQUIRED)
    find_package(rcutils REQUIRED)
    find_package(rosbag2_cpp REQUIRED)
    find_package(rosbag2_test_common REQUIRED)
    find_package(test_msgs REQUIRED)

    ament_lint_auto_find_test_dependencies()

    ament_add_gmock(test_cdr_converter
      test/rosbag2_converter_default_plugins/cdr/test_cdr_converter.cpp
      src/rosbag2_converter_default_plugins/cdr/cdr_converter.cpp
      WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
    if(TARGET test_cdr_converter)
      ament_target_dependencies(test_cdr_converter
        pluginlib
        rcutils
        rmw_fastrtps_cpp
        rosbag2_cpp
        rosbag2_test_common
        test_msgs)
    endif()
  endif()
else()
  message(STATUS "Skipping [${PROJECT_NAME}]. rmw_fastrtps_cpp isn't available.")
endif()

ament_package()
