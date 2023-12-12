cmake_minimum_required(VERSION 3.14)
project(kalman_filter)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
  set(CMAKE_CXX_EXTENSIONS OFF)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Werror)
endif()

find_package(Eigen3 REQUIRED)

add_library(${PROJECT_NAME} SHARED
  src/kalman_filter.cpp
  src/time_delay_kalman_filter.cpp
)

target_include_directories(${PROJECT_NAME}
  PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>"
)

target_link_libraries(${PROJECT_NAME}
  Eigen3::Eigen
)

if(BUILD_TESTING)
  find_package(GTest REQUIRED)
  enable_testing()
  file(GLOB_RECURSE test_files test/*.cpp)
  add_executable(test_${PROJECT_NAME} ${test_files})

  target_link_libraries(test_${PROJECT_NAME}
    ${PROJECT_NAME}
    GTest::gtest_main
  )
  gtest_discover_tests(test_${PROJECT_NAME})
endif()

install(TARGETS ${PROJECT_NAME} EXPORT export_${PROJECT_NAME})
install(DIRECTORY include/ DESTINATION include/${PROJECT_NAME})

install(
  EXPORT export_${PROJECT_NAME}
  DESTINATION share/${PROJECT_NAME}/cmake
  NAMESPACE "${PROJECT_NAME}::"
  FILE "${PROJECT_NAME}Export.cmake"
)

# catkin/ament/colcon integration (ROS1/2)
install(FILES package.xml DESTINATION share/${PROJECT_NAME})
# Allows Colcon to find non-Ament packages when using workspace underlays
file(
  WRITE
  ${CMAKE_CURRENT_BINARY_DIR}/share/ament_index/resource_index/packages/${PROJECT_NAME}
  "")
install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/share/ament_index/resource_index/packages/${PROJECT_NAME}
  DESTINATION share/ament_index/resource_index/packages)
file(
  WRITE
  ${CMAKE_CURRENT_BINARY_DIR}/share/${PROJECT_NAME}/hook/ament_prefix_path.dsv
  "prepend-non-duplicate;AMENT_PREFIX_PATH;")
install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/share/${PROJECT_NAME}/hook/ament_prefix_path.dsv
  DESTINATION share/${PROJECT_NAME}/hook)
