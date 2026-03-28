cmake_minimum_required(VERSION 3.8)
set(PROJECT_NAME "yeast_ros2_utilities_node")
project(${PROJECT_NAME})

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Wno-psabi)
endif()

set(boost_libraries_list
  date_time
  filesystem
)

set(THIRD_PARTY_LIB_FOLDER "/mnt/working/third_party_libs")

set(third_party_find_package_list
  ament_cmake
  Threads
)

find_package(yeast_ros2_interfaces REQUIRED)

set(ros_find_package_list
  rclcpp
  std_msgs
  geometry_msgs
  nav_msgs
  ck_ros2_base_msgs_node
  tf2
  tf2_eigen
  tf2_ros
  tf2_geometry_msgs
  yeast_ros2_interfaces

)

set(link_libraries
  Threads::Threads
)

add_custom_target(make_inc_folder ALL
  COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_SOURCE_DIR}/include/${PROJECT_NAME}
)

add_custom_target(make_src_folder ALL
  COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_SOURCE_DIR}/src
)

add_custom_command(
  OUTPUT ${PROJECT_SOURCE_DIR}/src/yeast_msg_helper.cpp

  COMMAND python3 ${PROJECT_SOURCE_DIR}/template/render_jinja.py
    --src-dir ${PROJECT_SOURCE_DIR}/yeast-schema/data_definitions/
    --header-template ${PROJECT_SOURCE_DIR}/template/yeast_msg_helper.hpp.j2
    --source-template ${PROJECT_SOURCE_DIR}/template/yeast_msg_helper.cpp.j2
    --header-dest ${PROJECT_SOURCE_DIR}/include/${PROJECT_NAME}/yeast_msg_helper.hpp
    --source-dest ${PROJECT_SOURCE_DIR}/src/yeast_msg_helper.cpp

  DEPENDS make_inc_folder
  DEPENDS make_src_folder
  DEPENDS ${PROJECT_SOURCE_DIR}/yeast-schema/data_definitions/
  DEPENDS ${PROJECT_SOURCE_DIR}/template/yeast_msg_helper.hpp.j2
  DEPENDS ${PROJECT_SOURCE_DIR}/template/yeast_msg_helper.cpp.j2
)

foreach(tdependency ${third_party_find_package_list})
  find_package(${tdependency} REQUIRED)
endforeach()


foreach(rdependency ${ros_find_package_list})
  find_package(${rdependency} REQUIRED)
endforeach()

find_package(Boost REQUIRED COMPONENTS ${boost_libraries_list}) 
find_package (Eigen3 3.4.90 REQUIRED NO_MODULE)

add_library(${PROJECT_NAME} ${PROJECT_SOURCE_DIR}/src/yeast_msg_helper.cpp)

# Specify the include directories for your target, with the new header directory first
include_directories(SYSTEM /usr/local/include/wpimath)
target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  $<BUILD_INTERFACE:${Boost_INCLUDE_DIRS}>
)

include_directories(${PROJECT_NAME} SYSTEM
  ${THIRD_PARTY_LIB_FOLDER}/yeastcpp/include
)

foreach(rtdependency ${ros_find_package_list})
  ament_target_dependencies(${PROJECT_NAME} ${rtdependency})
endforeach()

target_link_libraries(
  ${PROJECT_NAME}
  ${THIRD_PARTY_LIB_FOLDER}/yeastcpp/build/libyeastcpp.a

  ${link_libraries}
  ${Boost_LIBRARIES}
  Eigen3::Eigen
)


target_compile_features(${PROJECT_NAME} PUBLIC c_std_99 cxx_std_20)  # Require C99 and C++20


# this line to exports the library
ament_export_targets(${PROJECT_NAME} HAS_LIBRARY_TARGET)

# install the include/my_cpp_library directory to the install/include/my_cpp_library
install(
  DIRECTORY include/${PROJECT_NAME}
  DESTINATION include
)
 
install(
  TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)


if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # comment the line when this package is in a git repo and when
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
