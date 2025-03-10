cmake_minimum_required( VERSION 3.15 )
project( mqtt_plain_text )

# Compiler flags.
add_compile_options( -Wall -Wextra -Wpedantic )
set( CMAKE_CXX_FLAGS_RELEASE "-O3" )
set( CMAKE_CXX_FLAGS_DEBUG "-O0 -g3" )

set( FREERTOS_ROOT_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../../../.." )

#########################################################################
# Build the FreeRTOS Kernel and FreeRTOS+TCP

# Kernel Paths
set( FREERTOS_KERNEL_ROOT "${FREERTOS_ROOT_PATH}/FreeRTOS/Source" )
set( FREERTOS_KERNEL_CONFIG_PATH 
        "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Demo/Common/FreeRTOS-Kernel" )
set( FREERTOS_HEAP "4" CACHE STRING "" FORCE )
set( FREERTOS_PORT "GCC_POSIX" CACHE STRING "" FORCE )

# Plus TCP Paths
set( FREERTOS_PLUS_TCP_ROOT "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Source/FreeRTOS-Plus-TCP" )
set( FREERTOS_PLUS_TCP_CONFIG_PATH 
        "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Demo/Common/FreeRTOS-PlusTCP"
        "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Source/FreeRTOS-Plus-TCP/tools/tcp_utilities/include" )
set( FREERTOS_PLUS_TCP_COMPILER "GCC" CACHE STRING "" FORCE )
set( FREERTOS_PLUS_TCP_NETWORK_IF "POSIX" CACHE STRING "" FORCE )

# Add the freertos_config for FreeRTOS-Kernel.
add_library( freertos_config INTERFACE )
target_include_directories( freertos_config INTERFACE
                            ${FREERTOS_KERNEL_CONFIG_PATH}
                            ${FREERTOS_PLUS_TCP_CONFIG_PATH} )

# Add the FreeRTOS-Kernel subdirectory.
add_subdirectory( ${FREERTOS_KERNEL_ROOT} ${CMAKE_CURRENT_BINARY_DIR}/FreeRTOS-Kernel )

add_subdirectory( ${FREERTOS_PLUS_TCP_ROOT} ${CMAKE_CURRENT_BINARY_DIR}/FreeRTOS-Plus-TCP )

#########################################################################
# Build Logging

add_library( logging STATIC
                "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Demo/Common/Logging/posix/Logging_Posix.c" )

target_include_directories( logging PUBLIC 
                            "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Source/Utilities/logging")  

target_link_libraries( logging
                        freertos_kernel
                        freertos_plus_tcp )

#########################################################################
# Build Demo

# coreMQTT paths
set( CORE_MQTT_ROOT "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Source/Application-Protocols/coreMQTT" )
set( CORE_MQTT_SOURCES
        "${CORE_MQTT_ROOT}/source/core_mqtt.c"
        "${CORE_MQTT_ROOT}/source/core_mqtt_state.c"
        "${CORE_MQTT_ROOT}/source/core_mqtt_serializer.c" )
set( CORE_MQTT_INCLUDE_DIRS
        "${CORE_MQTT_ROOT}/source/include"
        "${CORE_MQTT_ROOT}/source/interface"
        "${CMAKE_CURRENT_SOURCE_DIR}/../Common" )

# backoff algorithm paths
set( BACKOFF_ALGORITHM_ROOT "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Source/Utilities/backoff_algorithm" )
set( BACKOFF_ALGORITHM_SOURCES
        "${BACKOFF_ALGORITHM_ROOT}/source/backoff_algorithm.c" )
set( BACKOFF_ALGORITHM_INCLUDE_DIRS
        "${BACKOFF_ALGORITHM_ROOT}/source/include" )

# network transport paths
set( NETWORK_TRANSPORT_PATH "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Source/Application-Protocols/network_transport" )
set( NETWORK_TRANSPORT_SOURCES 
        "${NETWORK_TRANSPORT_PATH}/transport_plaintext.c" )
set( NETWORK_TRANSPORT_INCLUDE_DIRS
        "${NETWORK_TRANSPORT_PATH}" )

# TCP sockets paths
set( TCP_SOCKETS_WRAPPER_PATH "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Source/Application-Protocols/network_transport/tcp_sockets_wrapper" )
set( TCP_SOCKETS_WRAPPER_SOURCES 
        "${TCP_SOCKETS_WRAPPER_PATH}/ports/freertos_plus_tcp/tcp_sockets_wrapper.c" )
set( TCP_SOCKETS_WRAPPER_INCLUDE_DIRS
        "${TCP_SOCKETS_WRAPPER_PATH}/include" )

# Demo paths
set( DEMO_SOURCES 
        "DemoTasks/PlaintextMQTTExample.c"
        "../Common/main.c" )
set( DEMO_INCLUDE_DIRS
        . )

# Paths for the final executable
set( MAIN_SOURCES 
        "${DEMO_SOURCES}"
        "${CORE_MQTT_SOURCES}"
        "${BACKOFF_ALGORITHM_SOURCES}"
        "${NETWORK_TRANSPORT_SOURCES}"
        "${TCP_SOCKETS_WRAPPER_SOURCES}"
        "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Demo/Common/FreeRTOS-Kernel/freertos_hooks_posix.c"
        "${FREERTOS_ROOT_PATH}/FreeRTOS-Plus/Demo/Common/FreeRTOS-PlusTCP/plus_tcp_hooks_posix.c" )

add_executable( mqtt_plaintext_demo
                ${MAIN_SOURCES} )

target_include_directories( mqtt_plaintext_demo PRIVATE
                            "${DEMO_INCLUDE_DIRS}"
                            "${CORE_MQTT_INCLUDE_DIRS}"
                            "${BACKOFF_ALGORITHM_INCLUDE_DIRS}"
                            "${NETWORK_TRANSPORT_INCLUDE_DIRS}"
                            "${TCP_SOCKETS_WRAPPER_INCLUDE_DIRS}" )

target_link_libraries( mqtt_plaintext_demo PRIVATE
                        freertos_kernel
                        freertos_plus_tcp
                        logging )