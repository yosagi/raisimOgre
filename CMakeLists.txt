cmake_minimum_required(VERSION 3.10)
project(raisimOgre VERSION 0.7.0)
cmake_policy(SET CMP0077 NEW)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

option(RAISIM_OGRE_EXAMPLES "Build example applications.")
option(RAISIM_OGRE_ENABLE_INSTALL_RULES "include install rules" ON)
option(RAISIM_OGRE_STATICLIB "build static lib" ON)

####################################
########### dependencies ###########
####################################
find_package(raisim 0.6.0 CONFIG REQUIRED)
if(NOT TARGET OgreMain)
include(FetchContent)
FetchContent_Declare(ogre
 GIT_REPOSITORY https://github.com/leggedrobotics/ogre.git
 GIT_TAG raisimOgre
)
set(OGRE_BUILD_COMPONENT_BITES ON)
set(OGRE_BUILD_COMPONENT_CSHARP OFF)
set(OGRE_BUILD_COMPONENT_JAVA OFF)
set(OGRE_BUILD_COMPONENT_BITES ON)
set(OGRE_BUILD_COMPONENT_CSHARP OFF)
set(OGRE_BUILD_COMPONENT_JAVA OFF)
set(OGRE_BUILD_COMPONENT_PYTHON OFF)
set(OGRE_BUILD_DEPENDENCIES OFF)
set(OGRE_BUILD_SAMPLES OFF)
set(OGRE_BUILD_TOOLS OFF)
set(OGRE_STATIC ${RAISIM_OGRE_STATICLIB})
set(OGRE_INSTALL_SAMPLES OFF)
set(OGRE_INSTALL_TOOLS OFF)
set(OGRE_INSTALL_DOCS OFF)
FetchContent_MakeAvailable(ogre)
endif()
find_package(Eigen3 REQUIRED HINTS ${Eigen3_HINT})

####################################
############# building #############
####################################
get_target_property(target_type OgreMain TYPE)
if(target_type STREQUAL STATIC_LIBRARY)
add_library(${PROJECT_NAME} STATIC
  ${CMAKE_CURRENT_SOURCE_DIR}/src/imgui.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/imgui_plot.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/imgui_draw.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/imgui_widgets.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ImguiManager.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ImguiRenderable.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/AssimpLoader.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/OgreVis.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/CameraMan.cpp
)
else()
add_library(${PROJECT_NAME} SHARED
  ${CMAKE_CURRENT_SOURCE_DIR}/src/imgui.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/imgui_plot.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/imgui_draw.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/imgui_widgets.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ImguiManager.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ImguiRenderable.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/AssimpLoader.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/OgreVis.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/CameraMan.cpp
)
target_compile_options(${PROJECT_NAME} PUBLIC -fPIC)
endif()
add_library(raisim::${PROJECT_NAME} ALIAS ${PROJECT_NAME})
add_subdirectory(third-party/assimp)

target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_14)
target_compile_options(${PROJECT_NAME} PRIVATE "$<$<CONFIG:RELEASE>:-O3>")
target_compile_options(${PROJECT_NAME} PRIVATE "$<$<CONFIG:DEBUG>:-O0>")
target_compile_options(${PROJECT_NAME} PRIVATE "$<$<CONFIG:DEBUG>:-g>")
target_compile_options(${PROJECT_NAME} PRIVATE "$<$<CONFIG:RELEASE>:-g>")

set(RAISIM_OGRE_RESOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/rsc)
target_compile_definitions(${PROJECT_NAME} INTERFACE -DOGRE_CONFIG_DIR=${CMAKE_CURRENT_BINARY_DIR}/ogre/ -DRAISIM_OGRE_RESOURCE_DIR=${RAISIM_OGRE_RESOURCE_DIR})


target_link_libraries(${PROJECT_NAME} PUBLIC OgreMain OgreBites OgreRTShaderSystem OgreMeshLodGenerator raisim::assimp raisim::raisim pthread zzip SDL2)
target_include_directories(${PROJECT_NAME}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${EIGEN3_INCLUDE_DIR}>
    $<INSTALL_INTERFACE:include>
)
target_compile_options(${PROJECT_NAME} PUBLIC -Wl,--no-as-needed) # workaround for gcc pthread link bug


####################################
########### installation ###########
####################################

# Install package configuration
configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/cmake/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake
)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/cmake/${PROJECT_NAME}Config-version.cmake
  VERSION ${raisimOgre_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/cmake/${PROJECT_NAME}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/cmake/${PROJECT_NAME}Config-version.cmake
  DESTINATION
    ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake
)

###################################################################################
# This section is a hack to make resource found for both example build, installation and inclusion from upper-level project
# Install resources for installed version
set(RAISIM_OGRE_RESOURCE_DIR ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/rsc)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/resources.cfg.in ${CMAKE_CURRENT_BINARY_DIR}/ogre/resources.cfg @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/ogre.cfg.in ${CMAKE_CURRENT_BINARY_DIR}/ogre/ogre.cfg @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/plugins.cfg.in ${CMAKE_CURRENT_BINARY_DIR}/ogre/plugins.cfg @ONLY)

install(
        FILES
        ${CMAKE_CURRENT_BINARY_DIR}/ogre/resources.cfg
        ${CMAKE_CURRENT_BINARY_DIR}/ogre/ogre.cfg
        ${CMAKE_CURRENT_BINARY_DIR}/ogre/plugins.cfg
        DESTINATION
        ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/ogre
)

# Install resources for this cmake build.
set(RAISIM_OGRE_RESOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/rsc)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/resources.cfg.in ${CMAKE_CURRENT_BINARY_DIR}/resources.cfg @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/ogre.cfg.in ${CMAKE_CURRENT_BINARY_DIR}/ogre.cfg @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/plugins.cfg.in ${CMAKE_CURRENT_BINARY_DIR}/plugins.cfg @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/resources.cfg.in ${CMAKE_CURRENT_BINARY_DIR}/ogre/resources.cfg @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/ogre.cfg.in ${CMAKE_CURRENT_BINARY_DIR}/ogre/ogre.cfg @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/plugins.cfg.in ${CMAKE_CURRENT_BINARY_DIR}/ogre/plugins.cfg @ONLY)
######################################################################################


add_definitions(-DOGRE_CONFIG_DIR=${CMAKE_CURRENT_BINARY_DIR}/)

install(
  DIRECTORY
    ${CMAKE_CURRENT_SOURCE_DIR}/rsc
  DESTINATION
    ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}
)

file(
  COPY
    ${CMAKE_CURRENT_SOURCE_DIR}/rsc
  DESTINATION
    ${CMAKE_CURRENT_BINARY_DIR}
)

if(RAISIM_OGRE_ENABLE_INSTALL_RULES)
# Install targets

install(
  TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}-targets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${LIBLEGACY_INCLUDE_DIRS}
)

export(
  TARGETS ${PROJECT_NAME}
  NAMESPACE raisim::
  FILE cmake/${PROJECT_NAME}-targets.cmake
)

install(
  EXPORT ${PROJECT_NAME}-targets
  FILE ${PROJECT_NAME}-targets.cmake
  NAMESPACE raisim::
  DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake
)

# Install headers
install(
  DIRECTORY
    ${CMAKE_CURRENT_SOURCE_DIR}/include/raisim
  DESTINATION
    ${CMAKE_INSTALL_INCLUDEDIR}
)
endif()

####################################
############# examples #############
####################################
if(RAISIM_OGRE_EXAMPLES)
  set(CMAKE_PREFIX_PATH ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_PREFIX_PATH})
  add_subdirectory(examples)
endif()
