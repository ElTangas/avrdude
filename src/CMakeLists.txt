#
# CMakeLists.txt - CMake project for AVRDUDE
# Copyright (C) 2021 Marius Greuel
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program. If not, see <http://www.gnu.org/licenses/>.
#

# =====================================
# Set up flex target
# =====================================

FLEX_TARGET(Parser "lexer.l" "${PROJECT_BINARY_DIR}/lexer.c")

if (FLEX_VERSION VERSION_GREATER_EQUAL 2.5.9)
    set(HAVE_YYLEX_DESTROY 1)
endif()

# =====================================
# Set up yacc/bison target
# =====================================

if(BISON_FOUND)
    BISON_TARGET(Parser config_gram.y "${PROJECT_BINARY_DIR}/config_gram.c" DEFINES_FILE "${PROJECT_BINARY_DIR}/config_gram.h")
else()
    set(YACC_TARGET_outputs "${PROJECT_BINARY_DIR}/config_gram.c")
    add_custom_command(OUTPUT ${YACC_TARGET_outputs}
        COMMAND ${YACC_EXECUTABLE} -d -o ${YACC_TARGET_outputs} config_gram.y
        VERBATIM
        COMMENT "[YACC][Parser] Building parser with yacc"
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        )
    set(BISON_Parser_OUTPUTS ${YACC_TARGET_outputs})
endif()

# =====================================
# Setup target specific options
# =====================================

