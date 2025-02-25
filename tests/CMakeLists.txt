set(LIBNAME test)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)
# dbus
set(CMAKE_CXX_FLAGS "-fno-access-control -ldl")
add_compile_options(-fsanitize=address)
add_link_options(-fsanitize=address)
add_definitions(-DPREFIX="${CMAKE_PREFIX_PATH}")
add_definitions(-DOS_VERSION_TEST_FILE="/tmp/etc/os-version")
find_package(Qt5 REQUIRED COMPONENTS Gui)
find_package(Qt5 REQUIRED COMPONENTS Core)
if(LINUX)
find_package(Qt5 REQUIRED COMPONENTS DBus)
find_package(PkgConfig REQUIRED)
pkg_check_modules(QGSettings REQUIRED gsettings-qt)
endif()
find_package(Qt5 REQUIRED COMPONENTS Xml)
find_package(Qt5 REQUIRED COMPONENTS Concurrent)
find_package(Qt5 REQUIRED COMPONENTS Test)
find_package(GTest REQUIRED)

# start base 
include(../src/base/base.cmake)

# end base
if(LINUX)
  include(../src/dbus/dbus.cmake)
endif()
#message(${dbus_SRCS})
# end dbus



# start dci
include(../src/dci/dci.cmake)
#end dci

#start filesystem
include(../src/filesystem/filesystem.cmake)
#end filesystem
# start log 
include(../src/log/log.cmake)
#end log
# start settings 
include(../src/settings/settings.cmake)
#message(${settings_SRC})
#end settings

#start utils 

# TODO match linux and others
include(../src/util/util.cmake)
#end utils

#GLOB
include(../src/glob.cmake)

#endglob


# test 
file(GLOB TEST_HEADER
  ut_.*h 
)
file(GLOB TEST_SOURCE
  *.cpp 
)
set(test_SRC 
  ${TEST_HEADER}
  ${TEST_SOURCE}
)
# end test
if(LINUX)
  add_executable(${LIBNAME}
    ${dbus_SRCS}
    ${base_SRCS}
    ${dci_SRCS}
    ${filesystem_SRCS}
    ${log_SRCS}	
    ${settings_SRC}
    ${utils_SRC}
    ${glob_SRC}
    ${test_SRC}
    ./data.qrc
  )
target_link_libraries(
  ${LIBNAME} PRIVATE 
    Qt5::Gui 
    Qt5::Core 
    Qt5::DBus
    Qt5::Xml
    Qt5::Concurrent 
    Qt5::Test
    ${GTEST_LIBRARIES}
    ${QGSettings_LIBRARIES}
    -lpthread
  )
  target_include_directories( ${LIBNAME} PUBLIC
    ${Qt5Core_PRIVATE_INCLUDE_DIRS}
    ${QGSettings_INCULDE_DIRS}
    ../include/util/
    ../include/dci/
    ../include/log/
    ../include/base/
    ../include/base/private/
    ../include/global/
    ../include/DtkCore/
    ../include/settings/
    ../include/filesystem/
    ../include/
  )
else()
  add_executable(${LIBNAME}
    ${base_SRCS}
    ${dci_SRCS}
    ${filesystem_SRCS}
    ${log_SRCS}	
    ${settings_SRC}
    ${utils_SRC}
    ${glob_SRC}
    ${test_SRC}
    ./data.qrc
  )
  target_link_libraries(
    ${LIBNAME} PRIVATE 
    Qt5::Gui
    Qt5::Core
    Qt5::Xml
    Qt5::Concurrent
    Qt5::Test
    ${GTEST_LIBRARIES}
    -lpthread
  )
  target_include_directories( ${LIBNAME} PUBLIC
    ${Qt5Core_PRIVATE_INCLUDE_DIRS}
    ../include/util/
    ../include/dci/
    ../include/log/
    ../include/base/
    ../include/base/private/
    ../include/global/
    ../include/DtkCore/
    ../include/settings/
    ../include/filesystem/
    ../include/
  )
endif()

