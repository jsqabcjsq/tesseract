cmake_minimum_required(VERSION 3.5.0)
project(tesseract_environment VERSION 0.1.0 LANGUAGES CXX)

## System dependencies are found with CMake's conventions
find_package(Eigen3 REQUIRED)
find_package(orocos_kdl REQUIRED)
find_package(console_bridge REQUIRED)
find_package(tesseract_collision REQUIRED)
find_package(tesseract_scene_graph REQUIRED)

# Create interface for core
add_library(${PROJECT_NAME}_core SHARED src/core/environment.cpp)
target_link_libraries(${PROJECT_NAME}_core PUBLIC tesseract_collision_core tesseract_scene_graph)
target_compile_options(${PROJECT_NAME}_core PRIVATE -Wall -Wextra -Wsuggest-override -Wconversion -Wsign-conversion)
target_compile_features(${PROJECT_NAME}_core PRIVATE cxx_std_11)
target_include_directories(${PROJECT_NAME}_core PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include>")
target_include_directories(${PROJECT_NAME}_core SYSTEM PUBLIC
    ${EIGEN3_INCLUDE_DIRS})

add_library(${PROJECT_NAME}_kdl SHARED src/kdl/kdl_env.cpp src/kdl/kdl_state_solver.cpp)
target_link_libraries(${PROJECT_NAME}_kdl PUBLIC ${PROJECT_NAME}_core tesseract_collision_fcl tesseract_collision_bullet tesseract_scene_graph orocos-kdl console_bridge ${BULLET_LIBRARIES})
target_compile_options(${PROJECT_NAME}_kdl PRIVATE -Wall -Wextra -Wsuggest-override -Wconversion -Wsign-conversion)
target_compile_features(${PROJECT_NAME}_kdl PRIVATE cxx_std_11)
target_include_directories(${PROJECT_NAME}_kdl PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include>")
target_include_directories(${PROJECT_NAME}_kdl SYSTEM PUBLIC
    ${EIGEN3_INCLUDE_DIRS}
    ${BULLET_INCLUDE_DIRS}
    ${orocos_kdl_INCLUDE_DIRS}) # Orocos did not export target correctly

list (APPEND PACKAGE_LIBRARIES ${PROJECT_NAME}_core)
list (APPEND PACKAGE_LIBRARIES ${PROJECT_NAME}_kdl)

install(TARGETS ${PACKAGE_LIBRARIES} EXPORT ${PROJECT_NAME}-targets DESTINATION lib)
install(EXPORT ${PROJECT_NAME}-targets DESTINATION lib/cmake/${PROJECT_NAME})

# Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}
  DESTINATION include
  FILES_MATCHING PATTERN "*.h"
  PATTERN ".svn" EXCLUDE
)

install(FILES package.xml DESTINATION share/${PROJECT_NAME})

# Create cmake config files
include(CMakePackageConfigHelpers)
configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/${PROJECT_NAME}-config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
  INSTALL_DESTINATION lib/cmake/${PROJECT_NAME}
  NO_CHECK_REQUIRED_COMPONENTS_MACRO)

write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  VERSION ${PROJECT_VERSION} COMPATIBILITY ExactVersion)

install(FILES
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake"
  DESTINATION lib/cmake/${PROJECT_NAME})

export(EXPORT ${PROJECT_NAME}-targets FILE ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-targets.cmake)

option(ENABLE_TESTS "Enable tests" OFF)
if (${ENABLE_TESTS})
  enable_testing()
  add_custom_target(run_tests ALL
      WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
      COMMAND ${CMAKE_CTEST_COMMAND} -C $<CONFIGURATION> -V)

  add_subdirectory(test)
endif()