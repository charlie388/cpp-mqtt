cmake_minimum_required(VERSION 3.8)
project(cpp-mqtt CXX)

find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
# Update submodules as needed
    option(GIT_SUBMODULE "Check submodules during build" ON)
    if(GIT_SUBMODULE)
        message(STATUS "Submodule update")
        execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                        RESULT_VARIABLE GIT_SUBMOD_RESULT)
        if(NOT GIT_SUBMOD_RESULT EQUAL "0")
            message(FATAL_ERROR "git submodule update --init --recursive failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
        endif()
    endif()
endif()

set(CMAKE_CXX_STANDARD 17)

set(PAHO_BUILD_STATIC TRUE CACHE BOOL "my choice" FORCE)
set(PAHO_BUILD_SHARED FALSE CACHE BOOL "my choice" FORCE)
set(PAHO_WITH_SSL TRUE CACHE BOOL "my choice" FORCE)
set(PAHO_ENABLE_TESTING FALSE CACHE BOOL "my choice" FORCE)
set(PAHO_HIGH_PERFORMANCE TRUE CACHE BOOL "my choice" FORCE)
set(PAHO_MQTT_C_LIBRARIES "${PROJECT_BINARY_DIR}/paho.mqtt.c/src/libpaho-mqtt3as.a" CACHE FILEPATH "my choice" FORCE) 
set(PAHO_MQTT_C_INCLUDE_DIRS "${PROJECT_SOURCE_DIR}/paho.mqtt.c/src" CACHE PATH "my choice" FORCE) 

add_subdirectory(paho.mqtt.c)
add_subdirectory(paho.mqtt.cpp)

add_dependencies(paho-mqttpp3-static paho-mqtt3as-static)

add_executable(cpp-mqtt main.cpp)
target_link_libraries(cpp-mqtt paho-mqttpp3-static)

add_custom_command(
            TARGET cpp-mqtt POST_BUILD
            COMMAND openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -subj "/C=US/ST=Denial/L=Springfield/O=Dis/CN=localhost" -keyout tls.key  -out tls.crt
            VERBATIM)        