cmake_minimum_required(VERSION 3.10.2 FATAL_ERROR)

#============================================================================
# Initialize the project
#============================================================================
project(ardupilot_sitl_gazebo)

#------------------------------------------------------------------------
# Compile as C++14

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#============================================================================
# Find gz-cmake
#============================================================================
find_package(gz-cmake3 3.0.0 REQUIRED)
set(GZ_CMAKE_VER ${gz-cmake3_VERSION_MAJOR})

#============================================================================
# Search for project-specific dependencies
#============================================================================

#--------------------------------------
# Find gz-sim
gz_find_package(gz-sim7 REQUIRED)
set(GZ_SIM_VER ${gz-sim7_VERSION_MAJOR})

#--------------------------------------
# Find RapidJSON
find_package(RapidJSON REQUIRED)

#======================================
# Build plugin

add_library(ArduPilotPlugin
    SHARED
    src/ArduPilotPlugin.cc
    src/Socket.cpp
    src/Util.cc
)
target_include_directories(ArduPilotPlugin PUBLIC
  include
	${GZ-SIM_INCLUDE_DIRS}
)
target_link_libraries(ArduPilotPlugin PUBLIC
  ${GZ-SIM_LIBRARIES}
)

if(CYPHAL_PLUGIN)
  set(PLATFORM socketcan)
  include(~/software/libcanard_cyphal_application/CMakeLists.txt)
  include_directories(~/software/libcanard_cyphal_application/tests)
  include_directories(~/software/libcanard_cyphal_application/tests/autopilot_hitl)
  include_directories(~/software/libcanard_cyphal_application/Udral)
  add_library(ArduPilotCyphalPlugin
      SHARED
      src/ArduPilotCyphalPlugin.cc
      src/Socket.cpp
      src/Util.cc

      ~/software/libcanard_cyphal_application/Udral/actuator.cpp
      ~/software/libcanard_cyphal_application/Udral/barometer.cpp
      ~/software/libcanard_cyphal_application/Udral/gnss.cpp
      ~/software/libcanard_cyphal_application/Udral/magnetometer.cpp
      ~/software/libcanard_cyphal_application/Udral/imu.cpp

      ~/software/libcanard_cyphal_application/tests/autopilot_hitl/params.cpp
      ~/software/libcanard_cyphal_application/tests/autopilot_hitl/hitl_application.cpp
      ${CYPHAL_SRC}
  )
  target_include_directories(ArduPilotCyphalPlugin PUBLIC
    include
    ${GZ-SIM_INCLUDE_DIRS}
  )
  target_link_libraries(ArduPilotCyphalPlugin PUBLIC
    ${GZ-SIM_LIBRARIES}
  )
endif()

add_library(ParachutePlugin
  SHARED
  src/ParachutePlugin.cc
)
target_include_directories(ParachutePlugin PUBLIC
  include
  ${GZ-SIM_INCLUDE_DIRS}
)

target_link_libraries(ParachutePlugin PUBLIC
  ${GZ-SIM_LIBRARIES}
)