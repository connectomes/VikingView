PROJECT( VikingView )

cmake_minimum_required(VERSION 2.8)  


#Match everything when analyzing dependencies
INCLUDE_REGULAR_EXPRESSION("^.*$")
OPTION (BUILD_SHARED_LIBS "Build shared libraries" OFF)
OPTION (BUILD_TESTS "Build VikingView test applications." ON)
OPTION (INSTALL_TEST_FILES "Include VikingView test images and help files in the install." OFF)
OPTION (INSTALL_SOURCE  "Install VikingView source code." OFF)
OPTION (USE_PRECOMPILED_HEADERS "Use precompiled headers to speed up compilation" OFF)
OPTION (LOAD_DEFAULT_PROJECT "Load default project" OFF)

MESSAGE(STATUS "** PRECOMPILED_HEADERS: ${USE_PRECOMPILED_HEADERS}")

#set(CMAKE_BUILD_TYPE Release)

IF(LOAD_DEFAULT_PROJECT)
  ADD_DEFINITIONS(-DLOAD_DEFAULT_PROJECT)
ENDIF(LOAD_DEFAULT_PROJECT)

IF (WIN32 AND MSVC)
  ADD_DEFINITIONS(-D_SCL_SECURE_NO_WARNINGS)
  ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS)
ENDIF (WIN32 AND MSVC)

find_path(QT5_LIB_DIR NAMES lib DOC "QT5 lib directory for desired platform.  Helps locate QT packages" NO_DEFAULT_PATH)
find_path(QT5_PLUGIN_DIR NAMES plugins DOC "QT5 plugin directory for desired platform.  Helps locate files used for distributions" NO_DEFAULT_PATH)

find_package(VTK REQUIRED)
include(${VTK_USE_FILE})

#FIND_PACKAGE(VTK COMPONENTS
 # vtkCommonCore
#  vtkInfovisCore
#  vtkInteractionStyle
#  vtkViewsQt
#  vtkFiltersCore
#  vtkGUISupportQt
#  vtkIOLegacy
#  vtkIOGeometry
#  vtkFiltersCore
#  vtkIOPLY
 # )

MESSAGE(STATUS "** USE_VTK_FILE: ${USE_VTK_FILE}")
MESSAGE(STATUS "** VTK_LIBRARIES: ${VTK_LIBRARIES}")


#INCLUDE(${VTK_USE_FILE})

#INCLUDE( ${USE_VTK_FILE} )
MESSAGE(STATUS "** VTK_DIR: ${VTK_DIR}")
#MESSAGE(STATUS "** VTK_LIBRARIES: ${VTK_LIBRARIES}")

#FIND_PACKAGE(ITK REQUIRED)
#IF (ITK_FOUND)
#  INCLUDE (${USE_ITK_FILE})
#ENDIF(ITK_FOUND)
#MESSAGE(STATUS "** ITK_DIR: ${ITK_DIR}")
#MESSAGE(STATUS "** ITK_LIBRARIES: ${ITK_LIBRARIES}")


#FIND_PACKAGE(CGAL REQUIRED)

#MESSAGE(STATUS "** USE_CGAL_FILE: ${USE_CGAL_FILE}")


#if ( CGAL_FOUND )
#  include( ${CGAL_USE_FILE} )
#ENDIF( CGAL_FOUND )

#MESSAGE(STATUS "** CGAL_DIR: ${CGAL_DIR}")
#MESSAGE(STATUS "** CGAL_LIBRARIES: ${CGAL_LIBRARIES}")




#set( Boost_DEBUG TRUE )
#SET(CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH} "/usr/local/include/boost")
#SET(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} "/usr/local/lib")
#set( Boost_USE_STATIC_LIBS ON )
#FIND_PACKAGE(Boost REQUIRED COMPONENTS system filesystem thread)
#link_directories ( ${Boost_LIBRARY_DIRS} )
#include_directories ( ${Boost_INCLUDE_DIRS} )

