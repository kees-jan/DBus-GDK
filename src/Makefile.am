# Process this file with automake to produce Makefile.in
#
# Copyright (C) 2001, 2002, 2003, 2006, 2007, 2008, 2009, 2010, 2011 Rob Caelers & Raymond Penners
#

noinst_LTLIBRARIES = 	libworkrave-common.la

dbussources =
if HAVE_DBUS
if HAVE_DBUS_GIO
dbussources += 		DBus-gio.cc \
			DBusBinding-gio.cc
else
dbussources += 		DBus-freedesktop.cc \
			DBusBinding-freedesktop.cc
endif
endif

libworkrave_common_la_SOURCES =  \
			${dbussources}



libworkrave_common_la_LIBADD = ${ldadd_platform}

AM_CPPFLAGS = 		-W -DWORKRAVE_PKGDATADIR="\"${pkgdatadir}\"" -DWORKRAVE_DATADIR="\"${datadir}\"" \
			-D_XOPEN_SOURCE=600 \
			@WR_COMMON_INCLUDES@ \
			@GLIB_CFLAGS@ @DBUS_CFLAGS@ 

EXTRA_DIST = 		$(wildcard $(srcdir)/*.hh) $(wildcard $(srcdir)/*.h) $(wildcard $(srcdir)/*.icc) locale.inc

