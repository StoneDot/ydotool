cmake_minimum_required(VERSION 3.0)
project(ydotool)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(PROJECT_VERSION "0.1.5")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_VERSION_MAJOR "0")
set(CPACK_PACKAGE_VERSION_MINOR "1")
set(CPACK_PACKAGE_VERSION_PATCH "5")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Generic Linux command-line automation tool (no X!)")
set(CPACK_PACKAGE_CONTACT "Reimu NotMoe <https://github.com/ReimuNotMoe>")
set(CMAKE_PROJECT_HOMEPAGE_URL "https://github.com/ReimuNotMoe/ydotool")
set(CPACK_DEBIAN_PACKAGE_HOMEPAGE ${CMAKE_PROJECT_HOMEPAGE_URL})
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6 (>= 2.14), libstdc++6 (>= 5.2), libuinputplus (>= 0.1.3), libevdevplus (>= 0.1.0)")
set(CPACK_PACKAGE_FILE_NAME ${PROJECT_NAME}_${PROJECT_VERSION}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE})
set(CPACK_DEBIAN_DEBUGINFO_PACKAGE yes)
set(CPACK_DEBIAN_COMPRESSION_TYPE "xz")
set(CPACK_GENERATOR "DEB;RPM;TXZ")
set(CPACK_SOURCE_GENERATOR "DEB;RPM;TXZ")
include(CPack)

include(GNUInstallDirs)

set(SOURCE_FILES_LIBRARY
        CommonIncludes.hpp
        Library/libydotool.cpp Library/libydotool.hpp
        Library/Tool.hpp Library/Tool.cpp
#        Library/Transport.cpp Library/Transport.hpp
        Library/Instance.cpp Library/Instance.hpp
        Library/Utils.cpp Library/Utils.hpp Tools/Tools.hpp
        Tools/MouseMoveRelative/MouseMoveRelative.hpp Tools/MouseMoveRelative/MouseMoveRelative.cpp
        Tools/MouseUp/MouseUp.hpp Tools/MouseUp/MouseUp.cpp
        Tools/MouseDown/MouseDown.hpp Tools/MouseDown/MouseDown.cpp
        Tools/MouseMove/MouseMove.hpp Tools/MouseMove/MouseMove.cpp
        Tools/Key/Key.hpp Tools/Key/Key.cpp
        Tools/Click/Click.hpp Tools/Click/Click.cpp
        Tools/Type/Type.hpp Tools/Type/Type.cpp
        Tools/Recorder/Recorder.cpp Tools/Recorder/Recorder.hpp
        )
#        Library/Transports/UnixSocket.cpp Library/Transports/UnixSocket.hpp )

set(SOURCE_FILES_DAEMON
        CommonIncludes.hpp
        Daemon/ydotoold.cpp Daemon/ydotoold.hpp)


set(SOURCE_FILES_CLIENT
        CommonIncludes.hpp
        Client/ydotool.cpp Client/ydotool.hpp)

link_libraries(-L.)

if (NOT DEFINED DYNAMIC_BUILD OR (${DYNAMIC_BUILD}))
    add_library(ydotool_library SHARED ${SOURCE_FILES_LIBRARY})
    set_target_properties(ydotool_library PROPERTIES OUTPUT_NAME ydotool)
    set_target_properties(ydotool_library PROPERTIES VERSION ${PROJECT_VERSION})
    set_target_properties(ydotool_library PROPERTIES SOVERSION ${CPACK_PACKAGE_VERSION_MAJOR})

    target_link_libraries(ydotool_library dl boost_program_options uInputPlus evdevPlus)

    add_executable(ydotoold ${SOURCE_FILES_DAEMON})
    target_link_libraries(ydotoold ydotool_library dl pthread boost_program_options uInputPlus evdevPlus)

    add_executable(ydotool ${SOURCE_FILES_CLIENT})
    target_link_libraries(ydotool ydotool_library boost_program_options pthread uInputPlus evdevPlus)
    set_target_properties(ydotool PROPERTIES OUTPUT_NAME ydotool)
    
    install(TARGETS ydotool_library DESTINATION ${CMAKE_INSTALL_LIBDIR})
    install(TARGETS ydotoold ydotool DESTINATION ${CMAKE_INSTALL_BINDIR})
endif()

if (NOT DEFINED STATIC_BUILD OR (${STATIC_BUILD}))
    add_library(ydotool_library_static STATIC ${SOURCE_FILES_LIBRARY})
    set_target_properties(ydotool_library_static PROPERTIES OUTPUT_NAME ydotool)
    target_link_libraries(ydotool_library_static dl boost_program_options uInputPlus evdevPlus)

    add_executable(ydotoold_static ${SOURCE_FILES_DAEMON})
    target_link_libraries(ydotoold_static ydotool_library_static
            -static
            dl boost_program_options uInputPlus evdevPlus
            -Wl,--whole-archive
            pthread
            -Wl,--no-whole-archive)
    set_target_properties(ydotoold_static PROPERTIES OUTPUT_NAME ydotoold_static)

    add_executable(ydotool_static ${SOURCE_FILES_CLIENT})
    target_link_libraries(ydotool_static ydotool_library_static
            -static
            boost_program_options uInputPlus evdevPlus
            -Wl,--whole-archive
            pthread
            -Wl,--no-whole-archive)
    set_target_properties(ydotool_static PROPERTIES OUTPUT_NAME ydotool_static)
    
    install(TARGETS ydotool_library_static DESTINATION ${CMAKE_INSTALL_LIBDIR})
    install(TARGETS ydotoold_static ydotool_static DESTINATION ${CMAKE_INSTALL_BINDIR})
endif()