include_directories(BEFORE ${CMAKE_CURRENT_BINARY_DIR})
add_compile_definitions(CONFIG_DIR=\"${CONFIG_DIR}\")

if(WIN32)
    set(EXTRA_WINDOWS_RESOURCES "${PROJECT_BINARY_DIR}/src/windows.rc")
    set(EXTRA_WINDOWS_LIBRARIES setupapi ws2_32)
endif()

if(MSVC)
    enable_language(CXX)

    add_compile_definitions(_CRT_SECURE_NO_WARNINGS=1)
    add_compile_definitions(_CRT_NONSTDC_NO_WARNINGS=1)
    add_compile_definitions(_WINSOCK_DEPRECATED_NO_WARNINGS=1)
    add_compile_options(/W3)
    add_compile_options(/wd4018) # warning C4018: signed/unsigned mismatch
    add_compile_options(/wd4244) # warning C4244: conversion from '...' to '...', possible loss of data
    add_compile_options(/wd4267) # warning C4267: conversion from '...' to '...', possible loss of data
    add_compile_options(/wd5105) # warning C5105: macro expansion producing 'xxx' has undefined behavior
    add_compile_options(/wd6255) # warning C6255: _alloca indicates failure by raising a stack overflow exception. Consider using _malloca instead

    set(EXTRA_WINDOWS_SOURCES ${EXTRA_WINDOWS_SOURCES}
        "msvc/getopt.c"
        "msvc/gettimeofday.c"
        "msvc/usleep.cpp"
        )
    set(EXTRA_WINDOWS_INCLUDES ${EXTRA_WINDOWS_INCLUDES}
        "msvc"
        )
else()
    set(LIB_MATH m)
    add_compile_options(-Wall) # -Wextra
endif()

# =====================================
# Configure files
# =====================================

macro(configure_option option)
    if(${${option}})
        string(REGEX REPLACE "(.*)@${option}_BEGIN@(.*)@${option}_END@(.*)" "\\1\\2\\3" conf_file "${conf_file}")
    else()
        string(REGEX REPLACE "(.*)@${option}_BEGIN@(.*)@${option}_END@(.*)" "\\1\\3" conf_file "${conf_file}")
    endif()
endmacro()

file(READ avrdude.conf.in conf_file)
configure_option(HAVE_PARPORT)
configure_option(HAVE_LINUXGPIO)
configure_option(HAVE_LINUXSPI)
file(WRITE "${PROJECT_BINARY_DIR}/avrdude.conf.in" "${conf_file}")

configure_file(cmake_config.h.in ac_cfg.h)
configure_file("${PROJECT_BINARY_DIR}/avrdude.conf.in" avrdude.conf)
configure_file(avrdude.spec.in avrdude.spec)
if(WIN32)
    configure_file(windows.rc.in windows.rc)
endif()

# =====================================
# Project
# =====================================

add_library(libavrdude
    ac_cfg.h
    arduino.h
    arduino.c
    avr.c
    avr910.c
    avr910.h
    avrdude.h
    avrftdi.c
    avrftdi.h
    avrftdi_private.h
    avrftdi_tpi.c
    avrftdi_tpi.h
    avrpart.c
    bitbang.c
    bitbang.h
    buspirate.c
    buspirate.h
    butterfly.c
    butterfly.h
    config.c
    config.h
    confwin.c
    crc16.c
    crc16.h
    dfu.c
    dfu.h
    fileio.c
    flip1.c
    flip1.h
    flip2.c
    flip2.h
    freebsd_ppi.h
    ft245r.c
    ft245r.h
    jtagmkI.c
    jtagmkI.h
    jtagmkI_private.h
    jtagmkII.c
    jtagmkII.h
    jtagmkII_private.h
    jtag3.c
    jtag3.h
    jtag3_private.h
    libavrdude.h
    linuxgpio.c
    linuxgpio.h
    linuxspi.c
    linuxspi.h
    linux_ppdev.h
    lists.c
    micronucleus.c
    micronucleus.h
    par.c
    par.h
    pgm.c
    pgm_type.c
    pickit2.c
    pickit2.h
    pindefs.c
    ppi.c
    ppi.h
    ppiwin.c
    serbb.h
    serbb_posix.c
    serbb_win32.c
    ser_avrdoper.c
    ser_posix.c
    ser_win32.c
    serialupdi.c
    serialupdi.h
    solaris_ecpp.h
    stk500.c
    stk500.h
    stk500_private.h
    stk500v2.c
    stk500v2.h
    stk500v2_private.h
    stk500generic.c
    stk500generic.h
    teensy.c
    teensy.h
    tpi.h
    updi_constants.h
    updi_link.c
    updi_link.h
    updi_nvm.c
    updi_nvm.h
    updi_readwrite.c
    updi_readwrite.h
    updi_state.c
    updi_state.h
    usbasp.c
    usbasp.h
    usbdevs.h
    usb_hidapi.c
    usb_libusb.c
    usbtiny.h
    usbtiny.c
    update.c
    wiring.h
    wiring.c
    xbee.h
    xbee.c
    ${FLEX_Parser_OUTPUTS}
    ${BISON_Parser_OUTPUTS}
    "${EXTRA_WINDOWS_SOURCES}"
    )
    
set_target_properties(libavrdude PROPERTIES
    PREFIX ""
    PUBLIC_HEADER "libavrdude.h"
    VERSION 1.0.0
    SOVERSION 1
    )

target_include_directories(libavrdude
    PUBLIC
    "${PROJECT_SOURCE_DIR}"
    "${PROJECT_BINARY_DIR}"
    "${LIBUSB_COMPAT_DIR}"
    "${EXTRA_WINDOWS_INCLUDES}"
    )

target_link_libraries(libavrdude
    PUBLIC
    ${LIB_MATH}
    ${LIB_LIBELF}
    ${LIB_LIBUSB}
    ${LIB_LIBUSB_1_0}
    ${LIB_LIBHID}
    ${LIB_LIBHIDAPI}
    ${LIB_LIBFTDI}
    ${LIB_LIBFTDI1}
    ${LIB_LIBREADLINE}
    ${EXTRA_WINDOWS_LIBRARIES}
    )

add_executable(avrdude
    main.c
    term.c
    term.h
    developer_opts.c
    developer_opts.h
    developer_opts_private.h
    whereami.c
    whereami.h
    "${EXTRA_WINDOWS_RESOURCES}"
    )

target_link_libraries(avrdude PUBLIC libavrdude)

# =====================================
# Install
# =====================================

install(TARGETS avrdude DESTINATION bin)
install(TARGETS libavrdude
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION include COMPONENT dev
    )
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/avrdude.conf" TYPE SYSCONF)
install(FILES avrdude.1 TYPE MAN)
