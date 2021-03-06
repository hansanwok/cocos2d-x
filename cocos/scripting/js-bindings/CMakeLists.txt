#/****************************************************************************
# Copyright (c) 2014-2017 Chukong Technologies Inc.
#
# http://www.cocos2d-x.org
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
# ****************************************************************************/

set(cocos_root ${COCOS2DX_ROOT_PATH})

#platform
if(WIN32) # Win32
  set(PLATFORM_FOLDER win32)
elseif(APPLE)# osx or ios
  set(PLATFORM_FOLDER mac)
  set(PLATFORM_SRC manual/platform/ios/JavaScriptObjCBridge.mm)
elseif(ANDROID)
  set(PLATFORM_FOLDER android)
  set(PLATFORM_SRC manual/platform/android/CCJavascriptJavaBridge.cpp)
else() # Assume Linux
  set(PLATFORM_FOLDER linux)
endif()

if(ANDROID OR IOS)
  set(PLATFORM_SRC
    auto/jsb_cocos2dx_experimental_webView_auto.cpp
    manual/experimental/jsb_cocos2dx_experimental_webView_manual.cpp
    auto/jsb_cocos2dx_experimental_video_auto.cpp
    manual/experimental/jsb_cocos2dx_experimental_video_manual.cpp
    ${PLATFORM_SRC}
  )
endif()

set(JSBINDING_SRC
  auto/jsb_cocos2dx_auto.cpp
  auto/jsb_cocos2dx_audioengine_auto.cpp
  auto/jsb_cocos2dx_builder_auto.cpp
  auto/jsb_cocos2dx_extension_auto.cpp
  auto/jsb_cocos2dx_network_auto.cpp
  auto/jsb_cocos2dx_spine_auto.cpp
  auto/jsb_cocos2dx_studio_auto.cpp
  auto/jsb_cocos2dx_ui_auto.cpp
  auto/jsb_cocos2dx_3d_auto.cpp
  auto/jsb_cocos2dx_3d_extension_auto.cpp
  auto/jsb_cocos2dx_physics3d_auto.cpp
  auto/jsb_cocos2dx_navmesh_auto.cpp
  auto/js_bindings_chipmunk_functions.cpp
  auto/js_bindings_chipmunk_auto_classes.cpp
  manual/ScriptingCore.cpp
  manual/cocos2d_specifics.cpp
  manual/js_manual_conversions.cpp
  manual/js_bindings_core.cpp
  manual/js_bindings_opengl.cpp
  manual/jsb_opengl_functions.cpp
  manual/jsb_opengl_manual.cpp
  manual/jsb_opengl_registration.cpp
  manual/jsb_event_dispatcher_manual.cpp
  manual/component/CCComponentJS.cpp
  manual/chipmunk/js_bindings_chipmunk_manual.cpp
  manual/chipmunk/js_bindings_chipmunk_registration.cpp
  manual/cocosbuilder/js_bindings_ccbreader.cpp
  manual/cocostudio/jsb_cocos2dx_studio_manual.cpp
  manual/cocostudio/jsb_cocos2dx_studio_conversions.cpp
  manual/extension/jsb_cocos2dx_extension_manual.cpp
  manual/localstorage/js_bindings_system_functions.cpp
  manual/localstorage/js_bindings_system_registration.cpp
  manual/network/XMLHTTPRequest.cpp
  manual/network/jsb_websocket.cpp
  manual/network/jsb_socketio.cpp
  manual/network/js_network_manual.cpp
  manual/spine/jsb_cocos2dx_spine_manual.cpp
  manual/ui/jsb_cocos2dx_ui_manual.cpp
  manual/3d/jsb_cocos2dx_3d_manual.cpp
  manual/physics3d/jsb_cocos2dx_physics3d_manual.cpp
  manual/navmesh/jsb_cocos2dx_navmesh_conversions.cpp
  manual/navmesh/jsb_cocos2dx_navmesh_manual.cpp
  ${cocos_root}/cocos/storage/local-storage/LocalStorage.cpp
  ${PLATFORM_SRC}
)

include_directories(
  ${cocos_root}/external/spidermonkey/include/${PLATFORM_FOLDER}
  ${cocos_root}/cocos
  ${cocos_root}/cocos/platform/${PLATFORM_FOLDER}
  ${cocos_root}/cocos/editor-support
  ${cocos_root}/extensions
  ${cocos_root}/external
  ${CHIPMUNK_INCLUDE_DIRS}
)

add_library(jscocos2d STATIC
  ${JSBINDING_SRC}
)

find_library(SPIDERMONKEY_LIBRARY NAMES js_static mozjs HINTS
  ${cocos_root}/external/spidermonkey/prebuilt/${PLATFORM_FOLDER}/${ARCH_DIR}
  ${cocos_root}/external/spidermonkey/prebuilt/${PLATFORM_FOLDER}/${ANDROID_ABI}
  ${cocos_root}/external/spidermonkey/prebuilt/${PLATFORM_FOLDER}
  NO_CMAKE_FIND_ROOT_PATH
)

message("SPIDERMONKEY_LIBRARY: ${SPIDERMONKEY_LIBRARY}")

cocos_use_pkg(jscocos2d SQLITE3)

# should change to cocos_use_pkg() about,sqlite3 or set some flag
target_link_libraries(jscocos2d
  ${SPIDERMONKEY_LIBRARY}
)

if(APPLE)
  FIND_LIBRARY(GAME_CONTROLLER GameController)
  target_link_libraries(jscocos2d
  ${GAME_CONTROLLER}
  )
endif()

cocos_use_pkg(jscocos2d xxhash)

target_link_libraries(jscocos2d cocos2d)
add_dependencies(jscocos2d cocos2d)

set_target_properties(jscocos2d
    PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
)

if(USE_BULLET)
  include_directories(${BULLET_INCLUDE_DIRS})
endif()

if(XCODE OR VS)
    cocos_mark_code_files("jscocos2d")
endif()