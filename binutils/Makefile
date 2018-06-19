#
# CDDL HEADER START
#
# The contents of this file are subject to the terms of the
# Common Development and Distribution License, Version 1.0 only
# (the "License").  You may not use this file except in compliance
# with the License.
#
# You can obtain a copy of the license at COPYING
# See the License for the specific language governing permissions
# and limitations under the License.
#
# When distributing Covered Code, include this CDDL HEADER in each
# file and include the License file at COPYING.
# If applicable, add the following below this CDDL HEADER, with the
# fields enclosed by brackets "[]" replaced with your own identifying
# information: Portions Copyright [yyyy] [name of copyright owner]
#
# CDDL HEADER END
#
# Copyright 2017, Joyent, Inc.
#

aarch64_VER =		binutils-2.25.1
aarch64_PATCHES =	patches/arm.patch
aarch64_TARBALL =	$(aarch64_VER).tar.bz2
arm_VER =		$(aarch64_VER)
arm_PATCHES =		$(aarch64_PATCHES)
arm_TARBALL =		$(aarch64_TARBALL)
i86pc_VER =		binutils-2.25.1
i86pc_PATCHES =		patches/efi.patch patches/stdio-limit.patch
i86pc_TARBALL =		$(i86pc_VER).tar.bz2

include ../Makefile.defs

AUTOCONF_OPTS = \
	--prefix=/usr/gnu \
	--enable-64-bit-bfd \
	--program-prefix=g

aarch64_AUTOCONF_OPTS += \
	--target=aarch64-unknown-solaris2.11 \
	--enable-ld=no

arm_AUTOCONF_OPTS += \
	--target=arm-unknown-solaris2.11 \
	--enable-ld=no

i86pc_AUTOCONF_OPTS = \
	--target=i386-pc-solaris2.11

AUTOCONF_ENV += \
	CC=gcc \
	CXX=g++ \
	DESTDIR=$(DESTDIR) \
	MAKE=gmake \
	CONFIG_SHELL=/bin/bash

include ../Makefile.targ
include ../Makefile.targ.autoconf
