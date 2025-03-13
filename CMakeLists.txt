set(Qt_DIR "usr/local/Qt/Qt5.14.2/5.14.2/gcc_64/lib/cmake/Qt5" )

cmake_minimum_required(VERSION 3.14)
project(hplayer)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_C_STANDARD 99)

set(SDL2_DIR /usr/local/sdl/sdl2/lib/cmake/SDL2)
set(Qt5_DIR /usr/local/Qt/Qt5.14.2/5.14.2/gcc_64/lib/cmake/Qt5)

# 导入Qt5 组件
find_package(Qt5 REQUIRED COMPONENTS Core Gui Widgets)
find_package(OpenGL REQUIRED)
find_package(SDL2 REQUIRED)

# 主要项目设置
add_executable(${PROJECT_NAME}
    src/main.cpp
    # UI模块
    src/ui/htable.cpp
    src/ui/MainWindow.cpp
    src/ui/CentralWidget.cpp
    src/ui/HMultiView.cpp
    src/ui/HVideoWidget.cpp
    src/ui/HVideoTitlebar.cpp
    src/ui/HVideoToolbar.cpp
    src/ui/HVideoWnd.cpp
    src/ui/HOpenMediaDlg.cpp
    src/ui/HMediaInfoDlg.cpp
    src/ui/LsideWidget.cpp
    src/ui/RsideWidget.cpp
    src/ui/GLWnd.cpp
    # Qt相关模块
    src/qt/qtrcloader.cpp
    src/qt/HGLWidget.cpp
    # HV 基础模块
    src/hv/base/hversion.c
    src/hv/base/hbase.c
    src/hv/base/herr.c
    src/hv/base/htime.c
    src/hv/base/hlog.c
    src/hv/base/hstring.cpp
    src/hv/utils/iniparser.cpp
    # GL模块（包含glew源码）
    src/GL/glew.c
    # 视频模块
    src/video/hffplayer.cpp
    # 工具模块
    src/util/hframe.cpp
    # Win32设备驱动
    src/win32/hdevice.cpp
    # SDL窗口（如果启用）
    src/ui/SDL2Wnd.cpp
)

# 头文件包含路径
target_include_directories(${PROJECT_NAME} PRIVATE
    src
    src/hv
    src/hv/base
    src/hv/utils
    src/qt
    src/ui
    src/GL
    src/video
    src/util
    src/win32
    src/include # 第三方库头文件路径
)

# 接口定义
target_compile_definitions(${PROJECT_NAME} PRIVATE
    HV_SOURCE
    GLEW_STATIC
    QT_DEPRECATED_WARNINGS
    UNICODE
    _UNICODE # 注意原.pro移除了这两个定义，这里请根据实际需求调整
)

# 第三方库链接
target_link_libraries(${PROJECT_NAME} PRIVATE 
    Qt5::Core Qt5::Gui Qt5::Widgets
    # FFmpeg相关动态链接库
    avformat avdevice avcodec swresample swscale avutil
    # Windows系统库（仅Win）
    ${CMAKE_DL_LIBS}
    version kernel32 user32 gdi32 opengl32 glu32 ole32 oleaut32 ws2_32 secur32 bcrypt
    # Linux/GNU相关（根据平台自动选择）
    X11 GL GLU pthread dl
    # 其他需要的库：

    # 如果使用MinGW需要额外链接的：
    # iconv bz2 lzma crypto

    # 若使用SDL2
    SDL2
    # 其他动态库请在此处添加
)

if(DEFINED CMAKE_DL_LIBS)
    target_link_libraries(${PROJECT_NAME} PRIVATE ${CMAKE_DL_LIBS})
endif()
    
# 额外链接Linux专用库
target_link_libraries(${PROJECT_NAME} PRIVATE X11 GL GLU pthread dl SDL2 SDL2main)


# 资源文件处理
set(QRC_FILE "rc/skin.qrc" "rc/image.qrc")
qt5_add_resources(RESOURCES_RC ${QRC_FILE})

# 译文文件处理
set(TS_FILES "rc/lang/app_zh_CN.ts")
# qt5_create_translation(TRANSLATION_FILES ${TS_FILES})

# 添加资源和译文到目标
target_sources(${PROJECT_NAME} PRIVATE ${RESOURCES_RC} ${TRANSLATION_FILES})

# 配置后处理（可选）  
if(WIN32)
    # 设置输出路径
    if(CMAKE_CL_64)
        set(BIN_DIR "bin/msvc2015_x64")
    else()
        set(BIN_DIR "bin/msvc2015_x86")
    endif()
else()
    set(BIN_DIR "bin/${CMAKE_SYSTEM_NAME}")
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${BIN_DIR}
)