cmake_minimum_required(VERSION 3.21)
project(Platformer LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)

include(FetchContent)
FetchContent_Declare(SFML
    GIT_REPOSITORY https://github.com/SFML/SFML.git
    GIT_TAG 2.6.x)
FetchContent_MakeAvailable(SFML)

# B2D - Box2D physics library
add_subdirectory("lib/b2d/Box2D")
#include_directories("lib/b2d/Box2D/")
set(B2D_INCS "lib/b2d/Box2D/")
link_directories("${CMAKE_BINARY_DIR}/lib/sfml/lib")

set(SFML_INCS "lib/sfml/include")

#### Add Engine and our own library projects####

file(GLOB_RECURSE RESOURCE_FILES "res/*.*")
## Engine ##
file(GLOB_RECURSE SOURCE_FILES engine/*.cpp engine/*.h)
add_library(lib_engine STATIC ${SOURCE_FILES} ${RESOURCE_FILES})
target_include_directories(lib_engine INTERFACE "${CMAKE_SOURCE_DIR}/engine/" PUBLIC SYSTEM ${B2D_INCS} ${SFML_INCS})
target_link_libraries(lib_engine lib_tile_level_loader lib_maths lib_ecm Box2D sfml-graphics sfml-audio sfml-system sfml-window)

## Tile loader lib
file(GLOB_RECURSE SOURCE_FILES lib_tile_level_loader/*.cpp lib_tile_level_loader/*.h)
add_library(lib_tile_level_loader STATIC ${SOURCE_FILES})
target_include_directories(lib_tile_level_loader INTERFACE "${CMAKE_SOURCE_DIR}/lib_tile_level_loader/" )
target_link_libraries(lib_tile_level_loader lib_maths sfml-graphics)

# Maths lib
add_library(lib_maths INTERFACE)
target_sources(lib_maths INTERFACE "${CMAKE_SOURCE_DIR}/lib_maths/maths.h")
target_include_directories(lib_maths INTERFACE "${CMAKE_SOURCE_DIR}/lib_maths" SYSTEM INTERFACE ${SFML_INCS})
target_link_libraries(lib_maths INTERFACE sfml-system sfml-graphics)

## ECM lib
file(GLOB_RECURSE SOURCE_FILES lib_ecm/*.cpp lib_ecm/*.h)
add_library(lib_ecm STATIC ${SOURCE_FILES})
target_include_directories(lib_ecm INTERFACE "${CMAKE_SOURCE_DIR}/lib_ecm" )
target_link_libraries(lib_ecm PRIVATE lib_maths)

## Add Lab 7 Platformer as Library
file(GLOB_RECURSE SOURCE_FILES lab_7_platformer/*.cpp lab_7_platformer/*.h lab_7_platformer/components/*.cpp lab_7_platformer/components/*.h lab_7_platformer/scenes/*.cpp lab_7_platformer/scenes/*.h)
add_library(lab_7_platformer STATIC ${SOURCE_FILES})
target_include_directories(lab_7_platformer INTERFACE "${CMAKE_SOURCE_DIR}/lab_7_platformer/" SYSTEM INTERFACE ${SFML_INCS})
target_link_libraries(lab_7_platformer PRIVATE lib_maths lib_ecm lib_engine lib_tile_level_loader Box2D sfml-graphics sfml-audio sfml-system sfml-window)

add_executable(Platformer src/platformer.cpp)
target_link_libraries(Platformer PRIVATE sfml-graphics lab_7_platformer lib_engine lib_tile_level_loader lib_maths lib_ecm Box2D sfml-audio)
target_compile_features(Platformer PRIVATE cxx_std_17)
if (WIN32 AND BUILD_SHARED_LIBS)
    add_custom_command(TARGET Platformer POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_RUNTIME_DLLS:Platformer> $<TARGET_FILE_DIR:Platformer> COMMAND_EXPAND_LISTS)
endif()

install(TARGETS Platformer)
