cmake_minimum_required(VERSION 3.0.2)
project(tf_conversions_python3)

find_package(orocos_kdl REQUIRED)
find_package(catkin REQUIRED geometry_msgs kdl_conversions tf)
find_package(Eigen3 REQUIRED)

set(Python_ADDITIONAL_VERSIONS 3.6;3.7;3.8) ## https://stackoverflow.com/questions/29245558/how-to-link-with-python3-libs-with-cmake
find_package(PythonInterp 3 REQUIRED) ## tf_python3 :: use Python3 by default
find_package(PythonLibs "${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}")

catkin_python_setup()

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  DEPENDS EIGEN3 orocos_kdl
  CATKIN_DEPENDS geometry_msgs kdl_conversions tf
)

include_directories(include ${catkin_INCLUDE_DIRS} ${EIGEN3_INCLUDE_DIRS} ${orocos_kdl_INCLUDE_DIRS})

# Needed due to no full filename in orocos_kdl pkg-config export
link_directories(${orocos_kdl_LIBRARY_DIRS})

add_library(${PROJECT_NAME}
  src/tf_kdl.cpp
  src/tf_eigen.cpp
)
add_dependencies(${PROJECT_NAME} ${catkin_EXPORTED_TARGETS})
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES} ${orocos_kdl_LIBRARIES})

# Tests
if(CATKIN_ENABLE_TESTING)


catkin_add_gtest(test_eigen_tf test/test_eigen_tf.cpp)
target_link_libraries(test_eigen_tf ${PROJECT_NAME} ${orocos_kdl_LIBRARIES})

catkin_add_gtest(test_kdl_tf test/test_kdl_tf.cpp)
if(CMAKE_COMPILER_IS_GNUCXX)
set_target_properties(test_kdl_tf PROPERTIES COMPILE_FLAGS "-Wno-deprecated-declarations")
endif()
target_link_libraries(test_kdl_tf ${PROJECT_NAME} ${orocos_kdl_LIBRARIES})

catkin_add_nosetests(test/posemath.py)

endif()

## tf_conversions_python3 :: we can share include, library and executable directories.
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})

# install(TARGETS ${PROJECT_NAME}
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})

## tf_pytyhon3 :: re-set PROJECT_NAME to tf
set(PROJECT_NAME tf)
## tf_pytyhon3 :: re-set CATKIN_PACKAGE_PYTHON_DESTINATION with PROJECT_NAME as tf
set(CATKIN_PACKAGE_PYTHON_DESTINATION lib/python${PYTHON_VERSION_MAJOR}/dist-packages/${PROJECT_NAME})
