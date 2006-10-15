# Makefile for GNU Mach.

# Copyright (C) 2006 Free Software Foundation, Inc.

# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License as
# published by the Free Software Foundation; either version 2, or (at
# your option) any later version.
#
# This program is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 675 Mass Ave, Cambridge, MA 02139, USA.

#
# Various definitions of the Automake environment.
#
# These will be augmented below.
#

SUBDIRS =
DIST_SUBDIRS =
EXTRA_DIST =
noinst_LIBRARIES =
noinst_PROGRAMS =
TESTS=
info_TEXINFOS =
MOSTLYCLEANFILES =
AM_CPPFLAGS =
AM_CCASFLAGS =
AM_CFLAGS =
AM_LDFLAGS =

#
# Compilation flags
#

AM_CPPFLAGS += \
	-nostdinc -imacros config.h

AM_CPPFLAGS += \
	-I$(systype) \
	-I. \
	-I$(top_srcdir)/$(systype) \
	-I$(top_srcdir)/$(systype)/include/mach/sa \
	-I$(top_srcdir)/include

# Yes, this makes the eyes hurt.  But perhaps someone will finally take care of
# all that scruffy Mach code...  Also see <http://savannah.gnu.org/task/?5726>.
AM_CFLAGS += \
	-Wall

# See <http://lists.gnu.org/archive/html/bug-hurd/2006-01/msg00148.html>.
AM_CFLAGS += \
	-fno-strict-aliasing

# For pre Automake 1.10.
AM_CCASFLAGS += \
	$(AM_CPPFLAGS)

#
# MIG Setup.
#

# MIGCOM.
MIGCOM = $(MIG) -cc cat - /dev/null

MIGFLAGS =

# We need this because we use $(CPP) to preprocess MIG .defs files.
CPP = @CPP@ -x c

#
# The main kernel functionality.
#

noinst_LIBRARIES += \
	libkernel.a
libkernel_a_SOURCES =
nodist_libkernel_a_SOURCES =
MOSTLYCLEANFILES += \
	$(nodist_libkernel_a_SOURCES)

gnumach_o_LDADD =
gnumach_o_LDADD += \
	libkernel.a

# Makerules: how to do some things.
include Makerules.am

# Main Makefile fragment.
include Makefrag.am

# Test suite.
include tests/Makefrag.am

# Documentation.
include doc/Makefrag.am

# Linux device drivers and the glue code.
include linux/Makefrag.am

#
# Kernel Image
#

# TODO.  Is the following kosher from a Automake point of view?  (I.e. a
# program `gnumach.o' that is then later used again as an object file.)
gnumach_o_SOURCES =
# TODO.  ``--start-group''.  Want to keep it that way?
# TODO.  ``-u _start''.  System dependent?
gnumach_o_LINK = $(LD) -u _start -r -o $@ --start-group
noinst_PROGRAMS += \
	gnumach.o

# This is the list of routines we decide is OK to steal from the C library.
clib_routines := memcpy memmove memset bcopy bzero \
		 strchr strstr strsep strpbrk strtok \
		 htonl htons ntohl ntohs \
		 etext edata end # actually ld magic, not libc.
gnumach-undef: gnumach.$(OBJEXT)
	$(NM) -u $< | sed 's/  *U  *//;s/^_*//' | sort -u > $@
MOSTLYCLEANFILES += gnumach-undef
gnumach-undef-bad: gnumach-undef
	sed '$(foreach r,$(clib_routines),/^$r$$/d;)' $< > $@
MOSTLYCLEANFILES += gnumach-undef-bad
clib-routines.o: gnumach-undef gnumach-undef-bad
	if test -s gnumach-undef-bad; \
	then cat gnumach-undef-bad; exit 2; else true; fi
	$(CCLD) -nostdlib -nostartfiles -r -static \
	  -o $@ `sed 's/^/-Wl,-u,/' < $<` -x c /dev/null -lc

gnumach_SOURCES =
gnumach_LINK = $(LD) $(LINKFLAGS) $(LINKFLAGS_$@) -o $@
gnumach_LDADD = gnumach.o clib-routines.o

#
# Installation.
#

exec_bootdir = \
	$(exec_prefix)/boot
exec_boot_PROGRAMS = \
	gnumach

#
# Legacy support.
#

install-headers: install-data
	@echo '*****************************************************'
	@echo '* As you can see above, I was so kind to rewrite your'
	@echo '* `make $@'\'
	@echo '* into'
	@echo '* `make $^'\'
	@echo '* which is how it is to be spelled these days.'
	@echo '*'
	@echo '* Please get your instructions fixed.'
	@echo '*****************************************************'
	@echo
	@echo 'Penalty:'
	sleep 20