cmake_minimum_required(VERSION 2.4.6)
project(cob_surface_classification)

set(catkin_RUN_PACKAGES # all ROS packages from package.xml (libopencv-dev is system dependency --> sudo apt-get install)
  pcl_ros
  roscpp
  cv_bridge
  std_msgs
  sensor_msgs
  message_filters
  image_transport
  cob_3d_mapping_common
  cob_3d_segmentation
  cob_3d_features
)

set(catkin_BUILD_PACKAGES 
	${catkin_RUN_PACKAGES}
	cmake_modules
	message_generation
)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
## Creates a bunch of environment variables that may be used later in the script
## e.g. catkin_INCLUDE_DIRS

find_package(catkin REQUIRED COMPONENTS
	${catkin_BUILD_PACKAGES} # this makes ${catkin_LIBRARIES} include all libraries of ${catkin_BUILD_PACKAGES}
)
find_package(OpenCV REQUIRED)	# name identical to FindOpenCV.cmake in cmake_modules
find_package(Boost REQUIRED COMPONENTS system)
find_package(VTK REQUIRED)
# find_package(PCL REQUIRED) # is already done by inclusion of pcl_ros above


################################################
## Declare ROS messages, services and actions ##
################################################

## To declare and build messages, services or actions from within this
## package, follow these steps:
## * Let MSG_DEP_SET be the set of packages whose message types you use in
##   your messages/services/actions (e.g. std_msgs, actionlib_msgs, ...).
## * In the file package.xml:
##   * add a build_depend and a run_depend tag for each package in MSG_DEP_SET
##   * If MSG_DEP_SET isn't empty the following dependencies might have been
##     pulled in transitively but can be declared for certainty nonetheless:
##     * add a build_depend tag for "message_generation"
##     * add a run_depend tag for "message_runtime"
## * In this file (CMakeLists.txt):
##   * add "message_generation" and every package in MSG_DEP_SET to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * add "message_runtime" and every package in MSG_DEP_SET to
##     catkin_package(CATKIN_DEPENDS ...)
##   * uncomment the add_*_files sections below as needed
##     and list every .msg/.srv/.action file to be processed
##   * uncomment the generate_messages entry below
##   * add every package in MSG_DEP_SET to generate_messages(DEPENDENCIES ...)

## Generate messages in the 'msg' folder
add_message_files(
  FILES
  Int32Array.msg
  SegmentedPointCloud2.msg
)

## Generate services in the 'srv' folder
# add_service_files(
#   FILES
#   Service1.srv
#   Service2.srv
# )

## Generate actions in the 'action' folder
#add_action_files(
#  DIRECTORY
#    action
#  FILES
#    SetJointTarget.action
#    TrackNewSetpoints.action
#)

## Generate added messages and services with any dependencies listed here
generate_messages(
  DEPENDENCIES
  std_msgs
  sensor_msgs
#  actionlib_msgs  # Or other packages containing msgs
)

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if you package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need for build and run
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
INCLUDE_DIRS
	common/include
LIBRARIES
	cob_3d_curvatureSegmentation
CATKIN_DEPENDS
	${catkin_RUN_PACKAGES}
	message_runtime
DEPENDS
	OpenCV
	Boost
	VTK
#	PCL
)

###########
## Build ##
###########
## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
	common/include
	ros/include
	${catkin_INCLUDE_DIRS} # same name as find_package
	${OpenCV_INCLUDE_DIRS}
	${Boost_INCLUDE_DIRS}
	${VTK_INCLUDE_DIRS}
#	${PCL_INCLUDE_DIRS}
)

## Declares a shared library. Resulting name will be lib<name>.so
cmake_policy(PUSH)
cmake_policy(SET CMP0003 OLD) # use old-style link directories for now
add_library(cob_3d_curvatureSegmentation
	common/src/refine_segmentation.cpp
)
cmake_policy(POP)
	
target_link_libraries(cob_3d_curvatureSegmentation
	${catkin_LIBRARIES} # automatically links all catkin_BUILD_PACKAGES
	${VTK_LIBRARIES}
)

## Declare a cpp executable
cmake_policy(PUSH)
cmake_policy(SET CMP0003 OLD) # use old-style link directories for now
add_executable(surface_classification
	ros/src/surface_classification_node.cpp
	common/src/surface_classification.cpp
	ros/src/scene_recording.cpp
	ros/src/evaluation.cpp
)
cmake_policy(POP)

target_link_libraries(surface_classification
	cob_3d_curvatureSegmentation
	${catkin_LIBRARIES} # automatically links all catkin_BUILD_PACKAGES
	${Boost_LIBRARIES}
)

add_dependencies(cob_3d_curvatureSegmentation ${catkin_EXPORTED_TARGETS})
add_dependencies(surface_classification ${catkin_EXPORTED_TARGETS})

# set build flags for targets
#set_target_properties(cob_3d_curvatureSegmentation PROPERTIES COMPILE_FLAGS "-D__LINUX__ -DBOOST_FILESYSTEM_VERSION=2")
#set_target_properties(surface_classification PROPERTIES COMPILE_FLAGS "-D__LINUX__ -DBOOST_FILESYSTEM_VERSION=2")


#############
## Install ##
#############
## Mark executables and/or libraries for installation
install(TARGETS cob_3d_curvatureSegmentation surface_classification 
	ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
	LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
	RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY common/include/${PROJECT_NAME}/
	DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
	FILES_MATCHING PATTERN "*.h"
	PATTERN ".svn" EXCLUDE
)

install(DIRECTORY ros/launch
	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/ros
	PATTERN ".svn" EXCLUDE
)

#install(DIRECTORY common/files
#	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/common
#	PATTERN ".svn" EXCLUDE
#)

