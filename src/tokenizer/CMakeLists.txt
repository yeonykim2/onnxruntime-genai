set(TOKENIZER_ROOT ${CMAKE_CURRENT_SOURCE_DIR} PARENT_SCOPE)
include(${CMAKE_SOURCE_DIR}/cmake/cxx_standard.cmake)

file(GLOB tokenizer_srcs CONFIGURE_DEPENDS
   "${CMAKE_CURRENT_SOURCE_DIR}/*.cc"
   "${CMAKE_CURRENT_SOURCE_DIR}/utils/*.cc"
   "${CMAKE_CURRENT_SOURCE_DIR}/c_api/*.cc"
)

FetchContent_Declare(GSL URL https://github.com/microsoft/GSL/archive/refs/tags/v4.0.0.zip)
onnxruntime_fetchcontent_makeavailable(GSL)

FetchContent_Declare(simdjson URL https://github.com/simdjson/simdjson/archive/refs/tags/v3.6.3.zip URL_HASH SHA1=2b063a2e81f74a5d1cb937fadf3d2fca0f1edb09)
onnxruntime_fetchcontent_makeavailable(simdjson)
add_library(tokenizer STATIC ${tokenizer_srcs})

message(STATUS "GSL_SOURCE_DIR: ${GSL_SOURCE_DIR}")
target_include_directories(tokenizer PUBLIC
   ${CMAKE_CURRENT_SOURCE_DIR}
   ${GSL_SOURCE_DIR}/include
   ${simdjson_SOURCE_DIR}/singleheader/
)
target_compile_definitions(tokenizer PRIVATE _SILENCE_ALL_CXX20_DEPRECATION_WARNINGS)

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  target_link_libraries(tokenizer PRIVATE stdc++fs)
  set_target_properties(tokenizer PROPERTIES POSITION_INDEPENDENT_CODE ON)
  set_target_properties(simdjson PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()

target_link_libraries(tokenizer PUBLIC simdjson)