# 25/06/06 = Fri

# CMake 3.10 was released on 2017-11-20
cmake_minimum_required(VERSION 3.10)
project(CppFutuProto LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

file(GLOB PROTO_FILES "${CMAKE_SOURCE_DIR}/proto/*.proto")
if(PROTO_FILES STREQUAL "")
    message(FATAL_ERROR "No .proto files found in proto/")
endif()

set(PROTO_SRC_DIR "${CMAKE_SOURCE_DIR}/src")
set(PROTO_HDR_DIR "${CMAKE_SOURCE_DIR}/include")
file(MAKE_DIRECTORY ${PROTO_SRC_DIR})
file(MAKE_DIRECTORY ${PROTO_HDR_DIR})

foreach(PROTO_FILENAME ${PROTO_FILES})
    get_filename_component(PROTO_FILENAME_WE ${PROTO_FILENAME} NAME_WE)
    list(APPEND SRC_FILES "${PROTO_SRC_DIR}/${PROTO_FILENAME_WE}.pb.cc")
    list(APPEND HDR_FILES "${PROTO_HDR_DIR}/${PROTO_FILENAME_WE}.pb.h")
endforeach()

add_library(futuproto STATIC)
target_include_directories(futuproto PUBLIC
    ${PROTO_HDR_DIR}
    ${CMAKE_SOURCE_DIR}/protobuf/src
)
target_compile_options(futuproto PRIVATE -Wpedantic -Wall -Wextra)
target_link_libraries(futuproto PUBLIC protobuf)
target_sources(futuproto PRIVATE ${SRC_FILES})

set(PROTOC "${CMAKE_SOURCE_DIR}/protobuf/src/protoc")
if(NOT EXISTS "${PROTOC}")
    message(FATAL_ERROR "protoc compiler not found at ${PROTOC}")
endif()
add_custom_command(
    OUTPUT ${SRC_FILES}
    COMMAND ${PROTOC}
    ARGS --proto_path=${CMAKE_SOURCE_DIR}/proto
         --cpp_out=${PROTO_SRC_DIR}
         ${PROTO_FILES}
    DEPENDS ${PROTO_FILES}
    COMMENT "Generating all protobuf .pb.cc files ..."
    VERBATIM
)
add_custom_target(generate_pb_cc ALL DEPENDS ${SRC_FILES})
add_custom_command(
    OUTPUT ${HDR_FILES}
    COMMAND /bin/sh -c "mv ${CMAKE_SOURCE_DIR}/src/*.pb.h ${CMAKE_SOURCE_DIR}/include/"
    COMMENT "Moving all .pb.h files from src/ to include/ ..."
    DEPENDS generate_pb_cc
    VERBATIM
)
add_custom_target(generate_pb_h ALL DEPENDS ${HDR_FILES})
add_dependencies(futuproto generate_pb_h)

message(STATUS "Found protoc at: ${PROTOC}")
message(STATUS "Proto files: ${PROTO_FILES}")
message(STATUS "Headers dir: ${PROTO_HDR_DIR}")
message(STATUS "Sources dir: ${PROTO_SRC_DIR}")
