cmake_minimum_required(VERSION 2.8.3)
project(kaqi_driver)

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++11.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  diagnostic_msgs
  dynamic_reconfigure
  geometry_msgs
  message_generation
  std_msgs
  nav_msgs
  kaqi_msgs
  roscpp
  sound_play
  serial
  tf
)

## System dependencies are found with CMake's conventions
find_package(Boost REQUIRED COMPONENTS system thread)

########################################################################################
## not use, uncomment ${UIS_HDRS} if try to use
option(UseQt5 "UseQt5" OFF)
if(UseQt5)
  find_package(Qt5Widgets REQUIRED)
  find_package(Qt5Core REQUIRED)
  find_package(Qt5OpenGL REQUIRED)
else()
  find_package(Qt4 REQUIRED COMPONENTS QtCore QtGui QtOpenGL)
endif()

if(NOT UseQt5)
  include(${QT_USE_FILE})
endif()
add_definitions(-DQT_NO_KEYWORDS)

if(UseQt5)
    qt5_wrap_cpp( MOC_FILES src/drive_widget.h src/kaqi_dashboard.h )
else()
    qt4_wrap_cpp( MOC_FILES src/drive_widget.h src/kaqi_dashboard.h )
endif()

## Here we specify the list of source files, including the output of
## the previous command which is stored in ``${MOC_FILES}``.
set(SRC_FILES
    src/drive_widget.cpp
    src/kaqi_dashboard.cpp
    ${MOC_FILES}
##    ${UIS_HDRS}
)

catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES kaqi_driver
#  CATKIN_DEPENDS diagnostic_msgs dynamic_reconfigure geometry_msgs message_generation message_runtime  std_msgs nav_msgs kaqi_msgs roscpp serial tf
#  DEPENDS system_lib
)


include_directories(
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)


## Base driver
add_executable(base_driver_node src/base_driver_node.cpp src/base_driver.cpp)
target_link_libraries(base_driver_node
  ${catkin_LIBRARIES}
)

## Dashboard
add_executable( kaqi_dashboard src/kaqi_dashboard_node.cpp ${SRC_FILES} )
target_link_libraries( kaqi_dashboard ${Boost_LIBRARIES} ${QT_LIBRARIES} ${catkin_LIBRARIES})


