cmake_minimum_required(VERSION 3.5)
project(serial)

# Find catkin
find_package(ament_cmake REQUIRED)

if(APPLE)
  find_library(IOKIT_LIBRARY IOKit)
  find_library(FOUNDATION_LIBRARY Foundation)
elseif(UNIX AND NOT APPLE)
  set(EXTRA_LIBS rt pthread)
else()
  set(EXTRA_LIBS setupapi)
endif()

## Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)

if(APPLE)
  list(APPEND serial_SRCS src/impl/unix.cc)
  list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
  list(APPEND serial_SRCS src/impl/unix.cc)
  list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
  list(APPEND serial_SRCS src/impl/win.cc)
  list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

# Library target
add_library(serial ${serial_SRCS})

target_include_directories(serial PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

target_link_libraries(serial ${EXTRA_LIBS})

# Example (optional)
add_executable(serial_example examples/serial_example.cc)
target_link_libraries(serial_example serial)

# Install the lib
install(TARGETS serial
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

# Install headers
install(DIRECTORY include/serial
  DESTINATION include
)

# Install example (optional)
install(TARGETS serial_example
  DESTINATION lib/${PROJECT_NAME}
)

# ament package macro
ament_export_include_directories(include)
ament_export_libraries(serial)
ament_package()