#MESSAGE(STATUS "** Boost Include: ${Boost_INCLUDE_DIR}")
#MESSAGE(STATUS "** Boost Libraries: ${Boost_LIBRARY_DIRS}")
#MESSAGE(STATUS "** Boost Libraries: ${Boost_LIBRARIES}")

SET(CMAKE_INCLUDE_CURRENT_DIR ON)



#ADD_DEFINITIONS( -Wall )

# by default only QtCore and QtGui modules are enabled
# other modules must be enabled like this:
SET( QT_USE_QT3SUPPORT TRUE )   
SET( QT_USE_QTXML TRUE )
SET( QT_USE_QTSVG TRUE )
SET( QT_USE_QTNETWORK TRUE )
SET( QT_USE_QTSCRIPT TRUE )

#--------------------------------------------------------------------------------
# This sets the windows build that will need the special winmain@16 call. Qt provides
#  this for us in the qtmain.lib file. Using this cmake code will ensure we have it
#  linked into our build. Not needed on Unix/OS X/Linux which is why we have the 
#  IF(WIN32) conditional.
IF (WIN32)
    SET (QT_USE_QTMAIN TRUE)
ENDIF (WIN32)

# this command finds Qt4 libraries and sets all required variables
# note that it's Qt4, not QT4 or qt4
SET(QT_MIN_VERSION "4.7.4")
#FIND_PACKAGE( Qt4 REQUIRED )
find_package(Qt5Widgets REQUIRED HINTS "${QT5_LIB_DIR}" "${QT5_LIB_DIR}/cmake")
include_directories(${Qt5Widgets_INCLUDE_DIRS})
add_definitions(${Qt5Widgets_DEFINITION_DIRS})

find_package(Qt5Script REQUIRED HINTS "${QT5_LIB_DIR}" "${QT5_LIB_DIR}/cmake")
include_directories(${Qt5Script_INCLUDE_DIRS})
add_definitions(${Qt5Script_DEFINITION_DIRS})

find_package(Qt5Core REQUIRED HINTS "${QT5_LIB_DIR}" "${QT5_LIB_DIR}/cmake")
include_directories(${Qt5Core_INCLUDE_DIRS})
add_definitions(${Qt5Core_DEFINITION_DIRS})

find_package(Qt5Concurrent REQUIRED HINTS "${QT5_LIB_DIR}" "${QT5_LIB_DIR}/cmake")
include_directories(${Qt5Concurrent_INCLUDE_DIRS})
add_definitions(${Qt5Concurrent_DEFINITION_DIRS})

find_package(Qt5Network REQUIRED HINTS "${QT5_LIB_DIR}" "${QT5_LIB_DIR}/cmake")
include_directories(${Qt5Network_INCLUDE_DIRS})
add_definitions(${Qt5Network_DEFINITION_DIRS})

find_file(QT_PLUGINS_PLATFORMS_QWINDOWS_DLL NAMES qwindows.dll HINTS "${QT5_PLUGIN_DIR}/platforms" DOC "Path to Qt/5.6/msvc2015_64/plugins/platforms/qwindows.dll for Windows installer" )

# add some useful macros and variables
# (QT_USE_FILE is a variable defined by FIND_PACKAGE( Qt4 ) that contains a path to CMake script)
#INCLUDE( ${QT_USE_FILE} )

MESSAGE(STATUS "** QT_QMAKE_EXECUTABLE: ${QT_QMAKE_EXECUTABLE}")

