cmake_minimum_required(VERSION 2.8.3)
project(profile_manager)

set (Airlib_addr  $ENV{AirSim_base_dir}/AirLib)
set (common_utility_src $ENV{base_dir}/catkin_ws/src/MAV_apps/common)
set (common_utility_include $ENV{base_dir}/catkin_ws/src/MAV_apps/common)

find_package(Eigen3 REQUIRED)
find_package(CUDA REQUIRED)
message(darknet_include  ${CUDA_LIBRARIES})
find_package(catkin REQUIRED COMPONENTS
  trajectory_msgs
  mav_msgs
  tf
  cv_bridge
  image_transport
  roscpp
  std_msgs
  geometry_msgs
  sensor_msgs
  pcl_conversions
  pcl_ros
  tf
  message_generation
  nlopt
)

#add_message_files(
#    FILES
#    )


add_service_files(
  FILES
  profiling_data_srv.srv
  start_profiling_srv.srv
  )

generate_messages(
  DEPENDENCIES
  std_msgs
  geometry_msgs
  )

#message(cuda_include_dirs )
include_directories(
    ${CATKIN_DEVEL_PREFIX}/include
    ${Eigen_INCLUDE_DIRS} 
    ${catkin_INCLUDE_DIRS}
    ${OpenCV_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIR}
    $ENV{AirSim_base_dir}/AirLib/include
    ${common_utility_include} 
    ${darknet_include_dir}
)

link_directories(
    $ENV{AirSim_base_dir}/AirLib/lib
    $ENV{AirSim_base_dir}/AirLib/deps/rpclib/lib
  /usr/local/cuda/lib64  #added by Behzad (for Behzad)
  ${Airlib_addr}/deps/MavLinkCom/lib
)

catkin_package(
  INCLUDE_DIRS ${Eigen_INCLUDE_DIRS} ${catkin_INCLUDE_DIRS}
  LIBRARIES ${catkin_LIBRARIES}
  CATKIN_DEPENDS trajectory_msgs mav_msgs tf message_runtime
)


if(${CMAKE_HOST_SYSTEM_PROCESSOR} MATCHES "x86_64")
include_directories(/usr/local/cuda/include)
link_directories(/usr/lib/nvidia-384)
add_definitions(-DUSE_INTEL -DUSE_NVML)
add_executable(profile_manager src/profile_manager.cpp ${common_utility_src}/Drone.cpp ${common_utility_src}/common.cpp ${common_utility_src}/Profiling.cpp  ${common_utility_src}/timer.cpp)
target_link_libraries(profile_manager  ${catkin_LIBRARIES}  AirLib MavLinkCom rpc pthread nvidia-ml)
else(${CMAKE_HOST_SYSTEM_PROCESSOR} MATCHES "x86_64")
add_executable(profile_manager src/profile_manager.cpp src/slam_profiler.cpp ${common_utility_src}/Drone.cpp ${common_utility_src}/common.cpp ${common_utility_src}/Profiling.cpp ${common_utility_src}/timer.cpp)
target_link_libraries(profile_manager  ${catkin_LIBRARIES}  AirLib MavLinkCom rpc pthread)
endif(${CMAKE_HOST_SYSTEM_PROCESSOR} MATCHES "x86_64")

add_dependencies(profile_manager ${catkin_EXPORTED_TARGETS} profile_manager_generate_messages_cpp)

