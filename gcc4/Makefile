#
# CDDL HEADER START
#
# The contents of this file are subject to the terms of the
# Common Development and Distribution License (the "License").
# You may not use this file except in compliance with the License.
#
# You can obtain a copy of the license at usr/src/OPENSOLARIS.LICENSE
# or http://www.opensolaris.org/os/licensing.
# See the License for the specific language governing permissions
# and limitations under the License.
#
# When distributing Covered Code, include this CDDL HEADER in each
# file and include the License file at usr/src/OPENSOLARIS.LICENSE.
# If applicable, add the following below this CDDL HEADER, with the
# fields enclosed by brackets "[]" replaced with your own identifying
# information: Portions Copyright [yyyy] [name of copyright owner]
#
# CDDL HEADER END
#
# Copyright 2010 Sun Microsystems, Inc.  All rights reserved.
# Use is subject to license terms.
# Copyright (c) 2013, Joyent, Inc.  All rights reserved.
#

aarch64_VER =		gcc-4.9.4
aarch64_PATCHES =	Patches/arm.diff
aarch64_TARBALL =	$(aarch64_VER).tar.bz2
arm_VER =		$(aarch64_VER)
arm_PATCHES =		$(aarch64_PATCHES)
arm_TARBALL =		$(aarch64_TARBALL)
i86pc_VER =		gcc-4.4.4
i86pc_PATCHES =		Patches/mpfr-intree.diff

MPC_VER =	mpc-1.0.3
MPFR_VER =	mpfr-3.1.2
GMP_VER =	gmp-5.1.1
NEWLIB_VER =	newlib-2.5.0

include ../Makefile.defs

SEPARATE_BUILD =	yes

GCC =	/opt/local/bin/gcc
ifeq (X$(ARCH), $(filter X$(ARCH), Xaarch64 Xarm))
GXX =	/opt/local/bin/g++	# required for libcpp
else
GXX =	false
CFLAGS =	-nostdinc -isystem /usr/include -g -O2
endif

arm_common_AUTOCONF_OPTS = \
	--with-newlib \
	--without-headers \
	--disable-multilib \
	--enable-languages="c" \
	--disable-libssp \
	--disable-libada \
	--disable-libgomp \
	--disable-threads \
	--enable-frame-pointer \
	--enable-checking=yes

aarch64_AUTOCONF_OPTS = \
	--target=aarch64-unknown-solaris2.11 \
	$(arm_common_AUTOCONF_OPTS)

arm_AUTOCONF_OPTS = \
	--target=arm-unknown-solaris2.11 \
	--with-arch=armv6 \
	--with-fpu=vfp \
	--with-float=hard \
	--with-abi=aapcs-linux \
	--with-mode=arm \
	$(arm_common_AUTOCONF_OPTS)

i86pc_AUTOCONF_OPTS = \
	--enable-languages="c,c++" \
	--target=i386-pc-solaris2.11

AUTOCONF_OPTS += \
	--with-ld=/usr/bin/ld \
	--without-gnu-ld \
	--with-gnu-as \
	--with-as=$(DESTDIR)/usr/gnu/bin/gas \
	--enable-shared \
	--disable-nls

AUTOCONF_ENV += \
	DESTDIR=$(DESTDIR) \
	MAKE=$(MAKE)

aarch64_AUTOCONF_ENV = \
	AR_FOR_TARGET=$(DESTDIR)/usr/gnu/bin/gar \
	NM_FOR_TARGET=$(DESTDIR)/usr/gnu/bin/gnm \
	OBJDUMP_FOR_TARGET=$(DESTDIR)/usr/gnu/bin/gobjdump \
	RANLIB_FOR_TARGET=$(DESTDIR)/usr/gnu/bin/granlib

arm_AUTOCONF_ENV = $(aarch64_AUTOCONF_ENV)

OVERRIDES += \
	$(AUTOCONF_CFLAGS) \
	STAGE1_CFLAGS="$(CFLAGS)" \
	CFLAGS_FOR_TARGET="$(CFLAGS)"

#
# Because we use a different build directory, we need to make sure we
# catch that when cleaning.
#
CLEANFILES += $(VER)-$(ARCH)-32.build $(VER)-$(ARCH)-32strap.build

#
# The runtime libraries that we build into proto.strap should be made to search
# for their dependencies there as well, so that programs built to run on the
# build machine (from proto.strap) find the correct copies of these libraries.
#
ifeq ($(STRAP),strap)
i86pc_STRAPFIX_LINKS= \
	$(DESTDIR)/lib/64 \
	$(DESTDIR)/usr/lib/64
i86pc_STRAPFIX_FILES_32 = \
	$(DESTDIR)/usr/lib/libgcc_s.so.1 \
	$(DESTDIR)/usr/lib/libstdc++.so.6
