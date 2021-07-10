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
    add_definitions(-DYY_NO_UNISTD_H)
    SET(CommonExcludeList "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common/regexec.c" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common/regcomp.c")

    LibImportExclude("common" "win/dmbison/common" "${CommonExcludeList}")

    target_include_directories(common PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common")

    SET(libbisonExcludeList "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/main.c$" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/scan-code.c$" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/scan-gram.c$" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/scan-skel.c$")

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
   set(bison_src
        src/AnnotationList.c
        src/AnnotationList.h
        src/InadequacyList.c
        src/InadequacyList.h
        src/Sbitset.c 
        src/Sbitset.h 
        src/assoc.c 
        src/assoc.h 
        src/closure.c 
        src/closure.h 
        src/complain.c
        src/complain.h
        src/conflicts.c 
        src/conflicts.h 
        src/counterexample.c
        src/counterexample.h
        src/derivation.c
        src/derivation.h
        src/derives.c 
        src/derives.h 
        src/files.c 
        src/files.h 
        src/fixits.c
        src/fixits.h
        src/flex-scanner.h
        src/getargs.c 
        src/getargs.h 
        src/glyphs.c
        src/glyphs.h
        src/gram.c
        src/gram.h
        src/graphviz.c
        src/graphviz.h
        src/ielr.c
        src/ielr.h
        src/lalr.c
        src/lalr.h
        src/location.c
        src/location.h
        src/lr0.c 
        src/lr0.h 
        src/lssi.c
        src/lssi.h
        src/main.c
        src/muscle-tab.c
        src/muscle-tab.h
        src/named-ref.c 
        src/named-ref.h 
        src/nullable.c
        src/nullable.h
        src/output.c
        src/output.h
        src/parse-gram.y
        src/parse-simulation.c
        src/parse-simulation.h
        src/print-graph.c 
        src/print-graph.h 
        src/print-xml.c 
        src/print-xml.h 
        src/print.c 
        src/print.h 
        src/reader.c
        src/reader.h
        src/reduce.c
        src/reduce.h
        src/relation.c
        src/relation.h
        src/scan-code-c.c 
        src/scan-code.h 
        src/scan-gram-c.c 
        src/scan-gram.h 
        src/scan-skel-c.c 
        src/scan-skel.h 
        src/state.c 
        src/state.h 
        src/state-item.c
        src/state-item.h
        src/strversion.c
        src/strversion.h
        src/symlist.c 
        src/symlist.h 
        src/symtab.c
        src/symtab.h
        src/system.h
        src/tables.c
        src/tables.h
        src/uniqstr.c 
        src/uniqstr.h
   )

   add_library(bison_src ${bison_src})

   ExeImport("tools" "libbison.a;liby.a;bison_src")

   target_include_directories(dmbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib" "${CMAKE_CURRENT_SOURCE_DIR}/src")
   target_include_directories(bison_src PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib" "${CMAKE_CURRENT_SOURCE_DIR}/src")

   add_custom_target(dmbison_autogen
   COMMAND ./configure
   COMMAND make
   COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/lib/libbison.a ${LIBRARY_OUTPUT_PATH}
   COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/lib/liby.a ${LIBRARY_OUTPUT_PATH}
   COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/src/bison ${EXECUTABLE_OUTPUT_PATH}
   WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
   COMMENT "Running dmbison configure"
   )

   ADD_DEPENDENCIES(dmbison dmbison_autogen)

ENDIF(WIN32)
