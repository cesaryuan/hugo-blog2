---
title: CMake打包NSIS的模板
slug: cmake-package-nsis-template-z2o1kkv
url: /post/cmake-package-nsis-template-z2o1kkv.html
date: '2022-12-13 11:42:57'
lastmod: '2023-10-10 21:17:08'
toc: true
tags:
  - cmake
  - cpp
keywords: cmake,cpp
isCJKLanguage: true
---

用于打包和处理 MinGW 运行时和 Linked Shared Library DLL

```cmake
# 在Target Build完成后，复制显式通过target_link_libraries链接的库的dll文件到可执行文件的目录下
# 而不会复制其他的，比如复制MinGW的运行时dll文件
macro(CopyTargetRuntimeDlls target)
    if(WIN32)
        add_custom_command(
                TARGET opencv_test
                POST_BUILD COMMAND
                ${CMAKE_COMMAND} -E copy_if_different
                $<TARGET_RUNTIME_DLLS:opencv_test>
                $<TARGET_FILE_DIR:opencv_test>)
    endif()
    message(STATUS "Looking for deps in ${CMAKE_SYSTEM_LIBRARY_PATH};${CMAKE_MINGW_SYSTEM_LIBRARY_PATH}")
    file(GET_RUNTIME_DEPENDENCIES
        RESOLVED_DEPENDENCIES_VAR deps_resolved
        UNRESOLVED_DEPENDENCIES_VAR deps_unresolved
        LIBRARIES $<TARGET_FILE:mylib>
        DIRECTORIES ${CMAKE_SYSTEM_LIBRARY_PATH} ${CMAKE_MINGW_SYSTEM_LIBRARY_PATH}
        PRE_EXCLUDE_REGEXES "api-ms-*" "ext-ms-*"
        POST_EXCLUDE_REGEXES ".*system32/.*\\.dll"
        )
    message(STATUS "Resolving runtime dependencies for $<TARGET_FILE:mylib>")
    foreach(dep ${deps_resolved})
        file(INSTALL ${dep} DESTINATION ${CMAKE_INSTALL_PREFIX})
    endforeach()
    foreach(dep ${deps_unresolved})
        message(WARNING "Runtime dependency ${dep} could not be resolved.")
    endforeach()
endmacro()
CopyTargetRuntimeDlls(opencv_test)

# 在运行cmake --install的时候用到的配置
# 使能复制MinGW的运行时dll文件到CMAKE_INSTALL_PREFIX指定的目录下
macro(InstallConfig target)
    if(MINGW)
        get_filename_component(CMAKE_MINGW_SYSTEM_LIBRARY_PATH "${CMAKE_CXX_COMPILER}" DIRECTORY)
    endif()
    set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR})
    # 安装 opencv_test.exe
    install(TARGETS ${target}
            RUNTIME_DEPENDENCY_SET _RUNTIME_SET
            RUNTIME DESTINATION .)
    # 安装上一步依赖的运行时，主要是 MinGW 的运行时
    install(RUNTIME_DEPENDENCY_SET _RUNTIME_SET
            DESTINATION .
            # 在下面列出的文件夹中寻找_RUNTIME_SET中的文件(DLL on Windows)
            DIRECTORIES ${CMAKE_SYSTEM_LIBRARY_PATH} ${CMAKE_MINGW_SYSTEM_LIBRARY_PATH}
            PRE_EXCLUDE_REGEXES "api-ms-*" "ext-ms-*"
            POST_EXCLUDE_REGEXES ".*system32/.*\\.dll")
endmacro()
# 打包配置，运行cpack时使用
macro(CPackConfig target)
    # 设置NSIS不生成第一页的用户须知界面
    set(CPACK_NSIS_IGNORE_LICENSE_PAGE ON)
    # 开启安装程序DPI感知，避免在高DPI下显示模糊
    set(CPACK_NSIS_MANIFEST_DPI_AWARE ON)
    # 设置 NSIS Installer 的默认安装路径
    # is under this root dir. The full directory presented to the end user is:
    # ${CPACK_NSIS_INSTALL_ROOT}/${CPACK_PACKAGE_INSTALL_DIRECTORY}
    set(CPACK_NSIS_INSTALL_ROOT "$PROGRAMFILES64")
    # 设置打包产物(如NSIS Installer的文件名)
    set(CPACK_PACKAGE_FILE_NAME ${CMAKE_PROJECT_NAME}-${CMAKE_PROJECT_VERSION})
    # 设置默认的安装目录名
    set(CPACK_PACKAGE_INSTALL_DIRECTORY ${CMAKE_PROJECT_NAME})
    include(CPack)
    add_custom_target(
        PackageZIP COMMAND cpack -C Release -G ZIP
        DEPENDS ${target}
    )
endmacro()
InstallConfig(opencv_test)
CPackConfig(opencv_test)
```
