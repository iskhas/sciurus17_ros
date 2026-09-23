cmake_minimum_required(VERSION 3.20)
if(POLICY CMP0167)
  cmake_policy(SET CMP0167 NEW)
endif()
project(sciurus17_examples)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++20
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 20)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(angles REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(control_msgs REQUIRED)
find_package(image_geometry REQUIRED)
find_package(image_transport REQUIRED)
find_package(message_filters REQUIRED)
find_package(moveit_ros_planning_interface REQUIRED)
find_package(OpenCV REQUIRED COMPONENTS core)
find_package(jsoncpp REQUIRED)
find_package(VTK REQUIRED)
find_package(PCL REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(pcl_ros REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(rclcpp_action REQUIRED)
find_package(rclcpp_components REQUIRED)

include_directories(include)

set(components_list
  color_detection_2d
  neck_jt_control
  object_tracker
  waist_jt_control
)
foreach(loop_var IN LISTS components_list)
  add_library(${loop_var} SHARED
    src/${loop_var}.cpp)
  target_compile_definitions(${loop_var}
    PRIVATE "COMPOSITION_BUILDING_DLL")
  target_include_directories(${loop_var} PRIVATE
    ${OpenCV_INCLUDE_DIRS})
  target_link_libraries(${loop_var} PRIVATE
    angles::angles
    rclcpp_components::component
    rclcpp_action::rclcpp_action
    cv_bridge::cv_bridge
    geometry_msgs::geometry_msgs
    image_geometry::image_geometry
    image_transport::image_transport
    message_filters::message_filters
    ${OpenCV_LIBRARIES}
    rclcpp::rclcpp
    tf2_ros::tf2_ros
    control_msgs::control_msgs)
endforeach()

rclcpp_components_register_nodes(color_detection_2d "sciurus17_examples::ColorDetection2D")
rclcpp_components_register_nodes(neck_jt_control "sciurus17_examples::NeckJtControl")
rclcpp_components_register_nodes(object_tracker "sciurus17_examples::ObjectTracker")
rclcpp_components_register_nodes(waist_jt_control "sciurus17_examples::WaistJtControl")

install(TARGETS
  color_detection_2d
  neck_jt_control
  object_tracker
  waist_jt_control
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

set(LIBRARY_NAME "pose_presets")
add_library(${LIBRARY_NAME}
  SHARED
  src/${LIBRARY_NAME}.cpp
)
target_include_directories(
  ${LIBRARY_NAME}
  PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
target_link_libraries(
  ${LIBRARY_NAME}
  angles::angles
  geometry_msgs::geometry_msgs
  tf2_geometry_msgs::tf2_geometry_msgs
)

ament_export_targets(export_${LIBRARY_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(
  angles
  geometry_msgs
  tf2_geometry_msgs
)
install(
  DIRECTORY include/
  DESTINATION include/${LIBRARY_NAME}
)

install(
  TARGETS ${LIBRARY_NAME}
  EXPORT export_${LIBRARY_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

# Build and install node executables
set(executable_list
  gripper_control
  pose_groupstate
  joint_values
  cartesian_path
  neck_control
  waist_control
  pick_and_place_right_arm_waist
  pick_and_place_left_arm
  pick_and_place_tf
  aruco_detection
  color_detection
  point_cloud_detection
)
foreach(loop_var IN LISTS executable_list)
  add_executable(${loop_var} src/${loop_var}.cpp)
  target_include_directories(${loop_var} PRIVATE
    ${OpenCV_INCLUDE_DIRS}
    ${pcl_ros_INCLUDE_DIRS}
    ${PCL_INCLUDE_DIRS}
  )
  target_link_libraries(${loop_var} PRIVATE
    ${LIBRARY_NAME}
    ${OpenCV_LIBRARIES}
    ${PCL_LIBRARIES}
    ${pcl_ros_LIBRARIES}
    rclcpp_action::rclcpp_action
    cv_bridge::cv_bridge
    geometry_msgs::geometry_msgs
    image_geometry::image_geometry
    image_transport::image_transport
    message_filters::message_filters
    moveit_ros_planning_interface::moveit_ros_planning_interface
    pcl_conversions::pcl_conversions
    rclcpp::rclcpp
    tf2_ros::tf2_ros
    tf2_geometry_msgs::tf2_geometry_msgs
    control_msgs::control_msgs
  )

  install(TARGETS
    ${loop_var}
    DESTINATION lib/${PROJECT_NAME}
  )
endforeach()

install(DIRECTORY
  launch
  DESTINATION share/${PROJECT_NAME}/
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
