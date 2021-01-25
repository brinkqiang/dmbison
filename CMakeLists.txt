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

OPTION(USE_DMFLEX "use dmflex" OFF)

IF(WIN32)
    add_definitions(-D_LIB)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)

    SET(CommonExcludeList "${CMAKE_CURRENT_SOURCE_DIR}/win/dmflex/common/regexec.c" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmflex/common/regcomp.c")

    LibImportExclude("common" "win/dmflex/common" "${CommonExcludeList}")

    target_include_directories(common PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmflex/common")

    SET(libflexExcludeList "${CMAKE_CURRENT_SOURCE_DIR}/win/dmflex/src/main.c" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmflex/src/libmain.c" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmflex/src/libyywrap.c")

    LibImportExclude("libflex" "win/dmflex/src" "${libflexExcludeList}")

    target_include_directories(libflex PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmflex/src")
    target_link_libraries(libflex common)

    add_definitions(-D_CONSOLE)
    ExeImport("wintools" "libflex")

    target_include_directories(dmflex PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmflex/src")
    target_include_directories(dmflex PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmflex/common")
ELSE(UNIX)

  SET(ExcludeList "${CMAKE_CURRENT_SOURCE_DIR}/src/main.c" "${CMAKE_CURRENT_SOURCE_DIR}/src/main.cc" "${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp")

  LibImportExclude("libbison" "src" ${ExcludeList})

  ExeImport("tools" "libbison")

  ADD_CUSTOM_TARGET(
      dmbison_autogen
      COMMAND ./configure && make
      WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
      COMMENT "Running dmbison configure"
    )

  ADD_DEPENDENCIES(libbison dmbison_autogen)
ENDIF(WIN32)
