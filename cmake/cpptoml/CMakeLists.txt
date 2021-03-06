cmake_minimum_required(VERSION 3.5)
project(cpptoml-build CXX)

# set download config, source and build paths
set(DOWNLOAD_CONFIG_DIR ${CMAKE_BINARY_DIR}/dependencies/cpptoml/download)
set(SOURCE_DIR ${CMAKE_BINARY_DIR}/dependencies/cpptoml/src)
set(BUILD_DIR ${CMAKE_BINARY_DIR}/dependencies/cpptoml/build)
set(INSTALL_DIR ${CMAKE_BINARY_DIR}/dependencies/install)

# Download and unpack cpptoml at configure time
configure_file(cpptoml.cmake.in ${DOWNLOAD_CONFIG_DIR}/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" "${DOWNLOAD_CONFIG_DIR}"
    RESULT_VARIABLE result
    WORKING_DIRECTORY ${DOWNLOAD_CONFIG_DIR} )
if(result)
    message(FATAL_ERROR "CMake step [configure download] for cpptoml failed: ${result}")
endif()

execute_process(COMMAND ${CMAKE_COMMAND} --build .
    RESULT_VARIABLE result
    WORKING_DIRECTORY ${DOWNLOAD_CONFIG_DIR} )
if(result)
    message(FATAL_ERROR "Build step [download] for cpptoml failed: ${result}")
endif()

file(MAKE_DIRECTORY ${BUILD_DIR})

execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" "-DENABLE_LIBCXX=off" "-DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}" "${SOURCE_DIR}"
    RESULT_VARIABLE result
    WORKING_DIRECTORY ${BUILD_DIR} )
if(result)
    message(FATAL_ERROR "CMake step [configure] for cpptoml failed: ${result}")
endif()

execute_process(COMMAND ${CMAKE_COMMAND} --build . --target install
    RESULT_VARIABLE result
    WORKING_DIRECTORY ${BUILD_DIR} )
if(result)
    message(FATAL_ERROR "Build step [build and install] for cpptoml failed: ${result}")
endif()

list(APPEND CMAKE_PREFIX_PATH ${INSTALL_DIR})
set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} CACHE INTERNAL "" FORCE)

