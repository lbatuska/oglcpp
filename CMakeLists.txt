cmake_minimum_required(VERSION 3.5)

project(oglcpp)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

file(GLOB SRC_FOUND CONFIGURE_DEPENDS
     "src/*.c"
     "src/*.cpp"
)

# Removes main if it's in SRC_FOUND probably easy workaround is to move main outse of src folder
list(REMOVE_ITEM SRC_FOUND "${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp")
list(REMOVE_ITEM SRC_FOUND "${CMAKE_CURRENT_SOURCE_DIR}/src/main.c")

if (CMAKE_BUILD_TYPE MATCHES Debug)
    add_definitions(-DDEBUG)
endif()

set(OpenGL_GL_PREFERENCE GLVND) # Use the vendor neutral GL dispatch library instead of LEGACY
find_package( OpenGL REQUIRED )

include_directories( ${OPENGL_INCLUDE_DIRS} )

set( GLFW_BUILD_DOCS OFF CACHE BOOL  "GLFW lib only" )
set( GLFW_INSTALL OFF CACHE BOOL  "GLFW lib only" )
set( GLAD_GL "" )

# set variables that are needed
set(ZLIB_LIBRARIES zlibstatic)
set(ENABLE_BOOST_WORKAROUND ON)
set(BUILD_STATIC_LIB ON)
set(BUILD_ASSIMP_TOOLS  ON)
set(ASSIMP_BUILD_STATIC_LIB ON)

add_subdirectory(deps/glm/)
add_subdirectory(deps/assimp/)

include(FetchContent)

# ========== GLAD ==========
# Multi-Language Vulkan/GL/GLES/EGL/GLX/WGL Loader-Generator based on
# the official specs. http://glad.dav1d.de/
FetchContent_Declare(
  extern_glad

  GIT_REPOSITORY https://github.com/Dav1dde/glad.git
  GIT_TAG        v0.1.27)

FetchContent_GetProperties(extern_glad)
if(NOT extern_glad_POPULATED)
  FetchContent_Populate(extern_glad)

  set(GLAD_INSTALL ON)
  add_subdirectory(
    ${extern_glad_SOURCE_DIR}
    ${extern_glad_BINARY_DIR}
    EXCLUDE_FROM_ALL)
endif()

add_subdirectory(deps/glad/cmake/)
# ========== GLAD ==========


# ========== GLFW ==========
add_subdirectory(deps/glfw)
include_directories(deps/glfw/include)
# ========== GLFW ==========

# ========== IMGUI ==========
add_compile_definitions(IMGUI_IMPL_OPENGL_LOADER_GL3W)

set(IMGUI_PATH deps/imgui)

set(IMGUI_SOURCES
    ${IMGUI_PATH}/imgui.cpp
    ${IMGUI_PATH}/imgui_demo.cpp
    ${IMGUI_PATH}/imgui_draw.cpp
    ${IMGUI_PATH}/imgui_widgets.cpp
    ${IMGUI_PATH}/imgui_tables.cpp
    ${IMGUI_PATH}/backends/imgui_impl_glfw.cpp
    ${IMGUI_PATH}/backends/imgui_impl_opengl3.cpp
)
# ========== IMGUI ==========


add_executable(oglcpp "src/main.cpp" ${SRC_FOUND} ${IMGUI_SOURCES})
include_directories(include)


target_include_directories(oglcpp PUBLIC deps/glm/)
target_include_directories(oglcpp PUBLIC deps/assimp/include/)
target_include_directories(oglcpp PUBLIC ${IMGUI_PATH} ${IMGUI_PATH}/backends)

target_link_libraries(oglcpp PUBLIC glad)
# ========== GLAD ==========

# ========== GLFW ==========
target_link_libraries(oglcpp PUBLIC glfw)
# ========== GLFW ==========

target_link_libraries(oglcpp PUBLIC assimp)

target_link_libraries(oglcpp PUBLIC ${CMAKE_DL_LIBS}) # -ldl, see https://cmake.org/cmake/help/v3.0/variable/CMAKE_DL_LIBS.html


set(CMAKE_DEBUG "")
if(DEFINED CMAKE_DEBUG OR DEFINED ENV{CMAKE_DEBUG})
message("==================== DEBUG ====================")

    message("PROJECT => ${PROJECT_NAME}")
    message("CMAKE_BUILD_TYPE => ${CMAKE_BUILD_TYPE}")

    message("========== SRC_FOUND ==========")

    foreach(i ${SRC_FOUND})
    message("${i}")
    endforeach()


    message("========== TARGETS ==========")
    get_property(target_names_debug DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY BUILDSYSTEM_TARGETS)
    foreach(i ${target_names_debug})
    message("${i}")
    endforeach()


    foreach(i ${target_names_debug})
      message("========== ${i} INCLUDE DIRS ==========")
      get_property(include_dirs_debug TARGET ${i} PROPERTY INCLUDE_DIRECTORIES)
      foreach(i ${include_dirs_debug})
      message("${i}")
      endforeach()
    endforeach()


    message("========== C++ compiler features ==========")

    foreach(i ${CMAKE_CXX_COMPILE_FEATURES})
    message("${i}")
    endforeach()

    message("========== C compiler features ==========")

    foreach(i ${CMAKE_C_COMPILE_FEATURES})
    message("${i}")
    endforeach()
message("==================== DEBUG ====================")
endif()
