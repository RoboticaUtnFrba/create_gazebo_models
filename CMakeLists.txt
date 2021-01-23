cmake_minimum_required(VERSION 3.5.2)
project(create_gazebo_models)

add_compile_options(-std=c++11)

## By adding -Wall and -Werror, the compiler does not ignore warnings anymore,
## enforcing cleaner code.
add_definitions(-Wall -Werror)

# Supress a warning
cmake_policy(SET CMP0054 OLD)
find_package(gazebo 9 REQUIRED)
cmake_policy(SET CMP0054 NEW)

find_package(catkin REQUIRED
  COMPONENTS
    message_generation
    std_msgs
)

add_message_files(
  DIRECTORY msg
)

# Generate services in the 'srv' folder
add_service_files(
  DIRECTORY srv
)

# Generate added messages and services with any dependencies listed here
generate_messages(
  DEPENDENCIES
    std_msgs
)

########################
## ServiceSim plugins ##
########################

include_directories(
	${catkin_INCLUDE_DIRS}
	${GAZEBO_INCLUDE_DIRS}
)

set(SERVICESIM_PLUGIN_LIST
  AttachModelPlugin
  CollisionActorPlugin
  FollowActorPlugin
  TrajectoryActorPlugin
  VicinityPlugin
)

catkin_package(
  LIBRARIES ${SERVICESIM_PLUGIN_LIST}
  CATKIN_DEPENDS
    message_runtime
    std_msgs
)

foreach(PLUGIN ${SERVICESIM_PLUGIN_LIST})
  add_library(${PLUGIN} SHARED src/servicesim/${PLUGIN}.cc)

  add_dependencies(${PLUGIN} ${catkin_EXPORTED_TARGETS} ${${PROJECT_NAME}_EXPORTED_TARGETS})

  target_link_libraries(${PLUGIN}
    ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES})

  install(TARGETS ${PLUGIN}
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
  )
endforeach()

#############
## Install ##
#############

install(DIRECTORY
			media
			models
		DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
