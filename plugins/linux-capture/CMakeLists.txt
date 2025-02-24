project(linux-capture)

option(ENABLE_PIPEWIRE "Enable PipeWire support" ON)
if(NOT ENABLE_PIPEWIRE)
  message(STATUS "OBS:    -        PipeWire support disabled")
endif()

find_package(X11 REQUIRED)
if(NOT TARGET X11::Xcomposite)
  message(
    FATAL_ERROR "OBS:  DISABLED   linux-capture - Xcomposite library not found")
endif()
find_package(XCB COMPONENTS XCB XFIXES RANDR SHM XINERAMA)

add_library(linux-capture MODULE)
add_library(OBS::capture ALIAS linux-capture)

target_sources(
  linux-capture
  PRIVATE linux-capture.c
          xcursor.c
          xcursor.h
          xcursor-xcb.c
          xcursor-xcb.h
          xhelpers.c
          xhelpers.h
          xshm-input.c
          xcomposite-main.cpp
          xcompcap-main.cpp
          xcompcap-main.hpp
          xcompcap-helper.cpp
          xcompcap-helper.hpp)

target_link_libraries(
  linux-capture
  PRIVATE OBS::libobs
          OBS::obsglad
          X11::X11
          X11::Xfixes
          X11::Xcomposite
          XCB::XCB
          XCB::XFIXES
          XCB::RANDR
          XCB::SHM
          XCB::XINERAMA)

set_target_properties(linux-capture PROPERTIES FOLDER "plugins")

if(ENABLE_PIPEWIRE)
  find_package(PipeWire 0.3.32 QUIET)
  find_package(Gio QUIET)
  find_package(Libdrm QUIET)

  if(NOT TARGET PipeWire::PipeWire)
    message(
      FATAL_ERROR
        "OBS:    -        PipeWire library not found! Please install PipeWire or set ENABLE_PIPEWIRE=OFF"
    )
  elseif(NOT TARGET GIO::GIO)
    message(
      FATAL_ERROR
        "OBS:    -        Gio library not found! Please install GLib2 (or Gio) or set ENABLE_PIPEWIRE=OFF"
    )
  elseif(NOT TARGET Libdrm::Libdrm)
    message(
      FATAL_ERROR
        "OBS:    -        libdrm headers not found! Please install libdrm or set ENABLE_PIPEWIRE=OFF"
    )
  endif()

  target_sources(linux-capture PRIVATE pipewire.c pipewire.h pipewire-capture.c
                                       pipewire-capture.h portal.c portal.h)

  target_link_libraries(linux-capture PRIVATE PipeWire::PipeWire GIO::GIO
                                              Libdrm::Libdrm)
  target_compile_definitions(linux-capture PRIVATE ENABLE_PIPEWIRE)
endif()

setup_plugin_target(linux-capture)
