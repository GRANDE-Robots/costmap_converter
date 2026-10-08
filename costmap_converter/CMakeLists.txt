cmake_minimum_required(VERSION 3.5)
project(costmap_converter)

# Set to Release in order to speed up the program significantly
set(CMAKE_BUILD_TYPE Release) #None, Debug, Release, RelWithDebInfo, MinSizeRel

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(costmap_converter_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(nav2_costmap_2d REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(OpenCV REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_lifecycle REQUIRED)

set(dependencies
    costmap_converter_msgs
    geometry_msgs
    nav_msgs
    visualization_msgs
    nav2_costmap_2d
    tf2
    tf2_geometry_msgs
    OpenCV
    pluginlib
    rclcpp
    rclcpp_lifecycle
)

include_directories(
  include
)

add_library(costmap_converter SHARED
  src/costmap_to_polygons.cpp 
  src/costmap_to_polygons_concave.cpp
  src/costmap_to_lines_convex_hull.cpp
  src/costmap_to_lines_ransac.cpp
  src/costmap_to_dynamic_obstacles/costmap_to_dynamic_obstacles.cpp
  src/costmap_to_dynamic_obstacles/background_subtractor.cpp
  src/costmap_to_dynamic_obstacles/blob_detector.cpp
  src/costmap_to_dynamic_obstacles/multitarget_tracker/Ctracker.cpp
  src/costmap_to_dynamic_obstacles/multitarget_tracker/Kalman.cpp
  src/costmap_to_dynamic_obstacles/multitarget_tracker/HungarianAlg.cpp
)
target_include_directories(costmap_converter PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

if(COMMAND ament_target_dependencies)
  ament_target_dependencies(costmap_converter 
      ${dependencies}
  )
else()
  foreach(dependency IN ITEMS ${dependencies})
    if(dependency STREQUAL "Boost")
      target_link_libraries(costmap_converter  Boost::headers)
    elseif(dependency STREQUAL "OpenCV")
      target_include_directories(costmap_converter PUBLIC ${OpenCV_INCLUDE_DIRS})
      target_link_libraries(costmap_converter  ${OpenCV_LIBRARIES})
    elseif(NOT DEFINED ${dependency}_TARGETS OR "${${dependency}_TARGETS}" STREQUAL "")
      message(FATAL_ERROR "${dependency} has no exported targets for costmap_converter")
    else()
      target_link_libraries(costmap_converter  ${${dependency}_TARGETS})
    endif()
  endforeach()
endif()

install(TARGETS costmap_converter
   EXPORT costmap_converter_targets
   DESTINATION lib
)
ament_export_targets(costmap_converter_targets HAS_LIBRARY_TARGET)

install(DIRECTORY include/
   DESTINATION include/
)

ament_export_include_directories(include)
ament_export_libraries(costmap_converter)
ament_export_dependencies(${dependencies})
pluginlib_export_plugin_description_file(costmap_converter costmap_converter_plugins.xml)

ament_package()