# Instructs the MSVC toolset to use the precompiled header PRECOMPILED_HEADER
# for each source file given in the collection named by SOURCES.
function(enable_precompiled_headers PRECOMPILED_HEADER SOURCES)
  if(MSVC)
    set(files ${${SOURCES}})

    get_filename_component(pch_basename ${PRECOMPILED_HEADER} NAME_WE)
    set(pch_header ${CMAKE_CURRENT_SOURCE_DIR}/${PRECOMPILED_HEADER})
    set(pch_file ${CMAKE_CURRENT_BINARY_DIR}/${pch_basename}.cc)
    set_source_files_properties(${pch_file}  PROPERTIES COMPILE_FLAGS "/Yc\"${pch_header}\"")

    # Generate precompiled header translation unit
    IF(NOT EXISTS "${pch_file}")
      FILE(WRITE ${pch_file} "// Precompiled header unity generated by CMake\n")
      FILE(APPEND ${pch_file} "#include <${pch_header}>\n")
    ENDIF(NOT EXISTS "${pch_file}")

    # Update properties of source files to use the precompiled header.
    # Additionally, force the inclusion of the precompiled header at beginning of each source file.
    foreach(source_file ${files} )
      get_filename_component(extension ${source_file} EXT)
      if ("${extension}" STREQUAL ".cc")
#        message ("Setting property on ${source_file}")
        set_source_files_properties(
          ${source_file}
          PROPERTIES COMPILE_FLAGS
          "/Yu\"${pch_header}\" /FI\"${pch_header}\""
          )
      endif("${extension}" STREQUAL ".cc")
      if ("${extension}" STREQUAL ".cxx")
#        message ("setting property on ${source_file}")
        set_source_files_properties(
          ${source_file}
          PROPERTIES COMPILE_FLAGS
          "/Yu\"${pch_header}\" /FI\"${pch_header}\""
          )
      endif("${extension}" STREQUAL ".cxx")
    endforeach(source_file)

    # Finally, update the source file collection to contain the precompiled header translation unit
    set(${SOURCES} ${pch_file} ${${SOURCES}} PARENT_SCOPE)
  endif(MSVC)
endfunction(enable_precompiled_headers)



### Application
SET(VIKING_VIEW_APPLICATION_HDRS
  Application/VikingViewApp.h
  Application/Preferences.h
  Application/PreferencesWindow.h
  Application/CommandLineArgs.h
)
SET(VIKING_VIEW_APPLICATION_SRCS
  Application/VikingViewApp.cc
  Application/Preferences.cc
  Application/PreferencesWindow.cc
  Application/CommandLineArgs.cc
)
SET(VIKING_VIEW_APPLICATION_UIS
  Application/VikingViewApp.ui
  Application/PreferencesWindow.ui
)

### Data
SET(VIKING_VIEW_DATA_HDRS
  Data/Json.h
#  Data/AlphaShape.h
#  Data/FixedAlphaShape.h
#  Data/PointSampler.h
  Data/Downloader.h
  Data/Structure.h
  Data/Scale.h
  )
SET(VIKING_VIEW_DATA_SRCS
  Data/Json.cc
#  Data/AlphaShape.cc
#  Data/FixedAlphaShape.cc
#  Data/PointSampler.cc
  Data/Downloader.cc
  Data/Structure.cc
  )

### Visualization
SET(VIKING_VIEW_VISUALIZATION_HDRS
  Visualization/Viewer.h
  Visualization/customQuadricDecimation.h
)
SET(VIKING_VIEW_VISUALIZATION_SRCS
  Visualization/Viewer.cc
  Visualization/customQuadricDecimation.cc
)

# ### Util
# SET(VIKING_VIEW_UTIL_HDRS
#   Util/WheelEventForwarder.h
#   )
# SET(VIKING_VIEW_UTIL_SRCS
#   Util/WheelEventForwarder.cc
#   )


### Main Stuff
SET( VIKING_VIEW_SRCS
     main.cc
     ${VIKING_VIEW_APPLICATION_SRCS}
     ${VIKING_VIEW_DATA_SRCS}
     ${VIKING_VIEW_GROOM_SRCS}
     ${VIKING_VIEW_OPTIMIZE_SRCS}
     ${VIKING_VIEW_ANALYSIS_SRCS}
     ${VIKING_VIEW_VISUALIZATION_SRCS}
     ${VIKING_VIEW_UTIL_SRCS}
     ${TINYXML_SRCS}
)

