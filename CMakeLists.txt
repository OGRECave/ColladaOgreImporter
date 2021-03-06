# CMake build control file for ColladaOgreImporter
# by Jeff Trull <jetrull@sbcglobal.net>

#Copyright (c) 2012 Jeffrey E Trull
#
#Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
#
#The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
#
#THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#

# Example build procedure (Ubuntu):
# mkdir build;cd build;cmake -DOGRE_HOME=/usr/share/OGRE -DCOLLADA_ROOT=/home/me/opencollada-read-only/ ..;make
# if you are using a built (but not installed) version of Ogre from source, you can do this:
# cmake -DOGRE_BUILD=/path/to/ogre_src/build -DOGRE_SOURCE=/path/to/ogre_src -DCOLLADA_ROOT=/home/me/opencollada-read-only/ ..
# if you are also using your own built version of Boost (must be the same used to build Ogre):
# cmake -DOGRE_BUILD=/path/to/ogre_src/build -DOGRE_SOURCE=/path/to/ogre_src -DCOLLADA_ROOT=/home/me/opencollada-read-only/ -DBOOST_ROOT=/home/jet/boost_1_53_0 -DBoost_NO_SYSTEM_PATHS=1 ..

# Requires the library OpenCollada;  there doesn't seem to be an Ubuntu package so I do this (from their web page):
# svn checkout http://opencollada.googlecode.com/svn/trunk/ opencollada-read-only
# cd opencollada-read-only;mkdir build;cd build;cmake ..;make

cmake_minimum_required(VERSION 2.8)

enable_testing()

# We need the OpenCollada library.  Users can supply COLLADA_ROOT as a hint
# includes
find_path(COLLADAFW_INCLUDE_DIR COLLADAFWRoot.h HINTS ${COLLADA_ROOT}/COLLADAFramework/include)
find_path(COLLADASAX_INCLUDE_DIR COLLADASaxFWLLoader.h HINTS ${COLLADA_ROOT}/COLLADASaxFrameworkLoader/include)
find_path(COLLADABASE_INCLUDE_DIR COLLADABUPlatform.h HINTS ${COLLADA_ROOT}/COLLADABaseUtils/include)
find_path(COLLADASW_INCLUDE_DIR COLLADASWStreamWriter.h HINTS ${COLLADA_ROOT}/COLLADAStreamWriter/include)
find_path(COLLADA_GENERATED_SAX_INCLUDE_DIR GeneratedSaxParser.h HINTS ${COLLADA_ROOT}/GeneratedSaxParser/include)
# libraries
set( COLLADA_LIB_HINT "${COLLADA_ROOT}/build/lib" )
# For multi-configuration generators you will get an extra dir for the configuration
if( CMAKE_GENERATOR MATCHES "^((Visual Studio)|Xcode)" )
  set( COLLADA_LIB_HINT "${COLLADA_LIB_HINT}/Debug" )     # should really use one per config...
endif()
find_library(COLLADABU_LIB OpenCOLLADABaseUtils HINTS ${COLLADA_LIB_HINT} )
find_library(COLLADAFW_LIB OpenCOLLADAFramework HINTS ${COLLADA_LIB_HINT} )
find_library(COLLADASAX_LIB OpenCOLLADASaxFrameworkLoader HINTS ${COLLADA_LIB_HINT} )
find_library(COLLADASAXP_LIB GeneratedSaxParser HINTS ${COLLADA_LIB_HINT} )
find_library(COLLADASW_LIB OpenCOLLADAStreamWriter HINTS ${COLLADA_LIB_HINT} )
find_library(COLLADABF_LIB buffer HINTS ${COLLADA_LIB_HINT} )
find_library(F2A_LIB ftoa HINTS ${COLLADA_LIB_HINT} )
find_library(UTF_LIB UTF HINTS ${COLLADA_LIB_HINT} )
find_library(MATHML_LIB MathMLSolver ${COLLADA_LIB_HINT} )
# On Windows also get libxml2 and pcre from OpenCOLLADA
if ( CMAKE_HOST_WIN32 )
  find_library( XML2_LIB xml HINTS ${COLLADA_LIB_HINT} )
  find_library( PCRE_LIB pcre HINTS ${COLLADA_LIB_HINT} )
