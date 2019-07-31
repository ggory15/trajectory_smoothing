CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

INCLUDE(cmake/base.cmake)
INCLUDE(cmake/boost.cmake)
INCLUDE(cmake/eigen.cmake)
INCLUDE(cmake/python.cmake)
INCLUDE(cmake/ide.cmake)
INCLUDE(cmake/apple.cmake)

SET(PROJECT_NAME trajectory)
SET(PROJECT_DESCRIPTION "trajectory with time opt ")
SET(PROJECT_URL "http://github.com/ggory15/trajectory_smoothing")

SET(DOXYGEN_USE_MATHJAX YES)

# Handle APPLE Cmake policy
IF(APPLE)
  APPLY_DEFAULT_APPLE_CONFIGURATION()
ENDIF(APPLE)

# Disable -Werror on Unix for now.
SET(CXX_DISABLE_WERROR True)
SET(CMAKE_VERBOSE_MAKEFILE True)

SETUP_PROJECT()

IF(WIN32)
  SET(LINK copy_if_different)
ELSE(WIN32)
  SET(LINK create_symlink)
ENDIF(WIN32)

# --- OPTIONS ----------------------------------------
OPTION (BUILD_PYTHON_INTERFACE "Build the python binding" OFF)
OPTION (BUILD_UNIT_TESTS "Build the unitary tests" ON)
OPTION (INITIALIZE_WITH_NAN "Initialize Eigen entries with NaN" OFF)
OPTION (EIGEN_RUNTIME_NO_MALLOC "If ON, it can assert in case of runtime allocation" ON)
OPTION (EIGEN_NO_AUTOMATIC_RESIZING "If ON, it forbids automatic resizing of dynamics arrays and matrices" OFF)

IF(INITIALIZE_WITH_NAN)
  MESSAGE(STATUS "Initialize with NaN all the Eigen entries.")
  ADD_DEFINITIONS(-DEIGEN_INITIALIZE_MATRICES_BY_NAN)
ENDIF(INITIALIZE_WITH_NAN)

IF(EIGEN_RUNTIME_NO_MALLOC)
  MESSAGE(STATUS "Option EIGEN_RUNTIME_NO_MALLOC on.")
  ADD_DEFINITIONS(-DEIGEN_RUNTIME_NO_MALLOC)
ENDIF(EIGEN_RUNTIME_NO_MALLOC)

IF(EIGEN_NO_AUTOMATIC_RESIZING)
  MESSAGE(STATUS "Option EIGEN_NO_AUTOMATIC_RESIZING on.")
  ADD_DEFINITIONS(-DEIGEN_NO_AUTOMATIC_RESIZING)
ENDIF(EIGEN_NO_AUTOMATIC_RESIZING)

# ----------------------------------------------------
# --- DEPENDENCIES -----------------------------------
# ----------------------------------------------------
ADD_REQUIRED_DEPENDENCY("eigen3 >= 3.2.0") # Eigen::Ref appeared from 3.2.0

# Fail-safe support for catkin-ized pinocchio:
#  - If catkin-based pinocchio is installed it runs the CFG_EXTRAS to set up the Pinocchio preprocessor directives
#  - If it isn't, nothing happens and the subsequent pkg-config check takes care of everything.
#find_package(catkin QUIET COMPONENTS pinocchio)


SET(BOOST_REQUIERED_COMPONENTS filesystem system)
SET(BOOST_BUILD_COMPONENTS unit_test_framework)
SET(BOOST_OPTIONAL_COMPONENTS "")

IF(BUILD_PYTHON_INTERFACE)
  SET(BOOST_OPTIONAL_COMPONENTS ${BOOST_OPTIONAL_COMPONENTS} python)
  FINDPYTHON()
  INCLUDE_DIRECTORIES(SYSTEM ${PYTHON_INCLUDE_DIRS})
  ADD_REQUIRED_DEPENDENCY("eigenpy >= 1.4.0")
ENDIF(BUILD_PYTHON_INTERFACE)

SET(BOOST_COMPONENTS ${BOOST_REQUIERED_COMPONENTS} ${BOOST_OPTIONAL_COMPONENTS} ${BOOST_BUILD_COMPONENTS})
SEARCH_FOR_BOOST()

# Path to boost headers
INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIRS})

# --- MAIN LIBRARY -------------------------------------------------------------
ADD_SUBDIRECTORY(src)

# --- BINDINGS ----------------------------------------------------------------
IF(BUILD_PYTHON_INTERFACE)
  SET(PYWRAP ${PROJECT_NAME}_pywrap)
  ADD_SUBDIRECTORY(bindings)
ENDIF(BUILD_PYTHON_INTERFACE)

ADD_SUBDIRECTORY(example)
# --- PACKAGING ----------------------------------------------------------------
PKG_CONFIG_APPEND_LIBS(${PROJECT_NAME})


SETUP_PROJECT_FINALIZE()
