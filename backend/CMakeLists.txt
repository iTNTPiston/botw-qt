cmake_minimum_required(VERSION 3.16...3.21)

# Change obs-plugintemplate to your plugin's name in a machine-readable format (e.g.:
# obs-myawesomeplugin) and set
project(botw-qt-backend VERSION 0.1.0)
add_library(${CMAKE_PROJECT_NAME} MODULE)

set(CMAKE_CXX_STANDARD 20)

# Replace `Your Name Here` with the name (yours or your organization's) you want to see as the
# author of the plugin (in the plugin's metadata itself and in the installers)
set(PLUGIN_AUTHOR "iTNTPiston")

# Replace `com.example.obs-plugin-template` with a unique Bundle ID for macOS releases (used both in
# the installer and when submitting the installer for notarization)
set(MACOS_BUNDLEID "com.piston.${CMAKE_PROJECT_NAME}")

# Replace `me@contoso.com` with the maintainer email address you want to put in Linux packages
set(LINUX_MAINTAINER_EMAIL "me@mymailhost.com")

# Add your custom source files here - header files are optional and only required for visibility
# e.g. in Xcode or Visual Studio
target_sources(${CMAKE_PROJECT_NAME} PRIVATE src/botwqt.cpp)
target_sources(${CMAKE_PROJECT_NAME} PRIVATE src/botwqt-frame.cpp)
target_sources(${CMAKE_PROJECT_NAME} PRIVATE src/botwqt-server.cpp)

# Import libobs as main plugin dependency
find_package(libobs REQUIRED)
include(cmake/ObsPluginHelpers.cmake)

# Include OpenCV
find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )
target_link_libraries( ${CMAKE_PROJECT_NAME} PRIVATE ${OpenCV_LIBS} )

# Include Tesseract
# find_package(Tesseract REQUIRED)
# include_directories( ${Tesseract_INCLUDE_DIRS} )
# target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE ${Tesseract_LIBRARY_DIRS}/${Tesseract_LIBRARIES}.lib)

# Include Websocketpp from obs-websocket
include_directories(
  ${ObsWebSocket_DIR}/deps/asio/asio/include
  ${ObsWebSocket_DIR}/deps/websocketpp
)

add_definitions(-DASIO_STANDALONE)
if(WIN32)
  add_definitions(-D_WEBSOCKETPP_CPP11_STRICT_)
endif()
if(MSVC)
  target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE /wd4127 /wd4267 /wd4996)
else()
  target_compile_options(
    ${CMAKE_PROJECT_NAME}
    PRIVATE
      -Wall
      "$<$<COMPILE_LANG_AND_ID:CXX,GNU>:-Wno-error=format-overflow>"
      "$<$<COMPILE_LANG_AND_ID:CXX,AppleClang,Clang>:-Wno-error=null-pointer-subtraction;-Wno-error=deprecated-declarations>"
  )
endif()

# Uncomment these lines if you want to use the OBS Frontend API in your plugin
#[[
find_package(obs-frontend-api REQUIRED)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OBS::obs-frontend-api)
#]]

# Use Qt
find_qt(COMPONENTS Widgets Core)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE Qt::Core Qt::Widgets)
set_target_properties(
  ${CMAKE_PROJECT_NAME}
  PROPERTIES AUTOMOC ON
             AUTOUIC ON
             AUTORCC ON)


configure_file(src/botwqt-metadata.h.in ${CMAKE_SOURCE_DIR}/src/botwqt-metadata.generated.h)

target_sources(${CMAKE_PROJECT_NAME} PRIVATE src/botwqt.h)
target_sources(${CMAKE_PROJECT_NAME} PRIVATE src/botwqt-metadata.generated.h)

target_compile_features(${CMAKE_PROJECT_NAME} PRIVATE cxx_std_20)

# --- Platform-independent build settings ---

target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/src)

target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OBS::libobs)

# --- End of section ---

# --- Windows-specific build settings and tasks ---
if(OS_WINDOWS)
  configure_file(cmake/bundle/windows/installer-Windows.iss.in
                 ${CMAKE_BINARY_DIR}/installer-Windows.generated.iss)

  configure_file(cmake/bundle/windows/resource.rc.in ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.rc)
  target_sources(${CMAKE_PROJECT_NAME} PRIVATE ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.rc)

  if(MSVC)
    target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE /W4)
  endif()
  # --- End of section ---

  # -- macOS specific build settings and tasks --
elseif(OS_MACOS)
  configure_file(cmake/bundle/macos/installer-macos.pkgproj.in
                 ${CMAKE_BINARY_DIR}/installer-macos.generated.pkgproj)

  set(MACOSX_PLUGIN_GUI_IDENTIFIER "${MACOS_BUNDLEID}")
  set(MACOSX_PLUGIN_BUNDLE_VERSION "${CMAKE_PROJECT_VERSION}")
  set(MACOSX_PLUGIN_SHORT_VERSION_STRING "1")

  target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE -Wall)
  # --- End of section ---

  # --- Linux-specific build settings and tasks ---
else()
  target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE -Wall)
endif()
# --- End of section ---

setup_plugin_target(${CMAKE_PROJECT_NAME})
