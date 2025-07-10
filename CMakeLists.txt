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
    file(GLOB_RECURSE common_src
        "${CMAKE_CURRENT_SOURCE_DIR}/lib/*.c"
    )
    list(FILTER common_src EXCLUDE REGEX "windows-")
    list(FILTER common_src EXCLUDE REGEX "(windows-|msvc-)")
    list(FILTER common_src EXCLUDE REGEX "/lib/main\\.c$")
    list(FILTER common_src EXCLUDE REGEX "/lib/fsync\\.c$")
    list(FILTER common_src EXCLUDE REGEX "/lib/relocwrapper\\.c$")
    list(FILTER common_src EXCLUDE REGEX "/lib/argmatch\\.c$")
    list(FILTER common_src EXCLUDE REGEX "/lib/sigaction\\.c$")

    
    add_library(bison_src ${bison_src} ${common_src} ${missing_common_src})


    # 将生成的解析器文件添加到库中
    target_sources(bison_src PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/parse-gram.c)

    # 查找并链接 ncurses 库
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(NCURSES REQUIRED ncurses)

    ExeImport("tools" "bison_src;${NCURSES_LIBRARIES}")

    target_include_directories(dmbison BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib" "${CMAKE_CURRENT_SOURCE_DIR}/src")
    target_include_directories(bison_src BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib" "${CMAKE_CURRENT_SOURCE_DIR}/src")

    # 添加 ncurses 头文件路径
    target_include_directories(bison_src PUBLIC ${NCURSES_INCLUDE_DIRS})
    target_compile_options(bison_src PUBLIC ${NCURSES_CFLAGS_OTHER})

    ModuleConfigure(bison_src ${CMAKE_CURRENT_SOURCE_DIR})

    # ----------------------------------------------------
    # 从本地源码压缩包构建 libtextstyle
    # ----------------------------------------------------
    include(ExternalProject)

    # 1. 定义压缩包的路径
    set(GETTEXT_ARCHIVE ${CMAKE_CURRENT_SOURCE_DIR}/depends/gettext-0.21.tar.gz)

    # 2. 定义安装目录
    set(LIBTEXTSTYLE_INSTALL_DIR ${CMAKE_BINARY_DIR}/libtextstyle_install)

    ExternalProject_Add(
        libtextstyle_from_archive # 换个新名字，以示区别

        # 核心改动：使用 URL 指向本地压缩包
        URL ${GETTEXT_ARCHIVE}

        # 最佳实践：提供文件的 SHA256 哈希值以确保文件完整性
        # gettext-0.21.tar.gz 的 SHA256 哈希值如下：
        URL_HASH SHA256=C77D0DA3102AEC9C07F43671E60611EBFF89A996EF159497CE8E59D075786B12

        # 配置、构建、安装命令与之前完全相同
        # ExternalProject 会先自动解压，然后在解压后的目录中执行这些命令
        CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-shared --without-libxml2
        BUILD_COMMAND make -j4
        INSTALL_COMMAND make install
        INSTALL_DIR ${LIBTEXTSTYLE_INSTALL_DIR}
    )

    add_library(libtextstyle_imported STATIC IMPORTED GLOBAL)
    set_target_properties(libtextstyle_imported PROPERTIES
        IMPORTED_LOCATION "${LIBTEXTSTYLE_INSTALL_DIR}/lib/libtextstyle.a"
    )

    # 2. 像之前一样，设置构建依赖和链接关系
    add_dependencies(bison_src libtextstyle_from_archive)
    target_link_libraries(bison_src PUBLIC libtextstyle_imported)

    # 链接 ncurses 库
    target_link_libraries(bison_src PUBLIC ${NCURSES_LIBRARIES})

    # 3. ✅ 将头文件目录直接提供给最终使用的目标
    # 因为有 add_dependencies, 编译 bison_src 时, 这个目录必然已经存在
    target_include_directories(bison_src PUBLIC
        ${LIBTEXTSTYLE_INSTALL_DIR}/include
    )
endif()