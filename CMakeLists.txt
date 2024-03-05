cmake_minimum_required(VERSION 3.20)

project(deviceManager LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)

find_package(sdbus-c++ REQUIRED)

include_directories(
    ${CMAKE_CURRENT_LIST_DIR}/libmosquitto/lib/cpp/
)

add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/sources/ConnectivityStack)
add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/sources/RaucManager)
add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/sources/Common)

set(
    SOURCES
    ${Common}
    ${ConnectivityStack}
    ${RaucManager}
)

add_executable(
    futo_project
    ${PROJECT_SOURCE_DIR}/sources/main.cpp
    ${SOURCES}
)

dm_enable_qa(futo_project)

target_link_libraries(
    SDBusCpp::sdbus-c++
    mosquittopp_static libmosquitto_static
)

# clang-format
#

# Step 1 list the files to be formated.
file(GLOB_RECURSE SOURCE_FILES_TO_FORMAT RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.cpp *.hpp)
foreach (SOURCE_FILE ${SOURCE_FILES_TO_FORMAT})
    string(FIND ${SOURCE_FILE} "sources/" PROJECT_SOURCES_DIR_FOUND)
    string(FIND ${SOURCE_FILE} "tests/" PROJECT_TESTS_DIR_FOUND)
    if (NOT ${PROJECT_SOURCES_DIR_FOUND} EQUAL 0 AND NOT ${PROJECT_TESTS_DIR_FOUND} EQUAL 0)
        list(REMOVE_ITEM SOURCE_FILES_TO_FORMAT ${SOURCE_FILE})
    endif ()
endforeach ()
list(TRANSFORM SOURCE_FILES_TO_FORMAT PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/")
message(NOTICE "Files to be clang-format checked : ${SOURCE_FILES_TO_FORMAT}")

# Step 2 check the formating during build.
add_custom_target(
    clangformat-check
    ALL
    COMMAND /usr/bin/clang-format
    -style=file
    --dry-run
    --Werror
    ${SOURCE_FILES_TO_FORMAT}
)

# Step 3 Manually run "cmake clangformat-correct" to fix the format of files.
# Warning ! This will edit the source files inplace, commit your other changes before !
add_custom_target(
    clangformat-correct
    COMMAND /usr/bin/clang-format
    -style=file
    -i
    ${SOURCE_FILES_TO_FORMAT}
)