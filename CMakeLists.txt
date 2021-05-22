cmake_minimum_required (VERSION 3.3)
project(tegra-eeprom VERSION 1.2.0 DESCRIPTION "Library for reading and writing Tegra EEPROMs")

# otherwise the installation directories are empty by default
include(GNUInstallDirs)

add_compile_options (-D_DEFAULT_SOURCE -D_BSD_SOURCE)

find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
endif(CCACHE_FOUND)

set (LIB_SRC src/eeprom.c src/cvm.c src/boardspec.c)
set (APP1_SRC src/tegra-eeprom-tool.c)
set (APP2_SRC src/tegra-boardspec.c)

add_library (tegra-eeprom SHARED ${LIB_SRC})
set_target_properties (tegra-eeprom PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION 1
    PUBLIC_HEADER "include/boardspec.h;include/cvm.h;include/eeprom.h")

add_executable (tegra-eeprom-tool ${APP1_SRC})
add_executable (tegra-boardspec ${APP2_SRC})

target_include_directories(tegra-eeprom
    PRIVATE include
)

target_include_directories(tegra-eeprom-tool
    PRIVATE include
)
target_include_directories(tegra-boardspec
    PRIVATE include
)

target_link_libraries(tegra-eeprom-tool PRIVATE
    tegra-eeprom
    edit
)

target_link_libraries(tegra-boardspec PRIVATE
    tegra-eeprom
    edit
)

configure_file(
    ${PROJECT_SOURCE_DIR}/tegra-eeprom.pc.in
    ${PROJECT_BINARY_DIR}/tegra-eeprom.pc
    @ONLY
)
install (TARGETS tegra-eeprom
         LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
         PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/tegra-eeprom)
install (TARGETS tegra-eeprom-tool DESTINATION ${CMAKE_INSTALL_BINDIR})
install (TARGETS tegra-boardspec DESTINATION ${CMAKE_INSTALL_BINDIR})
install (FILES "${PROJECT_BINARY_DIR}/tegra-eeprom.pc"
         DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig" COMPONENT devel)