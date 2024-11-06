cmake_minimum_required(VERSION 3.5)
project(livox_sdk2)

# Find ROS 2 ament packages
find_package(ament_cmake REQUIRED)

set(CMAKE_CXX_STANDARD 11)
message(STATUS "Main project directory: ${PROJECT_SOURCE_DIR}")

# Handle cross-compiling
if (CMAKE_CROSSCOMPILING)
    set(THREADS_PTHREAD_ARG "PLEASE_FILL_OUT-FAILED_TO_RUN" CACHE STRING "Result from TRY_RUN" FORCE)
endif()

# Set threading flags for UNIX
if (UNIX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
endif()

# Library names
set(SDK_LIBRARY_STATIC livox_lidar_sdk_static)
set(SDK_LIBRARY_SHARED livox_lidar_sdk_shared)

# Create static and shared libraries
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
add_library(${SDK_LIBRARY_STATIC} STATIC "")
add_library(${SDK_LIBRARY_SHARED} SHARED "")

# Version information
set(LIVOX_SDK_VERSION_STRING "0.0.2")

# Header files
message(STATUS "Header files: ${PROJECT_SOURCE_DIR}/include")
set(LIVOX_API_HEADER
    ${PROJECT_SOURCE_DIR}/include/livox_lidar_def.h
    ${PROJECT_SOURCE_DIR}/include/livox_lidar_api.h
    ${PROJECT_SOURCE_DIR}/include/livox_lidar_cfg.h
)

# Platform-specific options
if (WIN32)
    set(PLATFORM win)
else()
    set(PLATFORM unix)
endif()

# Compiler-specific options
target_compile_options(${SDK_LIBRARY_STATIC}
    PRIVATE $<$<CXX_COMPILER_ID:GNU>:-Wall>
)

target_compile_options(${SDK_LIBRARY_SHARED}
    PRIVATE $<$<CXX_COMPILER_ID:GNU>:-Wall>
)

# Include directories
set(LIVOX_PUBLIC_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include)
set(LIVOX_PRIVATE_INCLUDE_DIR
    ${PROJECT_SOURCE_DIR}/3rdparty
    ${PROJECT_SOURCE_DIR}/sdk_core
)

target_include_directories(${SDK_LIBRARY_STATIC} PUBLIC ${LIVOX_PUBLIC_INCLUDE_DIR} PRIVATE ${LIVOX_PRIVATE_INCLUDE_DIR})
target_include_directories(${SDK_LIBRARY_SHARED} PUBLIC ${LIVOX_PUBLIC_INCLUDE_DIR} PRIVATE ${LIVOX_PRIVATE_INCLUDE_DIR})

set(MAIN_SOURCES
        sdk_core/device_manager.cpp
        sdk_core/livox_lidar_sdk.cpp
        sdk_core/params_check.cpp
        sdk_core/parse_cfg_file.cpp
        sdk_core/upgrade_manager.cpp
        )
set(BASE_SOURCES
        sdk_core/base/io_loop.cpp
        sdk_core/base/thread_base.cpp
        sdk_core/base/io_thread.cpp
        sdk_core/base/logging.cpp
        sdk_core/base/network/${PLATFORM}/network_util.cpp
        sdk_core/base/multiple_io/multiple_io_base.cpp
        sdk_core/base/multiple_io/multiple_io_epoll.cpp
        sdk_core/base/multiple_io/multiple_io_poll.cpp
        sdk_core/base/multiple_io/multiple_io_select.cpp
        sdk_core/base/multiple_io/multiple_io_kqueue.cpp
        sdk_core/base/wake_up/${PLATFORM}/wake_up_pipe.cpp
        )
set(COMM_SOURCES
		sdk_core/comm/comm_port.cpp
        sdk_core/comm/sdk_protocol.cpp
        sdk_core/comm/generate_seq.cpp
        )
set(UPGRADE_SOURCES
		sdk_core/upgrade_manager.cpp
        sdk_core/upgrade/firmware.cpp
        sdk_core/upgrade/livox_lidar_upgrader.cpp
        )
set(LOGGER_HANDLER_SOURCES
		sdk_core/logger_handler/logger_manager.cpp
        sdk_core/logger_handler/logger_handler.cpp
        sdk_core/logger_handler/file_manager.cpp
        )
set(DATA_HANDLER_SOURCES
		sdk_core/data_handler/data_handler.cpp
        )
set(COMMAND_HANDLER_SOURCES
		sdk_core/command_handler/command_impl.cpp
        sdk_core/command_handler/general_command_handler.cpp
        sdk_core/command_handler/hap_command_handler.cpp
        sdk_core/command_handler/mid360_command_handler.cpp
        sdk_core/command_handler/build_request.cpp
        sdk_core/command_handler/parse_lidar_state_info.cpp
        )
set(DEBUG_POINT_CLOUD_HANDLER_SOURCES
		sdk_core/debug_point_cloud_handler/debug_point_cloud_manager.cpp
        sdk_core/debug_point_cloud_handler/debug_point_cloud_handler.cpp
        )

set(LIVOX_SOURCES
    ${PROJECT_SOURCE_DIR}/3rdparty/FastCRC/FastCRC_tables.h
    ${PROJECT_SOURCE_DIR}/3rdparty/FastCRC/FastCRCsw.cpp
    ${MAIN_SOURCES}
    ${BASE_SOURCES}
    ${COMM_SOURCES}
    ${UPGRADE_SOURCES}
    ${LOGGER_HANDLER_SOURCES}
    ${DATA_HANDLER_SOURCES}
    ${COMMAND_HANDLER_SOURCES}
    ${DEBUG_POINT_CLOUD_HANDLER_SOURCES}
)

target_sources(${SDK_LIBRARY_STATIC} PRIVATE ${LIVOX_SOURCES})
target_sources(${SDK_LIBRARY_SHARED} PRIVATE ${LIVOX_SOURCES})

# Installation targets
install(TARGETS ${SDK_LIBRARY_STATIC} ${SDK_LIBRARY_SHARED}
    PUBLIC_HEADER DESTINATION include/${PROJECT_NAME}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

# Installation targets for header files
install(FILES ${LIVOX_API_HEADER}
    DESTINATION include/${PROJECT_NAME}
)

# Export include directories and libraries for ament
ament_export_include_directories(include/${PROJECT_NAME})
ament_export_libraries(${SDK_LIBRARY_STATIC} ${SDK_LIBRARY_SHARED})

# Required for ROS 2 packages
ament_package()