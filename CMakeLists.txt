cmake_minimum_required(VERSION 3.20)
project(robomas_plugins)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 26)
endif()
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
# uncomment the following section in order to fill in
# further dependencies manually.
# find_package(<dependency> REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(std_msgs REQUIRED)
# Boost.Asio / Boost.Bind are header-only; Boost.System is header-only since 1.69.
find_package(Boost CONFIG REQUIRED)
find_package(Threads REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Frame.msg"
  "msg/RobomasFrame.msg"
  "msg/RobomasTarget.msg"
  DEPENDENCIES std_msgs
)

ament_export_dependencies(rosidl_default_runtime)
rosidl_get_typesupport_target(cpp_typesupport_target
  ${PROJECT_NAME} "rosidl_typesupport_cpp"
)

add_library(robomas_bridge SHARED
  src/robomas_bridge.cpp)

target_include_directories(robomas_bridge PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

# ament_target_dependencies() is deprecated since Kilted; link imported targets directly.
target_link_libraries(robomas_bridge
  "${cpp_typesupport_target}"
  rclcpp::rclcpp
  rclcpp_components::component
  ${std_msgs_TARGETS}
  Boost::headers
  Threads::Threads
)

rclcpp_components_register_nodes(robomas_bridge "robomas_bridge::RobomasBridge")

install(
  DIRECTORY include/
  DESTINATION include
)
ament_export_include_directories(include)

install(TARGETS
  robomas_bridge
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

install(
  DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)


if(BUILD_TESTING)
  foreach(test_name robomas_protocol serial_write_queue target_epoch_filter)
    add_executable(test_${test_name} test/test_${test_name}.cpp)
    target_include_directories(test_${test_name} PRIVATE include)
    target_link_libraries(test_${test_name} Threads::Threads)
    add_test(NAME robomas_${test_name} COMMAND test_${test_name})
  endforeach()

  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
