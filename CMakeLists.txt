
# ============================================================================
# This file will generate the cmake target `nanospline::nanospline`.
#
# To add nanospline as a dependency:
#
#   add_subdirectory(nanospline)
#   target_link_libraries(your_target nanospline::nanospline)
#
# ============================================================================

cmake_minimum_required(VERSION 3.11)
set(CMAKE_CXX_STANDARD 14)

project(nanospline)
find_package(catkin REQUIRED COMPONENTS
  roscpp
)
find_package (Eigen3 3.3 REQUIRED NO_MODULE)

catkin_package(
  INCLUDE_DIRS include
  CATKIN_DEPENDS roscpp
#  DEPENDS other non-ROS libs
)
option(NANOSPLINE_HIGH_DEGREE_SUPPORT "With high degree polynomial solver support" OFF)
option(NANOSPLINE_BUILD_TESTS "Build Tests" OFF)
option(NANOSPLINE_HEADER_ONLY "Enable header-only mode" ON)
option(NANOSPLINE_SYMPY "Enable sympy generated code (slow to compile)" OFF)

include(FetchContent)
# include(cmake/Eigen3.cmake)
include(cmake/MshIO.cmake)
include(cmake/sanitizer-cmake.cmake)


file(GLOB INC_FILES "${PROJECT_SOURCE_DIR}/include/nanospline/*.h"
    "${PROJECT_SOURCE_DIR}/include/nanospline/internal/*.h")
file(GLOB SRC_FILES "${PROJECT_SOURCE_DIR}/src/*.cpp")


add_library(nanospline INTERFACE)
target_link_libraries(${PROJECT_NAME} INTERFACE ${catkin_LIBRARIES} Eigen3::Eigen mshio::mshio)
target_include_directories(nanospline INTERFACE
    ${PROJECT_SOURCE_DIR}/include)

if (NANOSPLINE_SYMPY)
    target_compile_definitions(nanospline INTERFACE -DNANOSPLINE_SYMPY)
endif()

add_custom_target(nanospline_ SOURCES ${INC_FILES})

if(MSVC)
    target_compile_definitions(nanospline INTERFACE -D_USE_MATH_DEFINES)
    target_compile_options(nanospline INTERFACE "/bigobj")
endif()

if (NOT NANOSPLINE_HEADER_ONLY)
    add_library(nanospline_forward_decl STATIC ${SRC_FILES})
    target_link_libraries(nanospline_forward_decl PUBLIC nanospline)
    set_target_properties(nanospline_forward_decl PROPERTIES OUTPUT_NAME
        nanospline)

    if (NANOSPLINE_HIGH_DEGREE_SUPPORT)
        target_compile_definitions(nanospline_forward_decl PRIVATE
            -DHIGH_DEGREE_SUPPORT)
    endif()

    if(NOT MSVC)
        target_compile_options(nanospline_forward_decl PRIVATE -Wconversion -Wall -Werror)
    endif()

    add_library(nanospline::nanospline ALIAS nanospline_forward_decl)
    install(TARGETS nanospline_forward_decl EXPORT nanospline)
    install(DIRECTORY include/nanospline DESTINATION include)

    if (SANITIZE_ADDRESS OR
            SANITIZE_LINK_STATIC OR
            SANITIZE_MEMORY OR
            SANITIZE_THREAD OR
            SANITIZE_UNDEFINED)
        add_sanitizers(nanospline)
    endif()
else()
    target_compile_definitions(nanospline INTERFACE -DNANOSPLINE_HEADER_ONLY)
    add_library(nanospline::nanospline ALIAS nanospline)
    install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})
    install(DIRECTORY include/${PROJECT_NAME} DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
endif()


if (NANOSPLINE_BUILD_TESTS)
    include(CTest)
    enable_testing()
    include(cmake/Catch2.cmake)

    file(GLOB TEST_FILES "${PROJECT_SOURCE_DIR}/tests/*.cpp")
    add_executable(nanospline_test ${TEST_FILES})
    target_link_libraries(nanospline_test nanospline::nanospline Catch2::Catch2)
    target_compile_definitions(nanospline_test PRIVATE -DCATCH_CONFIG_ENABLE_BENCHMARKING)
    catch_discover_tests(nanospline_test)

    if(NOT MSVC)
        target_compile_options(nanospline_test PRIVATE -Wconversion -Wall -Werror)
    else()
        target_compile_options(nanospline_test PRIVATE "/MP")
        target_compile_definitions(nanospline_test PRIVATE -D_USE_MATH_DEFINES)
    endif()

    if (SANITIZE_ADDRESS OR
            SANITIZE_LINK_STATIC OR
            SANITIZE_MEMORY OR
            SANITIZE_THREAD OR
            SANITIZE_UNDEFINED)
        add_sanitizers(nanospline_test)
    endif()

    add_custom_target(run_unit_tests
        COMMAND nanospline_test
        DEPENDS nanospline_test)
endif()

# # ============================================================================
# # This file will generate the cmake target `nanospline::nanospline`.
# #
# # To add nanospline as a dependency:
# #
# #   add_subdirectory(nanospline)
# #   target_link_libraries(your_target nanospline::nanospline)
# #
# # ============================================================================

# cmake_minimum_required(VERSION 3.5)
# set(CMAKE_CXX_STANDARD 14)

# project(nanospline)

# option(NANOSPLINE_HIGH_DEGREE_SUPPORT "With high degree polynomial solver support" OFF)
# option(NANOSPLINE_BUILD_TESTS "Build Tests" OFF)
# option(NANOSPLINE_HEADER_ONLY "Enable header-only mode" ON)
# option(NANOSPLINE_SYMPY "Enable sympy generated code (slow to compile)" OFF)

# find_package(catkin_simple REQUIRED)
# find_package (Eigen3 3.3 REQUIRED NO_MODULE)

# include(FetchContent)
# include(cmake/MshIO.cmake)
# include(cmake/sanitizer-cmake.cmake)


# file(GLOB INC_FILES "${PROJECT_SOURCE_DIR}/include/nanospline/*.h"
#     "${PROJECT_SOURCE_DIR}/include/nanospline/internal/*.h")
# file(GLOB SRC_FILES "${PROJECT_SOURCE_DIR}/src/*.cpp")


# cs_add_library(${PROJECT_NAME}_core INTERFACE)
# target_link_libraries(${PROJECT_NAME}_core INTERFACE Eigen3::Eigen mshio::mshio)
# include_directories(
#     include
#     ${catkin_INCLUDE_DIRS}
#     ${PROJECT_NAME}_core INTERFACE
# )


# add_custom_target(nanospline_ SOURCES ${INC_FILES})

# target_compile_definitions(${PROJECT_NAME}_core INTERFACE -DNANOSPLINE_HEADER_ONLY)
# cs_add_library(nanospline::nanospline ALIAS nanospline)
#     install(TARGETS nanospline EXPORT nanospline)
#     install(DIRECTORY include/nanospline DESTINATION include)
# cs_export()