SET( VIKING_VIEW_HDRS
     ${VIKING_VIEW_APPLICATION_HDRS}
     ${VIKING_VIEW_DATA_HDRS}
     ${VIKING_VIEW_GROOM_HDRS}
     ${VIKING_VIEW_OPTIMIZE_HDRS}
     ${VIKING_VIEW_ANALYSIS_HDRS}
     ${VIKING_VIEW_VISUALIZATION_HDRS}
     ${VIKING_VIEW_UTIL_HDRS}
     ${TINYXML_HDRS}
)

SET( VIKING_VIEW_MOC_HDRS
  Data/Downloader.h
)

SET( VIKING_VIEW_RCS
     Resources/VikingView.qrc
)

SOURCE_GROUP("Application" FILES ${VIKING_VIEW_APPLICATION_SRCS} ${VIKING_VIEW_APPLICATION_HDRS} ${VIKING_VIEW_APPLICATION_UIS})
SOURCE_GROUP("Visualization" FILES ${VIKING_VIEW_VISUALIZATION_SRCS} ${VIKING_VIEW_VISUALIZATION_HDRS})
SOURCE_GROUP("Data" FILES ${VIKING_VIEW_DATA_SRCS} ${VIKING_VIEW_DATA_HDRS})

# this command will generate rules that will run rcc on all files from VIKING_VIEW_RCS
# in result VIKING_VIEW_RC_SRCS variable will contain paths to files produced by rcc
QT5_ADD_RESOURCES( VIKING_VIEW_RC_SRCS ${VIKING_VIEW_RCS} )

# this will run uic on .ui files:
QT5_WRAP_UI( VIKING_VIEW_UI_HDRS ${VIKING_VIEW_APPLICATION_UIS} 
  ${VIKING_VIEW_GROOM_UIS}
  ${VIKING_VIEW_OPTIMIZE_UIS} 
  ${VIKING_VIEW_ANALYSIS_UIS} 
)

# and finally this will run moc:
QT5_WRAP_CPP( VIKING_VIEW_MOC_SRCS ${VIKING_VIEW_APPLICATION_HDRS} ${VIKING_VIEW_MOC_HDRS} )


# Turn on precompiled headers
IF(USE_PRECOMPILED_HEADERS)
  enable_precompiled_headers(PrefixHeader.h VIKING_VIEW_SRCS)
  enable_precompiled_headers(PrefixHeader.h VIKING_VIEW_MOC_SRCS)
ENDIF(USE_PRECOMPILED_HEADERS)


# we need this to be able to include headers produced by uic in our code
# (CMAKE_BINARY_DIR holds a path to the build directory)
INCLUDE_DIRECTORIES( ${CMAKE_BINARY_DIR} )
#INCLUDE_DIRECTORIES( ${CMAKE_CURRENT_BINARY_DIR} )

