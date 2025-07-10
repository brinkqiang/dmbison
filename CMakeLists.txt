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

   add_library(bison_src ${bison_src})

   ExeImport("tools" "libbison.a;liby.a;bison_src")

   target_include_directories(dmbison BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib" "${CMAKE_CURRENT_SOURCE_DIR}/src")
   target_include_directories(bison_src BEFORE PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lib" "${CMAKE_CURRENT_SOURCE_DIR}/src")

   ModuleConfigure(bison_src ${CMAKE_CURRENT_SOURCE_DIR})


    # 1. 引入 ExternalProject 模块
    include(ExternalProject)

    # 2. 定义 libtextstyle 的安装目录，这会在你的构建目录中创建
    set(LIBTEXTSTYLE_INSTALL_DIR ${CMAKE_BINARY_DIR}/libtextstyle_install)

    # 3. 配置 ExternalProject 来下载、编译和安装 gettext
    ExternalProject_Add(
        libtextstyle_external         # 为这个外部项目步骤起一个唯一的名字
        
        # Git 仓库信息
        GIT_REPOSITORY https://git.savannah.gnu.org/git/gettext.git
        GIT_TAG        v0.22.5         # 指定一个稳定的版本号以确保可重复构建
        
        # 配置步骤：运行 Autotools 的 configure 脚本
        # --prefix: 指定安装到我们设定的目录
        # --disable-shared: 只构建静态库 (.a)，这通常更容易集成
        # --without-libxml2: 避免不必要的依赖
        CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-shared --without-libxml2
        
        # 构建步骤：运行 make (使用 -j 并行编译以加快速度)
        BUILD_COMMAND     make -j4
        
        # 安装步骤：运行 make install
        INSTALL_COMMAND   make install
        
        # 指定我们之前定义的安装目录
        INSTALL_DIR       ${LIBTEXTSTYLE_INSTALL_DIR}
    )

    # 4. 创建一个 IMPORTED 目标来代表我们刚刚编译好的 libtextstyle
    #    这是使用 ExternalProject 的关键步骤

    # 首先，声明一个依赖关系，确保在编译 my_app 之前，libtextstyle_external 已经完成
    add_dependencies(bison_src libtextstyle_external)

    # 创建一个 CMake 目标来“指向”编译好的静态库文件
    add_library(libtextstyle_imported STATIC IMPORTED GLOBAL) # GLOBAL 使得在子目录也可见

    # 设置这个目标的属性：库文件的位置和头文件的位置
    set_target_properties(libtextstyle_imported PROPERTIES
        IMPORTED_LOCATION         "${LIBTEXTSTYLE_INSTALL_DIR}/lib/libtextstyle.a"
        INTERFACE_INCLUDE_DIRECTORIES "${LIBTEXTSTYLE_INSTALL_DIR}/include"
    )


    # 5. 将你的程序链接到这个新创建的 IMPORTED 目标上
    target_link_libraries(bison_src PRIVATE libtextstyle_imported)
endif()
