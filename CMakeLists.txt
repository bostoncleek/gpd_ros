cmake_minimum_required(VERSION 3.0)
project(gpd_ros)


# add_compile_options(-O3 -fopenmp -fPIC -Wno-deprecated -Wenum-compare -Wno-ignored-attributes)
add_compile_options(-fopenmp -fPIC -Wno-deprecated -Wenum-compare -Wno-ignored-attributes)
set(CMAKE_CXX_STANDARD 14)


## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
							eigen_conversions
							geometry_msgs
							message_generation
							message_runtime
							roscpp
							rospy
							sensor_msgs
							std_msgs)


# PCL
find_package(PCL REQUIRED)
add_definitions(${PCL_DEFINITIONS})

# Eigen
# find_package(Eigen3 3.3 REQUIRED NO_MODULE)



## System dependencies are found with CMake's conventions
find_library(GPD_LIB NAMES gpd PATHS /usr/local/lib PATH_SUFFIXES lib NO_DEFAULT_PATH)
if (GPD_LIB)
  message(STATUS "Library GPD found in ${GPD_LIB}")
else()
  message(FATAL_ERROR "Library GPD not found")
endif()

find_path(GPD_INCLUDE_DIRS NAMES gpd PATHS /usr/local/include NO_DEFAULT_PATH)
if (GPD_INCLUDE_DIRS)
	message(STATUS "Include directory GPD found in ${GPD_INCLUDE_DIRS}")
else()
  message(FATAL_ERROR "Include directory GPD not found")
endif()



## Generate messages in the 'msg' folder
add_message_files(FILES
                  CloudIndexed.msg
									CloudSamples.msg
									CloudSources.msg
									GraspConfig.msg
									GraspConfigList.msg
  								SamplesMsg.msg
									)


## Generate services in the 'srv' folder
add_service_files(FILES detect_grasps.srv)

## Generate added messages and services with any dependencies listed here
generate_messages(DEPENDENCIES
                  geometry_msgs
									sensor_msgs
									std_msgs
									)

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if your package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
	INCLUDE_DIRS include
	LIBRARIES ${PROJECT_NAME}
	CATKIN_DEPENDS
		eigen_conversions
		geometry_msgs
		message_runtime
		roscpp
		sensor_msgs
		std_msgs
	# DEPENDS PCL
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(include
 										${catkin_INCLUDE_DIRS}
										${PCL_INCLUDE_DIRS}
										${GPD_LIB_INCLUDE_DIR}
										)


## Declare a C++ library
add_library(${PROJECT_NAME}
 							src/${PROJECT_NAME}/grasp_messages.cpp
							src/${PROJECT_NAME}/grasp_plotter.cpp
						)


add_dependencies(${PROJECT_NAME}
                 ${${PROJECT_NAME}_EXPORTED_TARGETS}
								 ${catkin_EXPORTED_TARGETS}
								 )


## Declare a C++ executable
add_executable(detect_grasps src/${PROJECT_NAME}/grasp_detection_node.cpp)
add_executable(detect_grasps_server src/${PROJECT_NAME}/grasp_detection_server.cpp)


## Specify libraries to link a library or executable target against
target_link_libraries(${PROJECT_NAME}
                      ${catkin_LIBRARIES}
											${GPD_LIB}
											)


target_link_libraries(detect_grasps
                      ${catkin_LIBRARIES}
											${PROJECT_NAME}
											${PCL_LIBRARIES}
											${GPD_LIB}
											)


target_link_libraries(detect_grasps_server
                      ${catkin_LIBRARIES}
											${PROJECT_NAME}
											${PCL_LIBRARIES}
											${GPD_LIB}
											)

install(TARGETS detect_grasps detect_grasps_server
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)