# For Apple set the icns file containing icons
IF(APPLE)
  # set how it shows up in the Info.plist file
  SET(MACOSX_BUNDLE_ICON_FILE ${CMAKE_SOURCE_DIR}/Resources/VikingView.icns) 
  # set where in the bundle to put the icns file
  SET_SOURCE_FILES_PROPERTIES(${CMAKE_CURRENT_SOURCE_DIR}/Resources/VikingView.icns PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
  # include the icns file in the target
  SET(VIKING_VIEW_SRCS ${VIKING_VIEW_SRCS} ${CMAKE_CURRENT_SOURCE_DIR}/Resources/VikingView.icns)
ENDIF(APPLE)

#IF(WIN32)
#  SET(VIKING_VIEW_SRCS ${VIKING_VIEW_SRCS} ${CMAKE_CURRENT_SOURCE_DIR}/Resources/VikingView.rc)
#ENDIF(WIN32)

if(MSVC)
  SOURCE_GROUP(moc_sources FILES ${VIKING_VIEW_MOC_SRCS})
endif(MSVC)


# here we instruct CMake to build VIKING_VIEW executable from all of the source files
ADD_EXECUTABLE( VikingView MACOSX_BUNDLE WIN32 
  ${VIKING_VIEW_SRCS}
  ${VIKING_VIEW_HDRS}
  ${VIKING_VIEW_MOC_SRCS} 
  ${VIKING_VIEW_RC_SRCS} 
  ${VIKING_VIEW_UI_HDRS} 
  ${VIKING_VIEW_MOC_HDRS}
  )

TARGET_LINK_LIBRARIES( VikingView
  ${QT_LIBRARIES} 
  ${ITK_LIBRARIES}
  ${Boost_THREAD_LIBRARY}
  ${Boost_SYSTEM_LIBRARY}
  ${Boost_FILESYSTEM_LIBRARY}
  ${VTK_LIBRARIES}
  Qt5::Widgets
  Qt5::Core
  Qt5::Network
  Qt5::Script
  Qt5::Concurrent
  Qt5::WinMain
  )


#-- Add an Option to toggle the generation of the API documentation
option(BUILD_DOCUMENTATION "Use Doxygen to create the HTML based API documentation" OFF)
if(BUILD_DOCUMENTATION)
  FIND_PACKAGE(Doxygen)
  if (NOT DOXYGEN_FOUND)
    message(FATAL_ERROR 
      "Doxygen is needed to build the documentation. Please install it correctly")
  endif()
  #-- Configure the Template Doxyfile for our specific project
  configure_file(Doxyfile.in 
                 ${PROJECT_BINARY_DIR}/Doxyfile  @ONLY IMMEDIATE)
  #-- Add a custom target to run Doxygen when ever the project is built
  add_custom_target (docs
    COMMAND ${DOXYGEN_EXECUTABLE} ${PROJECT_BINARY_DIR}/Doxyfile
    SOURCES ${PROJECT_BINARY_DIR}/Doxyfile)
  # IF you do NOT want the documentation to be generated EVERY time you build the project
  # then leave out the 'ALL' keyword from the above command.
endif()




###########################################
# Flags for Microsoft Compiler
###########################################

IF (WIN32 AND MSVC)
  ADD_DEFINITIONS(-D_WIN32_WINNT=0x0501 -DNTDDI_VERSION=0x05010000)
#  ADD_DEFINITIONS(-D_WIN32_WINNT=0x0501)
  ADD_DEFINITIONS(-DPSAPI_VERSION=1)
  # Disable Visual C++ Secure Warnings
  ADD_DEFINITIONS(-D_SCL_SECURE_NO_WARNINGS)
  ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS)
  ADD_DEFINITIONS(-D_BIND_TO_CURRENT_VCLIBS_VERSION=1)
  ADD_DEFINITIONS(-D_BIND_TO_CURRENT_CRT_VERSION=1)
  SET(CMAKE_CXX_FLAGS "/Oi ${CMAKE_CXX_FLAGS} /MP4 /Zm250")
  MESSAGE(STATUS "** WINDOWS OPTIONS")
ENDIF (WIN32 AND MSVC)



# INSTALLATION AND PACKAGING
include(InstallRequiredSystemLibraries)

set(plugin_dest_dir bin)
set(qtconf_dest_dir bin)
set(APPS "\${CMAKE_INSTALL_PREFIX}/bin/VikingView")
if(APPLE)
  set(plugin_dest_dir VikingView.app/Contents/MacOS)
  set(qtconf_dest_dir VikingView.app/Contents/Resources)
  set(contents_dest_dir VikingView.app/Contents)
  set(APPS "\${CMAKE_INSTALL_PREFIX}/VikingView.app")
endif(APPLE)
if(WIN32)
  set(APPS "\${CMAKE_INSTALL_PREFIX}/bin/VikingView.exe")
endif(WIN32)



#--------------------------------------------------------------------------------
# Install the VIKINGVIEW application, on Apple, the bundle is at the root of the
# install tree, and on other platforms it'll go into the bin directory.
install(TARGETS VikingView
    BUNDLE DESTINATION . COMPONENT Runtime
    RUNTIME DESTINATION bin COMPONENT Runtime
    )

