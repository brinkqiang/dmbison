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

    SET(libcommonList 
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/allocator.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/areadlink.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/argmatch.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/gl_array_list.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/basename-lgpl.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/binary-io.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/bitrotate.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/bitset.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/bitset/array.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/bitset/stats.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/bitset/table.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/bitset/list.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/bitset/vector.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/bitsetv.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/c-strcasecmp.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/c-strncasecmp.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/careadlinkat.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/cloexec.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/close-stream.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/closeout.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/concat-filename.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/dirname.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/basename.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/dirname-lgpl.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/stripslash.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/exitfail.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/fatal-signal.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/fd-safer-flag.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/dup-safer-flag.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/fopen-safer.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/fstrcmp.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/gethrxtime.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/getprogname.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/gettime.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/hard-locale.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/hash.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/gl_linked_list.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/localcharset.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/glthread/lock.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/math.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/mbchar.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/mbfile.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/mbswidth.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/pipe2.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/pipe2-safer.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/printf-frexp.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/printf-frexpl.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/progname.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/quotearg.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/gl_rbtree_oset.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/gl_rbtreehash_list.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/setlocale_null.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/sig-handler.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/spawn-pipe.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/glthread/threadlib.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/timespec.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/timevar.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/glthread/tls.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/unicodeio.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/unistd.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/dup-safer.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/fd-safer.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/pipe-safer.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/unistr/u8-mbtoucr.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/unistr/u8-uctomb.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/unistr/u8-uctomb-aux.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/uniwidth/width.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/wait-process.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/wctype-h.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/xmalloc.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/xalloc-die.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/xconcat-filename.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/xhash.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/gl_xlist.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/xmemdup0.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/xreadlink.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/xsize.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/xstrndup.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/path-join.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/fcntl.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/mbrtowc.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/obstack.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/printf-args.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/printf-parse.c
      ${CMAKE_CURRENT_SOURCE_DIR}/lib/vasnprintf.c)

  SET(libbisonList 
      ${CMAKE_CURRENT_SOURCE_DIR}/src/AnnotationList.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/InadequacyList.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/Sbitset.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/assoc.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/closure.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/complain.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/conflicts.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/counterexample.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/derivation.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/derives.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/files.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/fixits.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/getargs.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/glyphs.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/gram.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/graphviz.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/ielr.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/lalr.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/location.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/lr0.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/lssi.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/main.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/muscle-tab.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/named-ref.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/nullable.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/output.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/parse-gram.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/parse-simulation.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/print-graph.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/print-xml.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/print.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/reader.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/reduce.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/relation.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/scan-ccde-c.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/scan-gram-c.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/scan-skel-c.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/state.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/state-item.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/strversicn.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/symlist.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/symtab.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/tables.c
      ${CMAKE_CURRENT_SOURCE_DIR}/src/uniqstr.c)

  SET(bisonList ${CMAKE_CURRENT_SOURCE_DIR}/src/main.c)

  
  ADD_LIBRARY(libcommon STATIC ${libcommonList})
  target_include_directories(libcommon PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib")

  ADD_LIBRARY(libbison STATIC ${libbisonList})
  target_include_directories(libbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib")
  target_include_directories(libbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/src")
  TARGET_LINK_LIBRARIES(libbison libcommon)

  ADD_EXECUTABLE(dmbison ${bisonList})

  TARGET_LINK_LIBRARIES(dmbison libbison)

  target_include_directories(dmbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib")
  target_include_directories(dmbison PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/src")

  ADD_CUSTOM_TARGET(
      dmbison_autogen
      COMMAND ./configure && make
      WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
      COMMENT "Running dmbison configure"
    )

  ADD_DEPENDENCIES(libcommon dmbison_autogen)
ENDIF(WIN32)
