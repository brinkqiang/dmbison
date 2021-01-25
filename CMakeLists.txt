cmake_minimum_required(VERSION 3.2)

PROJECT(dmbison)

LIST(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
INCLUDE(cmake/ModuleImport.cmake)
INCLUDE(cmake/ModuleCompileOptions.cmake)
ModuleSetCompileOptions()

SET(DMBISON_VERSION_MAJOR "1")
SET(DMBISON_VERSION_MINOR "0")
SET(DMBISON_VERSION_PATCH "1")
SET(DMBISON_VERSION "${DMBISON_VERSION_MAJOR}.${DMBISON_VERSION_MINOR}.${DMBISON_VERSION_PATCH}")

MESSAGE(STATUS "VERSION: ${DMBISON_VERSION}")

OPTION(USE_DMBISON "use dmbison" OFF)

IF(WIN32)
    add_definitions(-D_LIB)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)

    SET(CommonExcludeList "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common/regexec.c" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common/regcomp.c")

    LibImportExclude("common" "win/dmbison/common" "${CommonExcludeList}")

    target_include_directories(common PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common")

    SET(libbisonExcludeList "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/main.c" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/scan-code.c" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/scan-gram.c" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/scan-skel.c")

    LibImportExclude("libbison" "win/dmbison/src" "${libbisonExcludeList}")

    target_include_directories(libbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison")
    target_include_directories(libbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src")
    target_include_directories(libbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common")

    target_link_libraries(libbison common)

    add_definitions(-D_CONSOLE)
    ExeImport("wintools" "libbison;common")
    target_include_directories(dmbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison")
    target_include_directories(dmbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src")
    target_include_directories(dmbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common")
ELSE(UNIX)

  SET(ExcludeList "${CMAKE_CURRENT_SOURCE_DIR}/src/main.c" "${CMAKE_CURRENT_SOURCE_DIR}/src/scan-code.c" "${CMAKE_CURRENT_SOURCE_DIR}/src/scan-gram.c" "${CMAKE_CURRENT_SOURCE_DIR}/src/scan-skel.c")

  LibImportExclude("libbison" "src" ${ExcludeList})

  target_include_directories(libbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib")
  target_include_directories(libbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/src")

  ExeImport("tools" "libbison")

  ADD_CUSTOM_TARGET(
      dmbison_autogen
      COMMAND ./configure && make
      WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
      COMMENT "Running dmbison configure"
    )

  ADD_DEPENDENCIES(libbison dmbison_autogen)
ENDIF(WIN32)