else()
  set( XML2_LIB xml2 )
  set( PCRE_LIB pcre )
endif()

# fail if not found
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(OpenCollada REQUIRED_VARS
                                  COLLADAFW_INCLUDE_DIR COLLADASAX_INCLUDE_DIR
                                  COLLADABASE_INCLUDE_DIR COLLADA_GENERATED_SAX_INCLUDE_DIR
				  COLLADABU_LIB COLLADAFW_LIB COLLADASAX_LIB COLLADASAXP_LIB
				  UTF_LIB MATHML_LIB)

# find Ogre.  Unfortunately FindOGRE.cmake is not included as part of CMake, but as part of Ogre instead,
# so you have to know where Ogre is in order to find it...

if ((DEFINED OGRE_HOME) AND (EXISTS ${OGRE_HOME}/CMake/Packages))
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${OGRE_HOME}/CMake/Packages ${OGRE_HOME}/CMake/Utils)
elseif ((DEFINED OGRE_HOME) AND (EXISTS ${OGRE_HOME}/CMake))
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${OGRE_HOME}/CMake ${OGRE_HOME}/CMake/Utils)
elseif ((DEFINED OGRE_HOME) AND (EXISTS ${OGRE_HOME}/cmake/modules))
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${OGRE_HOME}/cmake/modules ${OGRE_HOME}/CMake/Utils)
elseif((DEFINED $ENV{OGRE_HOME}) AND (EXISTS $ENV{OGRE_HOME}/CMake/Packages))
  # Ogre's cmake script also accepts an environment variable of the same name
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} $ENV{OGRE_HOME}/CMake/Packages)
endif()

if (WIN32)
  set(Boost_USE_STATIC_LIBS ON)
endif()

find_package(Boost REQUIRED filesystem system regex)
# Ogre's find_package appears to clobber ${Boost_LIBRARIES}
set(BOOST_LIB_FILES ${Boost_LIBRARIES})

find_package(OGRE 1.8 REQUIRED)

include_directories(SYSTEM ${COLLADAFW_INCLUDE_DIR} ${COLLADASAX_INCLUDE_DIR} ${COLLADA_GENERATED_SAX_INCLUDE_DIR} ${COLLADABASE_INCLUDE_DIR} ${OGRE_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS})

if (CMAKE_COMPILER_IS_GNUCXX)
  # for unique_ptr, at least, and possibly other things (lambdas, move semantics)
  add_definitions(-std=c++0x -Wall)
endif()

# make a library out of the Collada stuff
add_library(collada_importer OgreMeshWriter.cpp OgreSceneWriter.cpp OgreColladaWriter.cpp OgreColladaSaxLoader.cpp)

target_link_libraries(collada_importer ${COLLADASAX_LIB} ${COLLADASAXP_LIB} ${COLLADAFW_LIB} ${COLLADABU_LIB} ${UTF_LIB} ${XML2_LIB} ${PCRE_LIB} ${MATHML_LIB} )
if (WIN32)
  # libxml2 needs this
  target_link_libraries(collada_importer Ws2_32 )
endif()

# data conversion app
add_executable(c2mesh collada2ogre.cpp)
set(APPLIBS ${OGRE_LIBRARIES} collada_importer ${BOOST_LIB_FILES})
target_link_libraries(c2mesh ${APPLIBS})

# viewer app

add_executable(cview simple_viewer.cpp)
target_link_libraries(cview ${APPLIBS})

# unit tests
add_subdirectory(test) # comment this out if you don't want tests to be built
