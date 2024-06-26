cmake_minimum_required (VERSION 3.10)
project(imgui-ws)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    set(IMGUI_WS_STANDALONE ON)
else()
    set(IMGUI_WS_STANDALONE OFF)
endif()

add_definitions(-DUWS_NO_ZLIB)

set(IMGUI_WS_WORKING_DIR "${CMAKE_CURRENT_SOURCE_DIR}")

macro(suppress_compile_warnings)
    if (MSVC)
        # Suppress warning C4996 (deprecated declarations) for Visual Studio
        add_compile_options(/wd4996 /wd4100 /wd4267 /wd4996)
    else()
        # Suppress warnings about deprecated OpenSSL functions and unused parameters for GCC and Clang
        add_compile_options(
            -Wno-deprecated-declarations
            -Wno-sign-compare
            -Wno-unused-parameter
            -Wno-unused-variable       
             )
    endif()

    if (IMGUI_WS_ALL_WARNINGS AND (CMAKE_COMPILER_IS_GNUCC OR CMAKE_CXX_COMPILER_ID MATCHES "Clang"))
        # Enable all warnings for GCC and Clang
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic")
    elseif (IMGUI_WS_ALL_WARNINGS)
        message(WARNING "imgui-ws is not supported on Windows!")
    endif()
endmacro()

macro(add_imgui target_name)
    if (NOT TARGET imgui)
        add_library(imgui STATIC
            ${imgui_SOURCE_DIR}/imgui.cpp
            ${imgui_SOURCE_DIR}/imgui_demo.cpp
            ${imgui_SOURCE_DIR}/imgui_draw.cpp
            ${imgui_SOURCE_DIR}/imgui_tables.cpp
            ${imgui_SOURCE_DIR}/imgui_widgets.cpp
            #
            ${implot_SOURCE_DIR}/implot.cpp
            ${implot_SOURCE_DIR}/implot_items.cpp
            ${implot_SOURCE_DIR}/implot_demo.cpp
        )
    endif()
    add_dependencies(${target_name} imgui)   
    target_link_libraries(${target_name} PRIVATE imgui)
endmacro()

set(CMAKE_EXPORT_COMPILE_COMMANDS "on")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# options
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

suppress_compile_warnings()

option(IMGUI_WS_ALL_WARNINGS            "imgui-ws: enable all compiler warnings" ${IMGUI_WS_STANDALONE})
option(IMGUI_WS_ALL_WARNINGS_3RD_PARTY  "imgui-ws: enable all compiler warnings in 3rd party libs" OFF)

option(IMGUI_WS_SANITIZE_THREAD         "imgui-ws: enable thread sanitizer" OFF)
option(IMGUI_WS_SANITIZE_ADDRESS        "imgui-ws: enable address sanitizer" OFF)
option(IMGUI_WS_SANITIZE_UNDEFINED      "imgui-ws: enable undefined sanitizer" OFF)

option(IMGUI_WS_SUPPORT_SDL2            "imgui-ws: imgui with sdl2" OFF)

option(IMGUI_WS_BUILD_EXAMPLES_NULL     "imgui-ws: build headless examples" ${IMGUI_WS_STANDALONE})
option(IMGUI_WS_BUILD_EXAMPLES_SDL2     "imgui-ws: build sdl2 examples" ${IMGUI_WS_STANDALONE})

option(IMGUI_WS_BUILD_TOOLS             "imgui-ws: build tools" ${IMGUI_WS_STANDALONE})

# sanitizers

if (IMGUI_WS_SANITIZE_THREAD)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=thread")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=thread")
endif()

if (IMGUI_WS_SANITIZE_ADDRESS)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=address -fno-omit-frame-pointer")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer")
endif()

if (IMGUI_WS_SANITIZE_UNDEFINED)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=undefined")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=undefined")
endif()

# dependencies

if (IMGUI_WS_SUPPORT_SDL2)
    find_package(OpenGL REQUIRED)
    find_package(SDL2 REQUIRED)
    string(STRIP "${SDL2_LIBRARIES}" SDL2_LIBRARIES)
endif()

# main

find_package(Threads REQUIRED)

if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "RelWithDebInfo")
endif ()

include(FetchContent)

   # Add Imgui, uWebSockets, incppect, etc. libraries:
   # 
   
   # incppect fetches uWebSockets
   #
   # FetchContent_Declare(
   #    "uwebsockets"
   #    GIT_REPOSITORY "https://github.com/uNetworking/uWebSockets"
   #    GIT_TAG "v20.62.0" 
   #    GIT_SUBMODULES_RECURSE YES
   #    GIT_SHALLOW TRUE
   # )
   # FetchContent_MakeAvailable("uwebsockets")
    
   FetchContent_Declare(
      "incppect"
      GIT_REPOSITORY "https://github.com/openalgz/glue.git"
      GIT_TAG "main"
      GIT_SHALLOW TRUE
   )
   FetchContent_MakeAvailable("incppect")
   
   FetchContent_Declare(
      "imgui"
      GIT_REPOSITORY "https://github.com/ocornut/imgui.git"
      GIT_TAG "v1.90.5"
      GIT_SHALLOW TRUE
      # GIT_PROGRESS FALSE
   )
   FetchContent_MakeAvailable("imgui")

   FetchContent_Declare(
      "implot"
      GIT_REPOSITORY "https://github.com/epezent/implot.git"
      GIT_TAG "v0.16"
      GIT_SHALLOW TRUE
      # GIT_PROGRESS FALSE
   )
   FetchContent_MakeAvailable("implot")

   include_directories(${imgui_SOURCE_DIR})
   include_directories(${implot_SOURCE_DIR})
   
   add_subdirectory(src)

#if (IMGUI_WS_STANDALONE)
    add_subdirectory(examples)
    add_subdirectory(tools)
#endif()
