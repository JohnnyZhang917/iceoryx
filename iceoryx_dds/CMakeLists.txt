# Copyright (c) 2020 by Robert Bosch GmbH. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 3.7)

file (STRINGS "../VERSION" iceoryx_dds_VERSION)
project(iceoryx_dds VERSION ${iceoryx_dds_VERSION})

option(BUILD_TESTS "build test binaries")

find_package(iceoryx_posh REQUIRED)
find_package(iceoryx_utils REQUIRED)

include(IceoryxPackageHelper)
include(IceoryxPlatformDetection)

#
########## set variables for library export ##########
#
setup_package_name_and_create_files(
    NAME ${PROJECT_NAME}
    NAMESPACE iceoryx_dds
    PROJECT_PREFIX ${PREFIX}
)

#
########## feature flags ##########
#
option(USE_CYCLONE_DDS "Bind to CycloneDDS implementation" on)

if(USE_CYCLONE_DDS)
    message(INFO " Using CycloneDDS stack")
    find_package(CycloneDDS CONFIG REQUIRED)
    find_package(CycloneDDS_CXX_API CONFIG REQUIRED)
    find_package(Idlpp-cxx CONFIG REQUIRED)
endif()

#
########## build building-block library ##########
#
add_library(iceoryx_dds
    STATIC
    source/iceoryx_dds/log/logging.cpp
    source/iceoryx_dds/gateway/gateway_config.cpp
    source/iceoryx_dds/gateway/toml_gateway_config_parser.cpp
)
add_library(${PROJECT_NAMESPACE}::iceoryx_dds ALIAS iceoryx_dds)

target_compile_options(iceoryx_dds PUBLIC -std=c++11)

target_include_directories(iceoryx_dds
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include/${PREFIX}>
)
target_link_libraries(iceoryx_dds
    PRIVATE
    iceoryx_posh::iceoryx_posh
    iceoryx_utils::iceoryx_utils
)

if(USE_CYCLONE_DDS)
    target_sources(iceoryx_dds
        PRIVATE
        source/iceoryx_dds/dds/cyclone_context.cpp
        source/iceoryx_dds/dds/cyclone_data_reader.cpp
        source/iceoryx_dds/dds/cyclone_data_writer.cpp
    )
    target_compile_definitions(iceoryx_dds PUBLIC -DUSE_CYCLONE_DDS)
    idl_ddscxx_generate(mempoolMessages_lib "msg/Mempool.idl")
    target_link_libraries(iceoryx_dds
        PUBLIC
        CycloneDDS_CXX_API::ddscxx
        mempoolMessages_lib
    )
endif()

target_compile_options(iceoryx_dds PRIVATE ${ICEORYX_WARNINGS})

#
########## build gateway apps ##########
#
add_executable(gateway_iceoryx2dds
    source/iceoryx2dds_app/main.cpp
)
target_link_libraries(gateway_iceoryx2dds
    iceoryx_posh::iceoryx_posh
    ${PROJECT_NAMESPACE}::iceoryx_dds
)

target_compile_options(gateway_iceoryx2dds PRIVATE ${ICEORYX_WARNINGS})

add_executable(gateway_dds2iceoryx
    source/dds2iceoryx_app/main.cpp
)
target_link_libraries(gateway_dds2iceoryx
    iceoryx_posh::iceoryx_posh
    ${PROJECT_NAMESPACE}::iceoryx_dds
)

target_compile_options(gateway_dds2iceoryx PRIVATE ${ICEORYX_WARNINGS})

#
########## build test executables ##########
#
if(BUILD_TESTS)
    add_subdirectory(test)
endif(BUILD_TESTS)

#
########## export library ##########
#
setup_install_directories_and_export_package(
    TARGETS gateway_iceoryx2dds gateway_dds2iceoryx
    INCLUDE_DIRECTORY include/
)

