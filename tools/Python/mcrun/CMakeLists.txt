cmake_minimum_required(VERSION 3.17.0)
cmake_policy(VERSION 3.17.0)

# This will install McStas or McXtrace mc/mxrun
project(python-tools-mcrun C)

message(STATUS "Configure ${CMAKE_PROJECT_NAME} ${FLAVOR} ${PROJECT_NAME}")

# Choose between McStas or McXtrace
option(BUILD_MCSTAS   "Choose the McStas flavor" off)
option(BUILD_MCXTRACE "Choose the McXtrace flavor (trumps BUILD_MCSTAS)" Off)

# Set module path
set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH}" "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules/")

# Set McCode values (from mkdist or defaults)
include(MCUtil)

if (${BUILD_MCXTRACE})
  set(FLAVOR "mcxtrace")
  set(CPACK_PACKAGE_HOMEPAGE_URL "http://mcxtrace.org/")
elseif(${BUILD_MCSTAS})
  set(FLAVOR "mcstas")
  set(CPACK_PACKAGE_HOMEPAGE_URL "http://mcstas.org/")
else()
  message(FATAL_ERROR "No flavor specified!\n"
    "Please use either -DBUILD_MCSTAS=1 or -DBUILD_MCXTRACE=1")
endif()

setupMCCODE("${FLAVOR}")

# Select prefix
set(P "${MCCODE_PREFIX}")

set(TOOLS_NAME "tools/Python/${P}run/")

# CPack configuration
set(CPACK_PACKAGE_NAME          "${FLAVOR}-tools-python-${P}run-${MCCODE_VERSION}")
set(CPACK_RESOURCE_FilE_LICENSE "${PROJECT_SOURCE_DIR}/../../COPYING")
set(CPACK_PACKAGE_CONTACT       "jsbn@fysik.dtu.dk")

set(CPACK_PACKAGE_VERSION       "${MCCODE_VERSION}")
set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "0")


# NSIS
set(NSIS_NAME "Python-based mcrun (mcrun-py), ${MCCODE_STRING}")
set(CPACK_NSIS_PACKAGE_NAME "${NSIS_NAME}")
set(CPACK_NSIS_DISPLAY_NAME "${NSIS_NAME}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "${NSIS_NAME}")

# Debian
set(CPACK_DEBIAN_PACKAGE_DEPENDS "${FLAVOR}-${MCCODE_VERSION}, python3-yaml")

# RPM
if (RPMPROFILE)
   if (RPMPROFILE STREQUAL "fedora")
        set(PYDEPS "python3-pyyaml")
   else()
	set(PYDEPS ${FLAVOR}-miniconda3-${MCCODE_VERSION})
   endif()
else()
	set(PYDEPS ${FLAVOR}-miniconda3-${MCCODE_VERSION})
endif()

set(CPACK_RPM_PACKAGE_REQUIRES
  "${FLAVOR}-${MCCODE_VERSION}, ${PYDEPS}"
)

include(CPack)

set(WORK "${PROJECT_BINARY_DIR}/work")

# Program files
if(WINDOWS)
  set(TOOLS_LIB "${CMAKE_INSTALL_PREFIX}/${lib}/${FLAVOR}/${MCCODE_VERSION}/${TOOLS_NAME}")
else()
  set(TOOLS_LIB "${CMAKE_INSTALL_PREFIX}/${FLAVOR}/${MCCODE_VERSION}/${TOOLS_NAME}")
endif()

# Configure fallback script
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/mcrun.in" "${WORK}/${P}run" @ONLY)

if(WINDOWS)
  install(
    FILES "${PROJECT_SOURCE_DIR}/mcrun.py"
    DESTINATION "${lib}/${TOOLS_NAME}"
    RENAME "${P}run.py"
    PERMISSIONS
    OWNER_READ OWNER_WRITE OWNER_EXECUTE
    GROUP_READ GROUP_EXECUTE
    WORLD_READ WORLD_EXECUTE
  )

  foreach(NAME "__init__.py" "log.py" "mccode.py" "optimisation.py")
    install(
      FILES "${PROJECT_SOURCE_DIR}/${NAME}"
      DESTINATION "${lib}/${TOOLS_NAME}"
      )
  endforeach()
endif()

if(NOT WINDOWS)
  foreach(NAME "__init__.py" "log.py" "mccode.py" "optimisation.py")
    install(
      FILES "${PROJECT_SOURCE_DIR}/${NAME}"
      DESTINATION "${FLAVOR}/${MCCODE_VERSION}/${TOOLS_NAME}"
      )
  endforeach()
  
  add_custom_target(
    "CREATE_${PROJECT_NAME}_SYMLINK" ALL DEPENDS "${WORK}/${P}run"
    )

  install(
    FILES "${WORK}/${P}run"
    DESTINATION "${FLAVOR}/${MCCODE_VERSION}/${TOOLS_NAME}"
    PERMISSIONS
    OWNER_READ OWNER_WRITE OWNER_EXECUTE
    GROUP_READ GROUP_EXECUTE
    WORLD_READ WORLD_EXECUTE
  )
  install(
    FILES "${PROJECT_SOURCE_DIR}/mcrun.py"
    DESTINATION "${FLAVOR}/${MCCODE_VERSION}/${TOOLS_NAME}"
    RENAME "${P}run.py"
    PERMISSIONS
    OWNER_READ OWNER_WRITE OWNER_EXECUTE
    GROUP_READ GROUP_EXECUTE
    WORLD_READ WORLD_EXECUTE
  )

  
  install(
    PROGRAMS "${WORK}/${P}run"
    DESTINATION ${FLAVOR}/${MCCODE_VERSION}/bin
    )

endif()
