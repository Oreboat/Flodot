cmake_minimum_required(VERSION 3.20)

# this is what your project is called (and will name the .so file using this as well)
set(GDEXTENSION_LIB_NAME Flodot)

# the location that the library should be copied into after it's built (this
# should be the `bin/` folder that includes your godot project files
set(GDEXTENSION_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/project/bin")

# this tells cmake which version of C++ you want to use
# (C++11, C++14, C++17, C++20, C++23, etc..) and a few
# other options that will dictate some C/C++ settings
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_EXTENSIONS ON)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# this will initialize/declare the cmake project
# (named after the variable defined above)
project("${GDEXTENSION_LIB_NAME}"
  LANGUAGES
    C CXX
  VERSION
    0.1.0
)

# this will end up calling the cmake files contained in the godot-cpp repo.
# it will handle configuration of that library and importing the headers, etc...
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/extern/godot-cpp")
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/extern/flecs")

# create a file listing of all source files in the extension
# this will create that list recursively of all .h/.c/.hpp/.cpp
# in the src folder and will auto-detect added/removed/renamed files
file(GLOB_RECURSE gdext_sources
     CONFIGURE_DEPENDS
       "${CMAKE_CURRENT_SOURCE_DIR}/src/*.[hc]"
       "${CMAKE_CURRENT_SOURCE_DIR}/src/*.[hc]pp"
)

# declare your gdextension shared library
# and all the source files to it
add_library(${PROJECT_NAME}
    SHARED
      ${gdext_sources}
)

# add the src/ folder as a path that your project can
# import as a header include path (useful for IDEs)
target_include_directories(${PROJECT_NAME} PUBLIC
    "${CMAKE_CURRENT_SOURCE_DIR}/src"
)

target_compile_features(${PROJECT_NAME} PUBLIC
    cxx_std_20
)

# linker options for the gdextension library
target_link_options(${PROJECT_NAME} PRIVATE
    -static-libgcc
    -static-libstdc++
    -Wl,-R,'$$ORIGIN'
)

# linker command that will link the godot-cpp library to yours
target_link_libraries(${PROJECT_NAME}
    PUBLIC godot::cpp
    flecs::flecs_static
)

set_target_properties(${PROJECT_NAME}
    PROPERTIES
        POSITION_INDEPENDENT_CODE ON
        CMAKE_EXPORT_COMPILE_COMMANDS ON
        CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELEASE ON
        ARCHIVE_OUTPUT_DIRECTORY "${GDEXTENSION_LIB_PATH}"
        LIBRARY_OUTPUT_DIRECTORY "${GDEXTENSION_LIB_PATH}"
        RUNTIME_OUTPUT_DIRECTORY "${GDEXTENSION_LIB_PATH}"
        CMAKE_PDB_OUTPUT_DIRECTORY "${GDEXTENSION_LIB_PATH}"
        CMAKE_COMPILE_PDB_OUTPUT_DIRECTORY "${GDEXTENSION_LIB_PATH}"
)
set(GODOT_COMPILE_FLAGS "${GODOT_COMPILE_FLAGS} -DHOT_RELOAD_ENABLED")
