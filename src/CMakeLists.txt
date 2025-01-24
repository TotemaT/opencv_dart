# The Flutter tooling requires that developers have CMake 3.10 or later
# installed. You should not increase this version, as doing so will cause
# the plugin to fail to compile for some customers of the plugin.
cmake_minimum_required(VERSION 3.10)

project(opencv_dart_library VERSION 0.0.1 LANGUAGES C CXX)
set(library_name "opencv_dart")

set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_INSTALL_PREFIX "install")
if (NOT DEFINED OPENCV_DART_ARCH)
    set(OPENCV_DART_ARCH "x64")
endif ()
#if (WIN32)
#    set(CMAKE_CXX_FLAGS "/std:c++20")
#elseif (UNIX)
#    set(CMAKE_CXX_FLAGS "-Wa,-mbig-obj")
#endif ()
if (NOT DEFINED OpenCV_BUILD_DIR)
    set(OpenCV_BUILD_DIR "${CMAKE_BINARY_DIR}/../../../opencv")
endif ()

message(STATUS "OpenCV_BUILD_DIR: ${OpenCV_BUILD_DIR}")

set(ARUCO_ON "aruco")
if (WIN32)
    if (${CMAKE_GENERATOR_PLATFORM} STREQUAL "Win32")
        set(OpenCV_DIR "${OpenCV_BUILD_DIR}/windows/x86")
    else ()
        set(OpenCV_DIR "${OpenCV_BUILD_DIR}/windows/x64")
    endif ()
elseif (APPLE)
    set(OpenCV_DIR "${OpenCV_BUILD_DIR}/macos/${OPENCV_DART_ARCH}")
elseif (ANDROID)
    set(ARUCO_ON "")
    set(OpenCV_DIR "${OpenCV_BUILD_DIR}/android/sdk/native/jni")
elseif (UNIX)
    set(OpenCV_DIR "${OpenCV_BUILD_DIR}/linux/${OPENCV_DART_ARCH}")
endif ()

message(STATUS "OpenCV_DIR: ${OpenCV_DIR}")
set(OpenCV_STATIC ON)
#find_package(OpenCV 4.9.0 REQUIRED)
find_package(OpenCV REQUIRED
    COMPONENTS
    ${ARUCO_ON} core calib3d dnn highgui features2d photo imgproc 
    objdetect video videoio stitching
)
find_package(Eigen3)
# find_package(OpenEXR)
# find_package(openjpeg-2.5)


message(STATUS "OpenCV library status:")
message(STATUS "    config: ${OpenCV_DIR}")
message(STATUS "    version: ${OpenCV_VERSION}")
message(STATUS "    libraries: ${OpenCV_LIBS}")
message(STATUS "    include path: ${OpenCV_INCLUDE_DIRS}")

#add_library(cv SHARED IMPORTED GLOBAL)
#set_property(TARGET cv PROPERTY IMPORTED)

file(GLOB GOCV_FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/gocv/*.cpp"
    # "${CMAKE_CURRENT_SOURCE_DIR}/gocv/contrib/*.cpp"
)
add_library(${library_name} SHARED ${GOCV_FILES}
)
target_include_directories(${library_name} SYSTEM PUBLIC
    ${OpenCV_INCLUDE_DIRS}
)
#target_link_options(${library_name} PUBLIC "LINKER:-s")
target_link_libraries(${library_name} PUBLIC ${OpenCV_LIBS})
set_target_properties(${library_name} PROPERTIES
    #  PUBLIC_HEADER opencv_dart.h
    OUTPUT_NAME ${library_name}
)

if (WIN32)
    set_target_properties(${library_name} PROPERTIES
        WINDOWS_EXPORT_ALL_SYMBOLS ON
    )
endif (WIN32)

target_compile_definitions(${library_name} PUBLIC DART_SHARED_LIB)

if(WITH_OPENCV_DART_DEMO)
    add_executable(
        demo
        "demo.c"
    )
    target_link_libraries(
        demo
        PUBLIC
        ${OpenCV_LIBS}
        ${library_name}
    )
endif()

#add_custom_command(TARGET ${library_name} POST_BUILD
#    COMMAND ${CMAKE_COMMAND} -E copy_if_different
#    "$<TARGET_RUNTIME_DLLS:${library_name}>"
#    "$<TARGET_FILE_DIR:${library_name}>"
#    COMMAND_EXPAND_LISTS
#)
#add_custom_command(TARGET demo POST_BUILD
#    COMMAND ${CMAKE_COMMAND} -E copy_if_different
#    "$<TARGET_RUNTIME_DLLS:demo>"
#    "$<TARGET_FILE_DIR:demo>"
#    COMMAND_EXPAND_LISTS
#)

### install
install(TARGETS ${library_name}
    DESTINATION ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_PREFIX})
if (WIN32)
    file(GLOB CV_DEPENDENCIES "${CMAKE_BINARY_DIR}/*.dll")
elseif (APPLE)
    file(GLOB CV_DEPENDENCIES "${CMAKE_BINARY_DIR}/*.dylib")
elseif (UNIX)
    file(GLOB CV_DEPENDENCIES "${CMAKE_BINARY_DIR}/*.so")
endif ()
install(FILES ${CV_DEPENDENCIES} DESTINATION ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_PREFIX})
#file(COPY ${CV_DEPENDENCIES} DESTINATION ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_PREFIX})

# file(GLOB OPENCV_DART_DLLS "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_PREFIX}/*")
# install(FILES ${OPENCV_DART_DLLS} DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/../bin")
# if (WIN32)
#     # file(COPY ${OPENCV_DART_WIN_DLLS} DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/../windows/libs")
#     install(FILES ${OPENCV_DART_DLLS} DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/../windows")
# elseif (UNIX)
#     install(FILES ${OPENCV_DART_DLLS} DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/../linux")
# endif ()

