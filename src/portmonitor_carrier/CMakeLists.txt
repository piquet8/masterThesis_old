# Copyright (C) 2006-2020 Istituto Italiano di Tecnologia (IIT)
# All rights reserved.
#
# This software may be modified and distributed under the terms of the
# BSD-3-Clause license. See the accompanying LICENSE file for details.

yarp_prepare_plugin(portmonitor2
                    TYPE PortMonitor
                    INCLUDE PortMonitor.h
                    CATEGORY carrier
                    EXTRA_CONFIG SUBTYPE=recv
                                 CODE="not applicable"
                    DEFAULT ON)

if (NOT SKIP_portmonitor2)
  yarp_add_plugin(yarp_portmonitor2)

  target_sources(yarp_portmonitor2 PRIVATE PortMonitor.h
                                          MonitorBinding.h
                                          MonitorEvent.h
                                          MonitorLogComponent.h
                                          PortMonitor.cpp
                                          MonitorBinding.cpp
                                          MonitorLogComponent.cpp
                                          dll/MonitorSharedLib.h
                                          dll/MonitorSharedLib.cpp
                                          DummyConnector2.h
                                          DummyConnector2.cpp)
  if(YARP_HAS_Lua)
    target_sources(yarp_portmonitor2 PRIVATE lua/MonitorLua.cpp
                                            lua/MonitorLua.h)
    if(YARP_HAS_SWIG AND NOT CMAKE_CROSSCOMPILING)
      include(${SWIG_USE_FILE})
      add_custom_command(
        OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/lua/src_gen/swigluarun.h"
        COMMAND "${CMAKE_COMMAND}" -E make_directory "${CMAKE_CURRENT_BINARY_DIR}/lua/src_gen/"
        COMMAND "${SWIG_EXECUTABLE}" -c++ -lua -external-runtime "${CMAKE_CURRENT_BINARY_DIR}/lua/src_gen/swigluarun.h"
        COMMENT "Generating swig-lua runtime")
      target_include_directories(yarp_portmonitor2 PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/lua/src_gen")
      target_sources(yarp_portmonitor2 PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/lua/src_gen/swigluarun.h")
    else()
      target_include_directories(yarp_portmonitor2 PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/lua/src_gen")
    endif()
  endif()

  target_include_directories(yarp_portmonitor2 PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}
                                                      ${CMAKE_CURRENT_SOURCE_DIR}/dll)
  target_link_libraries(yarp_portmonitor2 PRIVATE YARP::YARP_os)
  list(APPEND YARP_${YARP_PLUGIN_MASTER}_PRIVATE_DEPS YARP_os)

  if(YARP_HAS_Lua)
    target_include_directories(yarp_portmonitor2 PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/lua)
    target_include_directories(yarp_portmonitor2 SYSTEM PRIVATE ${LUA_INCLUDE_DIR})
    target_compile_definitions(yarp_portmonitor2 PRIVATE ENABLED_PORTMONITOR_LUA)
    target_link_libraries(yarp_portmonitor2 PRIVATE ${LUA_LIBRARY})
#     list(APPEND YARP_${YARP_PLUGIN_MASTER}_PRIVATE_DEPS Lua) (not using targets)
  endif()

  yarp_install(TARGETS yarp_portmonitor2
               EXPORT YARP_${YARP_PLUGIN_MASTER}
               COMPONENT ${YARP_PLUGIN_MASTER}
               LIBRARY DESTINATION ${YARP_DYNAMIC_PLUGINS_INSTALL_DIR}
               ARCHIVE DESTINATION ${YARP_STATIC_PLUGINS_INSTALL_DIR}
               YARP_INI DESTINATION ${YARP_PLUGIN_MANIFESTS_INSTALL_DIR})

  set(YARP_${YARP_PLUGIN_MASTER}_PRIVATE_DEPS ${YARP_${YARP_PLUGIN_MASTER}_PRIVATE_DEPS} PARENT_SCOPE)

  set_property(TARGET yarp_portmonitor2 PROPERTY FOLDER "Plugins/Carrier")
endif()
