########################################################
# Files

SET (SACONN_SRCS 
    sqlanystatement.cpp
    sqlanyconnection.cpp
    sacapi/sacapidll.c
)

IF (WIN32)
  IF (MSVC)
    ADD_DEFINITIONS("-DSACONN_EXPORT=__declspec(dllexport)")
    SET_SOURCE_FILES_PROPERTIES(sacapi/sacapidll.c PROPERTIES COMPILE_FLAGS "-wd4152 -wd4055")
  ELSE (MSVC)
    ADD_DEFINITIONS("-USACONN_EXPORT \"-DSACONN_EXPORT=__declspec(dllexport)\"")
  ENDIF (MSVC)
ELSE (WIN32)
  ADD_DEFINITIONS(-DSACONN_EXPORT=)
ENDIF (WIN32)

########################################################
# Build

INCLUDE_DIRECTORIES(
     ../../../core 
)

ADD_LIBRARY (qgissqlanyconnection SHARED ${SACONN_SRCS})

SET_TARGET_PROPERTIES(qgissqlanyconnection PROPERTIES
  CLEAN_DIRECT_OUTPUT 1
  FRAMEWORK 1
  FRAMEWORK_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}"
  MACOSX_FRAMEWORK_INFO_PLIST "${CMAKE_SOURCE_DIR}/mac/framework.info.plist.in"
  MACOSX_FRAMEWORK_SHORT_VERSION_STRING ${COMPLETE_VERSION}
  MACOSX_FRAMEWORK_IDENTIFIER org.qgis.qgis2_sqlanyconnection)

IF (APPLE)
  SET_TARGET_PROPERTIES(qgissqlanyconnection PROPERTIES
    LINK_FLAGS "${CMAKE_SHARED_LINKER_FLAGS}"
    )
ENDIF (APPLE)

#generate unversioned libs for android
IF (NOT ANDROID)
  SET_TARGET_PROPERTIES(qgissqlanyconnection PROPERTIES
    VERSION ${COMPLETE_VERSION}
    SOVERSION ${COMPLETE_VERSION}
    )
ENDIF (NOT ANDROID)

TARGET_LINK_LIBRARIES(qgissqlanyconnection
  qgis_core
)

IF(UNIX AND NOT ANDROID)
  TARGET_LINK_LIBRARIES(qgissqlanyconnection ${CMAKE_DL_LIBS})
ENDIF(UNIX AND NOT ANDROID)

IF (APPLE)
  SET_TARGET_PROPERTIES(qgissqlanyconnection PROPERTIES BUILD_WITH_INSTALL_RPATH TRUE)
ENDIF (APPLE)

########################################################
# Install

IF (WIN32)
    INSTALL(TARGETS qgissqlanyconnection
      RUNTIME DESTINATION ${QGIS_PLUGIN_DIR}
      LIBRARY DESTINATION ${QGIS_PLUGIN_DIR}
      )
ELSE (WIN32)
    INSTALL(TARGETS qgissqlanyconnection
      RUNTIME DESTINATION ${QGIS_PLUGIN_DIR}
      LIBRARY DESTINATION ${QGIS_LIB_DIR}
      FRAMEWORK DESTINATION ${QGIS_FW_SUBDIR}
      )
ENDIF (WIN32)

