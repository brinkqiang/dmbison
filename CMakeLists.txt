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
        src/parse-gram.c
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
    # -- 平台相关文件 --

    # -- spawn (进程创建) 系列函数 --

    set(spawn_exclude_list
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawni.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawnp.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawn_faction_addclose.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawn_faction_adddup2.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawn_faction_addopen.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawn_faction_destroy.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawn_faction_init.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawnattr_destroy.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawnattr_init.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawnattr_setflags.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawnattr_setpgroup.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/lib/spawnattr_setsigmask.c"
    )

    list(REMOVE_ITEM common_src ${spawn_exclude_list})


    list(FILTER common_src EXCLUDE REGEX "(windows-|msvc-|os2-)")

    # -- strerror (错误信息) 系列函数 --
    list(FILTER common_src EXCLUDE REGEX "/strerror") # 匹配 strerror.c, strerror_r.c 等

    # -- signal (信号处理) 系列函数 --
    list(FILTER common_src EXCLUDE REGEX "/(sigaction|sigprocmask)\\.c$")

    # -- 目录操作系列函数 --
    list(FILTER common_src EXCLUDE REGEX "/(opendir|readdir|rewinddir)\\.c$")

    # -- 其他单个的兼容性文件 --
    list(FILTER common_src EXCLUDE REGEX "/(access|at-func|fsync|math|pipe|relocwrapper|waitpid)\\.c$")

    # -- 库内自带的测试主程序 --
    list(FILTER common_src EXCLUDE REGEX "/lib/main\\.c$")



    # ----------------------------------------------------
    # 从本地源码压缩包构建 libtextstyle
    # ----------------------------------------------------
    include(ExternalProject)
    set(LIBTEXTSTYLE_INSTALL_DIR ${CMAKE_BINARY_DIR}/libtextstyle_install)

    ExternalProject_Add(
        libtextstyle_from_archive
        # ... (所有 ExternalProject_Add 的参数不变) ...
        URL ${CMAKE_CURRENT_SOURCE_DIR}/depends/gettext-0.21.tar.gz
        URL_HASH SHA256=C77D0DA3102AEC9C07F43671E60611EBFF89A996EF159497CE8E59D075786B12
        CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-shared --without-libxml2
        BUILD_COMMAND     make -j${CMAKE_BUILD_PARALLEL_LEVEL}
        INSTALL_COMMAND   make install
        INSTALL_DIR       ${LIBTEXTSTYLE_INSTALL_DIR}
    )

    # ✅ 2a. 为 gettext 生成的每一个库文件，创建独立的 IMPORTED 目标
    add_library(libtextstyle_imported STATIC IMPORTED)
    set_target_properties(libtextstyle_imported PROPERTIES
        IMPORTED_LOCATION "${LIBTEXTSTYLE_INSTALL_DIR}/lib/libtextstyle.a"
    )
    add_library(libasprintf_imported STATIC IMPORTED)
    set_target_properties(libasprintf_imported PROPERTIES
        IMPORTED_LOCATION "${LIBTEXTSTYLE_INSTALL_DIR}/lib/libasprintf.a"
    )
    add_library(libgettextpo_imported STATIC IMPORTED)
    set_target_properties(libgettextpo_imported PROPERTIES
        IMPORTED_LOCATION "${LIBTEXTSTYLE_INSTALL_DIR}/lib/libgettextpo.a"
    )

    # ✅ 2b. 创建一个 INTERFACE 库作为“捆绑包”或“代言人”
    add_library(gettext_bundle INTERFACE)

    # ✅ 2c. 将所有依赖信息（头文件、库文件）附加到这个“代言人”上
    target_include_directories(gettext_bundle INTERFACE
        "${LIBTEXTSTYLE_INSTALL_DIR}/include"
    )
    target_link_libraries(gettext_bundle INTERFACE
        libtextstyle_imported
        libasprintf_imported
        libgettextpo_imported
        tinfo # libtextstyle 的下游依赖也在这里封装好
    )
    
    # ✅ 2d. 确保在尝试使用这些库之前，ExternalProject 已经构建完成
    add_dependencies(gettext_bundle libtextstyle_from_archive)


    # ----------------------------------------------------
    # 步骤 3: 创建 bison_src 库，并以最清晰的方式链接依赖
    # ----------------------------------------------------
    add_library(bison_src ${bison_src} ${common_src})

    target_include_directories(bison_src BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib" "${CMAKE_CURRENT_SOURCE_DIR}/src")

    ModuleConfigure(bison_src ${CMAKE_CURRENT_SOURCE_DIR})

    # ✅ 现在，bison_src 只需要链接到一个目标，所有依赖都会被自动传递
    target_link_libraries(bison_src PUBLIC gettext_bundle)

    # 为 bison_src 添加它自己需要的头文件路径
    target_include_directories(bison_src PRIVATE
        "${CMAKE_CURRENT_SOURCE_DIR}/lib"
        "${CMAKE_CURRENT_SOURCE_DIR}/src"
    )

    # ----------------------------------------------------
    # 步骤 4: 创建最终的可执行文件
    # ----------------------------------------------------
    # 移除你自定义的 ExeImport, 使用标准的 add_executable
    add_executable(dmbison src/main.c)

    # dmbison 只需要链接到 bison_src，就会通过传递性依赖自动获得所有需要的库
    target_link_libraries(dmbison PRIVATE bison_src)
endif()
