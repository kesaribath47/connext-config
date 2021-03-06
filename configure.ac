dnl ****************************************************************************
dnl * Copyright 2020 Real-Time Innovations, Inc.                               *
dnl * Author: fabriziobertocci@gmail.com                                       *
dnl ****************************************************************************

AC_INIT(connext-config, 1.0, fabriziobertocci@gmail.com)
AC_CONFIG_SRCDIR([src/connext-config.c])

dnl subdir-objects tells Makefile to place object files in the same dir as sources.
dnl See: https://www.gnu.org/software/automake/manual/html_node/List-of-Automake-options.html
AM_INIT_AUTOMAKE([subdir-objects tar-ustar foreign])

AC_PROG_CC

AC_CONFIG_MACRO_DIRS([m4])

dnl **********************************************************************************************
dnl * Initialize build architecture
dnl * Defines one of the following macros made available to the library
dnl **********************************************************************************************
AC_MSG_CHECKING(the build architecture)
case `uname -s` in
    Darwin)
        myarch="Mac OS X"
        CFLAGS="$CFLAGS -DBUILD_ARCH_MACOS"
        ;;

    Linux)
        myarch="Linux"
        CFLAGS="$CFLAGS -DBUILD_ARCH_LINUX"
        ;;

    CYGWIN_NT-5.1)
        myarch="Cygwin"
	CFLAGS="$CFLAGS -DBUILD_ARCH_CYGWIN"
	LDFLAGS="$LDFLAGS -static-libgcc"
        ;;

    MINGW32_NT-6.1)
        myarch="MINGW"
	CFLAGS="$CFLAGS -DBUILD_ARCH_MINGW -D_WINDOWS -fexceptions"
	LDFLAGS="$LDFLAGS -static-libgcc"
	;;

    *)
        echo "Build architecture not supported"
        false
        ;;
esac

AC_MSG_RESULT($myarch)


dnl **********************************************************************************************
dnl * Debug/Release                                                                              *
dnl **********************************************************************************************
AC_ARG_ENABLE(debug,
[  --enable-debug          enable full debugging information (and disable optimization)],
[
        CFLAGS="-g -O0 -DDEBUG"
],[
        CFLAGS="-O2 -DNDEBUG"
])


dnl **********************************************************************************************
dnl * Enable/disable compiler warnings                                                           *
dnl **********************************************************************************************
AC_ARG_ENABLE(warnings,
[  --enable-warnings      enable compiler warnings during build ],
[
        CFLAGS="$CFLAGS -Wall -Wpointer-arith -Wwrite-strings -Wconversion -Wcast-qual -W"
],[
        CFLAGS="$CFLAGS -W"
])

dnl ****************************************************************************
dnl --with-rticonnextdds
dnl ****************************************************************************
AC_ARG_WITH(rticonnextdds,
[AC_HELP_STRING([--with-rticonnextdds],[specify the location of RTI Connext DDS (optional)])],
[
        if test "${withval}" = "yes"; then
            AC_MSG_ERROR(missing argument for --with-rticonnextdds parameter);
        else
            NDDSHOME="${withval}"
            [[[ ${NDDSHOME} != /* ]]] && NDDSHOME="`pwd`/${NDDSHOME#./}"
            dnl Realpath is not available on Mac, otherwise this would do the job:
            dnl NDDSHOME=`realpath "${withval}"`
        fi
],[
        AC_MSG_RESULT(Checking for NDDSHOME...)
        if test "a${NDDSHOME}a" = "aa"; then
            AC_MSG_ERROR(NDDSHOME not defined. You must specify the --with-rticonnextdds argument);
        else
            NDDSSHOME="${NDDSHOME}"
        fi
])

dnl Verify the provided location
AC_CHECK_FILE("${NDDSHOME}/include/ndds/ndds_c.h", 
[
        AC_MSG_RESULT(Using RTI Connext DDS at ${NDDSHOME})
],[
        AC_MSG_ERROR(RTI Connext DDS not found or NDDSHOME not defined)
])

CFLAGS="$CFLAGS -I${NDDSHOME}/include -I${NDDSHOME}/include/ndds"

AC_OUTPUT(Makefile src/Makefile)

