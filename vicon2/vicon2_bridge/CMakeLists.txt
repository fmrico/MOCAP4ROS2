cmake_minimum_required(VERSION 3.5)

project(vicon2_bridge)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(vicon2_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)

#find_package(Boost REQUIRED COMPONENTS thread)

# check for platform and use the right Vicon SDK
if(CMAKE_SIZEOF_VOID_P MATCHES "8")
    set(VICON_SDK             vicon_sdk/Vicon_SDK_1.3_Linux/64-bit/)
    set(VICON_SDK_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/${VICON_SDK})
    set(VICON_SDK_LINK_DIR    ${PROJECT_SOURCE_DIR}/${VICON_SDK})
elseif(CMAKE_SIZEOF_VOID_P MATCHES "4")
    set(VICON_SDK             vicon_sdk/Vicon_SDK_1.3_Linux/32-bit/)
    set(VICON_SDK_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/${VICON_SDK})
    set(VICON_SDK_LINK_DIR    ${PROJECT_SOURCE_DIR}/${VICON_SDK})
else(CMAKE_SIZEOF_VOID_P MATCHES "8")
    message(FATAL_ERROR "unable to detect platform")
endif (CMAKE_SIZEOF_VOID_P MATCHES "8")

set(dependencies
    rclcpp
    tf2
    tf2_ros
    vicon2_msgs
)

find_library(VICON_SDK_LIBRARY ViconDataStreamSDK_CPP PATHS ${VICON_SDK_LINK_DIR})
find_library(VICON_DEBUG_SERVICES_LIBRARY DebugServices PATHS ${VICON_SDK_LINK_DIR})

message(STATUS "VICON_SDK_LIBRARY " ${VICON_SDK_LIBRARY})
message(STATUS "VICON_DEBUG_SERVICES_LIBRARY " ${VICON_DEBUG_SERVICES_LIBRARY})
message(STATUS "VICON_SDK " ${VICON_SDK})
message(STATUS "VICON_SDK_INCLUDE_DIR " ${VICON_SDK_INCLUDE_DIR})

include_directories(include
  ${VICON_SDK_INCLUDE_DIR}
  vicon_sdk/Vicon_SDK_1.3_Linux/64-bit
)

add_library(${PROJECT_NAME} SHARED src/vicon2_bridge.cpp)
ament_target_dependencies(${PROJECT_NAME} ${dependencies})

add_executable(vicon2_bridge_main
  src/vicon2_bridge_main.cpp
)
ament_target_dependencies(vicon2_bridge_main ${dependencies})
target_link_libraries(vicon2_bridge_main
  ${PROJECT_NAME}
  ${VICON_SDK_LIBRARY}
  ${VICON_DEBUG_SERVICES_LIBRARY}
)

ament_target_dependencies(vicon2_bridge ${dependencies})
target_link_libraries(vicon2_bridge
  ${VICON_SDK_LIBRARY}
  ${VICON_DEBUG_SERVICES_LIBRARY}
)

install(TARGETS
  ${PROJECT_NAME}
  vicon2_bridge_main
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})
ament_export_dependencies(${dependencies})
ament_package()
