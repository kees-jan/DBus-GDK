dnl Process this file with autoconf to produce a configure script.
dnl
dnl Copyright (C) 2001 - 2012 Rob Caelers & Raymond Penners
dnl

m4_define([dbus_gdk_major_version], [1])
m4_define([dbus_gdk_minor_version], [9])
m4_define([dbus_gdk_micro_version], [906])
m4_define([dbus_gdk_version_suffix], [0])
m4_define([dbus_gdk_version],
          [dbus_gdk_major_version.dbus_gdk_minor_version.dbus_gdk_micro_version])
m4_define([dbus_gdk_resource_version],
          [dbus_gdk_major_version,dbus_gdk_minor_version,dbus_gdk_micro_version,dbus_gdk_version_suffix])

AC_PREREQ([2.64])
AC_INIT([dbus_gdk],
        [dbus_gdk_version],
        [http://issues.dbus_gdk.org/cgi-bin/bugzilla/enter_bug.cgi],
        [dbus_gdk],
        [http://www.dbus_gdk.org])

AC_CONFIG_SRCDIR([bin/dbusgen.py])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_HEADER([config.h])

AM_INIT_AUTOMAKE([-Wall -Werror foreign silent-rules])
AM_SILENT_RULES([yes])

GNOME_SHELL_VERSION=3.2.1
AC_SUBST(GNOME_SHELL_VERSION)

if test -d $srcdir/.git && test "$TINDERBOX_BUILD" = yes; then
    git_version=`git describe --tags --abbrev=10 --dirty 2>/dev/null`
    AC_DEFINE_UNQUOTED(GIT_VERSION, "$git_version", [GIT Version])
fi


dnl
dnl Command line options.
dnl

AC_ARG_ENABLE(dbus,
              [AS_HELP_STRING([--disable-dbus],
                              [compile without DBus support])])

AC_ARG_ENABLE(debug,
              [AS_HELP_STRING([--enable-debug],
                              [compile with debugging flags set])])


dnl
dnl Checks for programs
dnl

AC_USE_SYSTEM_EXTENSIONS

AC_PROG_CC
AM_PROG_CC_C_O
AC_PROG_CXX
AC_PROG_CPP
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AC_PROG_LIBTOOL
AC_CHECK_PROGS(M4, gm4 m4, m4)

dnl
dnl checks for compiler characteristics
dnl

AC_LANG([C++])
AC_REQUIRE_CPP
AC_C_INLINE
AC_C_CONST


dnl
dnl Headers/Types/Functions
dnl

AC_HEADER_STDC
AC_CHECK_HEADERS([errno.h stdlib.h sys/time.h sys/select.h unistd.h])

dnl Update version and date.
AC_SUBST(VERSION)
ISODATE=`date +%Y-%m-%d`
AC_SUBST(ISODATE)

RESOURCE_VERSION=dbus_gdk_resource_version
AC_SUBST(RESOURCE_VERSION)

AC_CANONICAL_HOST
AC_PREFIX_PROGRAM(dbus_gdk)

dnl<
dnl Debug
dnl

config_debug=no

if test "x$enable_debug" = "xyes"
then
   config_debug=yes
   CFLAGS="${CFLAGS} -O0 -g -Wall"
   CXXFLAGS="${CXXFLAGS} -O0 -g -Wall"
   LDFLAGS="${LDFLAGS} -g"

   AC_DEFINE([GLIBCXX_FORCE_NEW], , [Define for debugging purposes])
   AC_DEFINE([GLIBCPP_FORCE_NEW], , [Define for debugging purposes])
else
   AC_DEFINE([NDEBUG], , [Define if debugging code should be disabled])
fi

dnl
dnl DBus
dnl

config_dbus=no
config_dbus_gio=no
have_python_cheetah=no

PKG_CHECK_MODULES([GIO],
                  [gio-2.0 >= 2.26.0],
                  [config_dbus_gio=yes
		   config_dbus=yes
		   ], [ config_dbus_gio=no])

if test "x$config_dbus_gio" == "xno" -a "x$enable_dbus" != "xno"
then
        PKG_CHECK_MODULES([DBUS],
                          [dbus-1 >= 1.2],
                          [PKG_CHECK_MODULES([DBUSGLIB], [dbus-glib-1], [config_dbus=yes], [true])
                           PKG_CHECK_EXISTS([dbus-glib-1 >= 0.84],
                                         [AC_DEFINE(HAVE_DBUSGLIB_GET_PRIVATE, , [DBus-glib 0.84+ support])])],
                          [true])

        if test "x$config_dbus" != "xyes" -a "x$enable_dbus" = "xyes"; then
            AC_MSG_ERROR([DBus and DBus-glib development headers not found.])
        fi
fi

if test "x$config_dbus" = "xyes"
then
    AC_DEFINE(HAVE_DBUS, , [DBus support])
    if test "x$platform_os_win32" != "xyes"
    then
        AC_DEFINE(HAVE_DBUSGLIB, , [DBus-glib support])
    fi
    if test "x$config_dbus_gio" == "xyes";
    then
        AC_DEFINE(HAVE_DBUS_GIO, 1, [Have GIO DBUS support])
    fi

    AS_AC_EXPAND(DATADIR, "$datadir")
    DBUS_SERVICES_DIR="$DATADIR/dbus-1/services"
    AC_SUBST(DBUS_SERVICES_DIR)
    AC_DEFINE_UNQUOTED(DBUS_SERVICES_DIR, "$DBUS_SERVICES_DIR", [Where services dir for DBUS is])

    
    AC_CHECK_PROG(PYTHON, python, python)
    AC_MSG_CHECKING([for the Cheetah Python package])

    ac_cheetah_result=`$PYTHON -c "from Cheetah.Template import Template" 2>&1`
    if test -z "$ac_cheetah_result"; then
        AC_MSG_RESULT([yes])
        have_python_cheetah=yes
    else
        AC_MSG_ERROR([No, Please install python-cheetah])
    fi
fi

AM_CONDITIONAL(HAVE_DBUS, test x$config_dbus = xyes)
AM_CONDITIONAL(HAVE_DBUS_GIO, test $config_dbus_gio = yes)
AM_CONDITIONAL(HAVE_PYTHON_CHEETAH, test x$have_python_cheetah = xyes)

dnl
dnl GLIB
dnl

PKG_CHECK_MODULES([GLIB],
                  [glib-2.0 >= 2.10 gmodule-2.0 gthread-2.0],
                  [AC_DEFINE(HAVE_GLIB, , [Define if GLib is available])])


dnl
dnl GTK+
dnl

config_gtk=no
config_gtk_version=none

   if test x$config_gtk = xno
   then
      PKG_CHECK_MODULES(GTK,
                        gtk+-2.0 >= 2.6.0 \
                        gdk-2.0 >= 2.8 \
                        sigc++-2.0 \
                        glibmm-2.4 >= 2.10 \
                        gtkmm-2.4 >= 2.10,
                        [config_gtk=yes
			 config_gtk_version=2
                         AC_DEFINE([HAVE_APP_GTK], , [Define if GTK+ is available])])
fi

AM_CONDITIONAL(HAVE_APP_GTKMM, test $config_gtk = yes)
AM_CONDITIONAL(HAVE_GTK3, test "x$config_gtk_version" = "x3")


dnl
dnl Introspection
dnl

#if test "x$platform_os_win32" != "xyes"
#then
    GOBJECT_INTROSPECTION_CHECK([0.6.7])
#fi


dnl
dnl What variant to build
dnl

variant=""

if test "x$platform_os_win32" = "xyes"
then
   variant="$variant win32"
elif test "x$platform_os_osx" = "xyes"
then
   variant="$variant osx unix"
elif test "x$platform_os_unix" = "xyes"
then
   variant="$variant x11 unix"
fi

WR_BACKEND_INCLUDES="-I\$(top_srcdir)/backend/include"
WR_COMMON_INCLUDES="-I\$(top_srcdir)/common/include"
WR_FRONTEND_COMMON_INCLUDES="-I\$(top_srcdir)/frontend/common/include"

WR_LDADD="-ldbus_gdk-backend -ldbus_gdk-frontend-common -ldbus_gdk-common"
WR_LDFLAGS="-L\$(top_builddir)/common/src/ -L\$(top_builddir)/backend/src/ -L\$(top_builddir)/frontend/common/src"

AC_SUBST(WR_LDADD)
AC_SUBST(WR_LDFLAGS)

AC_SUBST(WR_FRONTEND_COMMON_INCLUDES)
AC_SUBST(WR_COMMON_INCLUDES)
AC_SUBST(WR_BACKEND_INCLUDES)



dnl
dnl Makefile output
dnl

AC_CONFIG_FILES([Makefile
				 bin/Makefile
                 include/Makefile
                 m4/Makefile
                 src/Makefile
])

AC_OUTPUT

dnl ========================================================================

echo ""
echo "                        dbus_gdk $VERSION"
echo "                       ==================="
echo ""
echo "                        prefix:   ${prefix}"
echo ""
echo "GObject-Introspection support :   ${found_introspection}"
if test "${config_dbus_gio}" = "yes"; then
echo "                 DBUS support :   ${config_dbus} (GIO)"
else
echo "                 DBUS support :   ${config_dbus}"
fi
echo "                    Debugging :   ${config_debug}"
echo ""
