cmake_minimum_required(VERSION 2.8.12)
project(webrtc)

find_package(catkin REQUIRED)

set(webrtc_src_root ${PROJECT_SOURCE_DIR}/src)
set(webrtc_build_root ${CMAKE_BINARY_DIR}/out/Release)

# checkout source to build and generate ninja build files
execute_process(
  COMMAND ./checkout_source.sh ${CMAKE_BINARY_DIR}
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/
)

# generate package cfg extras to be inherited
set(CMAKE_EXTRAS_FILE ${CMAKE_BINARY_DIR}/webrtc-extras.cmake)
execute_process(
  COMMAND ./ninja_to_cmake.py ${webrtc_build_root} generate-cmake-extras ${CMAKE_EXTRAS_FILE} ${CATKIN_DEVEL_PREFIX}/${CATKIN_GLOBAL_LIB_DESTINATION} # this must come before catkin_packate, which sets up CATKIN_PACKAGE_* macros so use global instead of package
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/
)


catkin_package(
  CFG_EXTRAS ${CMAKE_EXTRAS_FILE}
)

#############
##  Build  ##
#############

add_custom_target(webrtc ALL
  COMMAND ./build_webrtc.sh ${webrtc_build_root}
  COMMAND ./ninja_to_cmake.py ${webrtc_build_root} copy-libs ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION}
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/
)

#############
## Install ##
#############

# Generate list of built libraries
execute_process(
  COMMAND ./ninja_to_cmake.py ${webrtc_build_root} list-built-libs ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION}
  OUTPUT_VARIABLE BUILT_LIBRARIES
  OUTPUT_STRIP_TRAILING_WHITESPACE
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/
)

# Install libraries
install(FILES ${BUILT_LIBRARIES}
  DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

# Install includes
install(CODE "
execute_process(
  COMMAND ./ninja_to_cmake.py ${webrtc_build_root} install-includes ${CMAKE_INSTALL_PREFIX}/${CATKIN_GLOBAL_INCLUDE_DESTINATION}
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/
)")