#--------------------------------------------------------------------------------
# Install needed Qt plugins by copying directories from the qt installation
# One can cull what gets copied by using 'REGEX "..." EXCLUDE'
#install(DIRECTORY "${QT_PLUGINS_DIR}/imageformats" DESTINATION ${plugin_dest_dir}/plugins COMPONENT Runtime)

#--------------------------------------------------------------------------------
# Install needed Qt .nib files and others
#if(APPLE)
#  install(DIRECTORY "${QT_QTGUI_LIBRARY_RELEASE}/Versions/Current/Resources" DESTINATION ${contents_dest_dir} COMPONENT Runtime)
#endif(APPLE)

#--------------------------------------------------------------------------------
# install a qt.conf file
# this inserts some cmake code into the install script to write the file
install(CODE "
   file(WRITE \"\${CMAKE_INSTALL_PREFIX}/${qtconf_dest_dir}/qt.conf\" \"\")
    " COMPONENT Runtime)



MESSAGE(STATUS "Boost Library Dir " ${Boost_LIBRARY_DIR})

#--------------------------------------------------------------------------------
# Use BundleUtilities to get all other dependencies for the application to work.
# It takes a bundle or executable along with possible plugins and inspects it
# for dependencies.  If they are not system dependencies, they are copied.

# directories to look for dependencies
set(DIRS ${QT_LIBRARY_DIRS} ${CGAL_DIR}/../auxiliary/gmp/lib ${CGAL_DIR}/bin ${BOOST_LIBRARYDIR} ${QT_PLUGINS_DIR})

# Now the work of copying dependencies into the bundle/package
# The quotes are escaped and variables to use at install time have their $ escaped
# An alternative is the do a configure_file() on a script and use install(SCRIPT  ...).
# Note that the image plugins depend on QtSvg and QtXml, and it got those copied
# over.  

# fixup_bundle will run the appropriate install_name_tool
# commands on each of the dependencies of the main executable and the
# plugin libraries.  
#
# install_name_tool is used to point any non-system
# dynamically-loaded libraries to the appropriate libraries in the
# distribution
#
# fixup_bundle also grabs the appropriate parts of the qt framework
# (by following dependencies) and installs them in the distribution
#
IF(APPLE)

install(CODE "
    file(GLOB_RECURSE QTPLUGINS
      \"\${CMAKE_INSTALL_PREFIX}/${plugin_dest_dir}/plugins/*${CMAKE_SHARED_LIBRARY_SUFFIX}\")
    include(BundleUtilities)
    fixup_bundle(\"${APPS}\" \"\${QTPLUGINS}\" \"${DIRS}\")
    " COMPONENT Runtime)

ELSE(APPLE)

  install(CODE "
    file(GLOB_RECURSE QTPLUGINS
      \"\${CMAKE_INSTALL_PREFIX}/${plugin_dest_dir}/plugins/*${CMAKE_SHARED_LIBRARY_SUFFIX}\")
    include(BundleUtilities)
    fixup_bundle(\"${APPS}\" \"\${QTPLUGINS}\" \"${DIRS}\")
    " COMPONENT Runtime)

  install(FILES ${QT_PLUGINS_PLATFORMS_QWINDOWS_DLL} DESTINATION "bin/platforms")
ENDIF(APPLE)
# To Create a package, one can run "cpack -G DragNDrop CPackConfig.cmake" on Mac OS X
# where CPackConfig.cmake is created by including CPack
# And then there's ways to customize this as well
set(CPACK_BINARY_DRAGNDROP ON)

set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "VikingView")
set(CPACK_PACKAGE_VENDOR "University of Utah")
#set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/PackageDescription.txt")
#set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/Copyright.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "0")
set(CPACK_PACKAGE_VERSION_MINOR "5")
set(CPACK_PACKAGE_VERSION_PATCH "2")
set(CPACK_PACKAGE_EXECUTABLES "VikingView" "VikingView")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "VikingView ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")


set(CPACK_BUILD_CONFIG "Release")

include(CPack)
