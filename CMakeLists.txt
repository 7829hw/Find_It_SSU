cmake_minimum_required(VERSION 3.30)

project(find_it_ssu)

# Qt 6 패키지 찾기
find_package(Qt6 REQUIRED COMPONENTS Widgets)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON) # 시그널/슬롯 사용 시
set(CMAKE_AUTORCC ON) # 리소스 파일 사용 시

set(CMAKE_CXX_STANDARD 20)

# MSVC 특화 컴파일 옵션
add_compile_options(
    /W3                 # 경고 레벨 3
    /utf-8             # UTF-8 인코딩
)

# Windows 정의
add_compile_definitions(
    _CRT_SECURE_NO_WARNINGS
    NOMINMAX
    WIN32_LEAN_AND_MEAN
)

# SQLite3 헤더 경로
include_directories(libs/sqlite3)

# Qt 프로젝트를 위한 실행 파일 정의
qt_add_executable(find_it_ssu WIN32
        main.cpp
        database_manager.cpp
        file_manager.cpp
        tag_manager.cpp
        mainwindow.cpp
        MainWindow.ui
        app.rc # 아이콘용
        app_resources.qrc
)

# SQLite3 라이브러리 설정
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(SQLITE3_LIB_PATH "${CMAKE_SOURCE_DIR}/libs/sqlite3/sqlite3_debug.lib")
else()
    set(SQLITE3_LIB_PATH "${CMAKE_SOURCE_DIR}/libs/sqlite3/sqlite3.lib")
endif()

# Qt 모듈과 SQLite3 라이브러리 링크
target_link_libraries(find_it_ssu PRIVATE
        Qt6::Widgets
        ${SQLITE3_LIB_PATH}
)

# Qt DLL 자동 배포를 위한 설정
qt_finalize_executable(find_it_ssu)

# SQLite3 DLL이 있다면 출력 디렉토리로 복사
if(EXISTS "${CMAKE_SOURCE_DIR}/libs/sqlite3/sqlite3.dll")
    add_custom_command(TARGET find_it_ssu POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "${CMAKE_SOURCE_DIR}/libs/sqlite3/sqlite3.dll"
        $<TARGET_FILE_DIR:find_it_ssu>
        COMMENT "SQLite3 DLL 복사 중...")
endif()

# windeployqt 자동 실행 설정
find_program(WINDEPLOYQT_EXECUTABLE windeployqt HINTS ${Qt6_DIR}/../../../bin)

if(WINDEPLOYQT_EXECUTABLE)
    message(STATUS "windeployqt found: ${WINDEPLOYQT_EXECUTABLE}")
    
    add_custom_command(TARGET find_it_ssu POST_BUILD
        COMMAND ${WINDEPLOYQT_EXECUTABLE} --no-translations --no-system-d3d-compiler --no-opengl-sw $<TARGET_FILE:find_it_ssu>
        COMMENT "Qt 라이브러리 자동 배포 중... (windeployqt)")
else()
    message(WARNING "windeployqt not found! Qt DLL을 수동으로 복사해야 합니다.")
    message(STATUS "Qt6_DIR: ${Qt6_DIR}")
    message(STATUS "예상 windeployqt 위치: ${Qt6_DIR}/../../../bin")
endif()

# 디버그 정보 출력
message(STATUS "Compiler: ${CMAKE_CXX_COMPILER_ID}")
message(STATUS "SQLite3 Library Path: ${SQLITE3_LIB_PATH}")
message(STATUS "Qt Version: ${Qt6_VERSION}")
message(STATUS "Build Type: ${CMAKE_BUILD_TYPE}")