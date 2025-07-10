cmake_minimum_required(VERSION 3.21)

PROJECT(dmbison)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
include(cmake/ModuleImport.cmake)
include(cmake/ModuleCompileOptions.cmake)
ModuleSetCompileOptions()

set(DMBISON_VERSION_MAJOR "1")
set(DMBISON_VERSION_MINOR "0")
set(DMBISON_VERSION_PATCH "1")
set(DMBISON_VERSION "${DMBISON_VERSION_MAJOR}.${DMBISON_VERSION_MINOR}.${DMBISON_VERSION_PATCH}")

message(STATUS "VERSION: ${DMBISON_VERSION}")

option(USE_DMBISON "use dmbison" OFF)

if(WIN32)
    add_definitions(-D_LIB)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    add_definitions(-DYY_NO_UNISTD_H)
    set(CommonExcludeList "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common/regexec.c" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common/regcomp.c")

    LibImportExclude("common" "win/dmbison/common" "${CommonExcludeList}")

    target_include_directories(common BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common")

    set(libbisonExcludeList "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/main.c$" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/scan-code.c$" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/scan-gram.c$" "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src/scan-skel.c$")

    LibImportExclude("libbison" "win/dmbison/src" "${libbisonExcludeList}")

    target_include_directories(libbison BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison")
    target_include_directories(libbison BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src")
    target_include_directories(libbison BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common")

    target_link_libraries(libbison common)

    add_definitions(-D_CONSOLE)
    ExeImport("wintools" "libbison;common")
    target_include_directories(dmbison BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison")
    target_include_directories(dmbison BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/src")
    target_include_directories(dmbison BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/win/dmbison/common")
else(UNIX)

    include(ExternalProject)

    # 定义源码路径和安装路径
    # 核心改动在这里，指向您的新目录
    set(GETTEXT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/depends/gettext) 
    set(LIBTEXTSTYLE_INSTALL_DIR ${CMAKE_BINARY_DIR}/libtextstyle_install)

    ExternalProject_Add(
        libtextstyle_local_build
        
        # 指定本地源码目录
        SOURCE_DIR        ${GETTEXT_SOURCE_DIR}
        
        # 配置、构建、安装命令保持不变
        CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-shared --without-libxml2
        BUILD_COMMAND     ./autogen.sh && make -j4
        INSTALL_COMMAND   make install
        INSTALL_DIR       ${LIBTEXTSTYLE_INSTALL_DIR}
        
        # 告诉 ExternalProject 这是本地源码，不需要下载
        DOWNLOAD_COMMAND ""
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
    )

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

   target_include_directories(dmbison BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib" "${CMAKE_CURRENT_SOURCE_DIR}/src")
   target_include_directories(bison_src BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib" "${CMAKE_CURRENT_SOURCE_DIR}/src")

   ModuleConfigure(bison_src ${CMAKE_CURRENT_SOURCE_DIR})

   add_dependencies(bison_src libtextstyle_local_build)

endif()
