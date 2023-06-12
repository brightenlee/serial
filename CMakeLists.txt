cmake_minimum_required(VERSION 3.5)
project(serial)

find_package(ament_cmake REQUIRED)

# Add serial library
add_library(${PROJECT_NAME}
  src/serial.cc
  include/serial.h
  include/v8stdint.h
)

if(APPLE) # macOS
  find_library(IOKIT_LIBRARY IOKit)
  find_library(FOUNDATION_LIBRARY Foundation)
  target_sources(${PROJECT_NAME} PRIVATE
    src/impl/unix.cc
    src/impl/list_ports/list_ports_osx.cc
  )
  target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX) # .*nix
  target_sources(${PROJECT_NAME} PRIVATE
    src/impl/unix.cc
    src/impl/list_ports/list_ports_linux.cc
  )
  target_link_libraries(${PROJECT_NAME} rt pthread)
elseif(WIN32) # Windows
  target_sources(${PROJECT_NAME} PRIVATE
    src/impl/win.cc
    src/impl/list_ports/list_ports_win.cc
  )
  target_link_libraries(${PROJECT_NAME} setupapi)
  ament_export_libraries(setupapi)
endif()

target_include_directories(${PROJECT_NAME} PRIVATE include)

## Uncomment for example
# add_executable(serial_example examples/serial_example.cc)
# add_dependencies(serial_example ${PROJECT_NAME})
# target_link_libraries(serial_example ${PROJECT_NAME})

## Include headers
include_directories(include)

## Install executable
install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

## Install headers
install(FILES include/serial/serial.h include/serial/v8stdint.h
  DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}/serial)

ament_package()
