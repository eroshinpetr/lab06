cmake_minimum_required(VERSION 3.4)

project(print)
set(PRINT_VERSION_MAJOR 0)
set(PRINT_VERSION_MINOR 1)
set(PRINT_VERSION_PATCH 0)
set(PRINT_VERSION_TWEAK 0)
set(PRINT_VERSION
  ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK})
set(PRINT_VERSION_STRING "v${PRINT_VERSION}")

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_EXAMPLES "Build examples" OFF)
option(BUILD_TESTS "Build tests" OFF)

add_library(${PROJECT_NAME} STATIC
  ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp
)

target_include_directories(${PROJECT_NAME}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

if(BUILD_EXAMPLES)
  add_executable(example
    ${CMAKE_CURRENT_SOURCE_DIR}/examples/example.cpp
  )

  target_link_libraries(example
    ${PROJECT_NAME}
  )
endif()

install(TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}-config
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)

install(EXPORT ${PROJECT_NAME}-config DESTINATION cmake)

if(BUILD_TESTS)
  enable_testing()

  add_subdirectory(third-party/gtest)

  if(TARGET gtest)
    target_compile_options(gtest PRIVATE -Wno-error=maybe-uninitialized)
  endif()

  if(TARGET gtest_main)
    target_compile_options(gtest_main PRIVATE -Wno-error=maybe-uninitialized)
  endif()

  if(TARGET gmock)
    target_compile_options(gmock PRIVATE -Wno-error=maybe-uninitialized)
  endif()

  if(TARGET gmock_main)
    target_compile_options(gmock_main PRIVATE -Wno-error=maybe-uninitialized)
  endif()

  file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)

  add_executable(check
    ${${PROJECT_NAME}_TEST_SOURCES}
  )

  target_link_libraries(check
    ${PROJECT_NAME}
    gtest_main
  )

  add_test(NAME check COMMAND check)
endif()

include(CPackConfig.cmake)