i86pc_STRAPFIX_FILES_64 = \
	$(DESTDIR)/usr/lib/64/libgcc_s.so.1 \
	$(DESTDIR)/usr/lib/64/libstdc++.so.6
i86pc_STRAPFIX_FILES = $(i86pc_STRAPFIX_FILES_32) $(i86pc_STRAPFIX_FILES_64)
endif

#
# Unlike everything else, gcc is built to be a cross-compiler, really.  It
# never runs on the target system, only the build system.  So it should not
# be using the proto area's headers or libraries.  Its own libraries will be
# built by the new compiler, and generally aren't using system headers anyway.
# This is not really completely true; for example, libstdc++, and this will
# need more work to be really right.  We only deliver the libraries.
#
AUTOCONF_CPPFLAGS =
AUTOCONF_LIBS =
GENLDFLAGS =

ifeq (X$(ARCH), $(filter X$(ARCH), Xaarch64 Xarm))
ALL_TGT =	all-gcc all-target-libgcc
INSTALL_TGT =	install-gcc install-target-libgcc
else
ALL_TGT =	bootstrap
endif

include ../Makefile.targ
include ../Makefile.targ.autoconf

#
# We could use the mpfr and gmp from the build system (since the compiler will
# be run here), but there's no good way to make gcc build itself correctly if
# we do.  There are two main problems:
#
# 1. Stage2 and later cc1 and company don't honour any combination of LDFLAGS,
# LDFLAGS_FOR_TARGET, BOOT_LDFLAGS, STAGE2_LDFLAGS, etc.  So the only ways to
# get these things built with -R/opt/local/lib are to use LD_OPTIONS which will
# yield the wrong results for libraries or to hack configure.
#
# 2. Even if we could get -R/opt/local/lib into these programs without building
# the libraries incorrectly, it would still be wrong.  The pkgsrc lib directory
# on many systems contains libgcc_s.so.1; however, cc1 was built by the stage2
# compiler and should find the libgcc that matches that compiler.  That happens
# to be the one we just built, not the pkgsrc one.
#
# So Instead we have mpfr and gmp built as part of the gcc build itself.
#
$(AUTOCONF_OUT.32): $(VER.32)/mpc $(VER.32)/mpfr $(VER.32)/gmp $(VER.32)/newlib

$(VER.32)/mpc: $(MPC_VER).tar.gz | $(VER.32)
	-rm -rf $@
	mkdir -p .unpack32
	gtar x -C .unpack32 -z --no-same-owner -f $(MPC_VER).tar.gz
	mv -f .unpack32/$(MPC_VER) ./$@
	-rmdir .unpack32
	chmod 755 $@/configure
	touch $@/configure

$(VER.32)/mpfr: $(MPFR_VER).tar.gz | $(VER.32)
	-rm -rf $@
	mkdir -p .unpack32
	gtar x -C .unpack32 -z --no-same-owner -f $(MPFR_VER).tar.gz
	mv -f .unpack32/$(MPFR_VER) ./$@
	-rmdir .unpack32
	chmod 755 $@/configure
	touch $@/configure

$(VER.32)/gmp: $(GMP_VER).tar.bz2 | $(VER.32)
	-rm -rf $@
	mkdir -p .unpack32
	gtar x -C .unpack32 -j --no-same-owner -f $(GMP_VER).tar.bz2
	mv -f .unpack32/$(GMP_VER) ./$@
	-rmdir .unpack32
	chmod 755 $@/configure
	touch $@/configure

$(VER.32)/newlib: $(NEWLIB_VER).tar.gz | $(VER.32)
	-rm -rf $@
	mkdir -p .unpack32
	gtar x -C .unpack32 -z --no-same-owner -f $(NEWLIB_VER).tar.gz
	mv -f .unpack32/$(NEWLIB_VER)/newlib ./$@
	-rm -rf .unpack32
	chmod 755 $@/configure
	touch $@/configure

.PHONY: strapfix
strapfix: $($(ARCH)_STRAPFIX_LINKS) $($(ARCH)_STRAPFIX_FILES) | install
	for f in $($(ARCH)_STRAPFIX_FILES_32); do \
		/usr/bin/elfedit -e \
		    'dyn:runpath $(DESTDIR)/lib:$(DESTDIR)/usr/lib' \
		    $$f; \
	done
	for f in $($(ARCH)_STRAPFIX_FILES_64); do \
		/usr/bin/elfedit -e \
		    'dyn:runpath $(DESTDIR)/lib/64:$(DESTDIR)/usr/lib/64' \
		    $$f; \
	done

$($(ARCH)_STRAPFIX_LINKS):
	mkdir -p `dirname $@`
	ln -s amd64 $@
