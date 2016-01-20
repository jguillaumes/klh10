#                                               -*- Autoconf -*-
# Process this file with autogen.sh to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([klh10], [2.0i], [https://github.com/Rhialto/klh10])
AC_CONFIG_SRCDIR([src/klh10.h])
AC_CONFIG_HEADERS([src/config.h])

# Filter CFLAGS; the default is something like -g -O2 and we want -g3 -O3.
NEWCFLAGS=""
for cf in $CFLAGS
do
    case "$cf" in
	-O*)					;;
	-g*)					;;
	*)	NEWCFLAGS="$NEWCFLAGS $cf"	;;
    esac
done
CFLAGS="$NEWCFLAGS -g3 -O3"

# Checks for programs.
AC_PROG_CC
AC_PROG_MAKE_SET

# Checks for libraries.
# They may be needed for Solaris:
# -lsocket and -lnsl are needed only for osdnet.c.
# -lrt is needed for nanosleep().
# FIXME: Check if -lnsl is actually still needed for anything
dnl AC_CHECK_LIB([nsl], [main])
AC_CHECK_FUNC([nanosleep],
    [],
    [
	AC_CHECK_LIB([rt], [nanosleep])
    ])

# -lsocket and -lpcap should go into NETLIBS
SAVE_LIBS="$LIBS"
LIBS=""
AC_CHECK_LIB([socket], [if_nameindex])
AC_CHECK_LIB([pcap], [pcap_findalldevs])
# Not all libpcap versions seem to have pcap_set_immediate_mode().
AC_CHECK_LIB([pcap], [pcap_set_immediate_mode],
	     [AC_DEFINE(HAVE_PCAP_SET_IMMEDIATE_MODE, 1, [Set to 1 if we have pcap_set_immediate_mode])]
)
NETLIBS="$LIBS"
LIBS="$SAVE_LIBS"

# Checks for header files.
AC_CHECK_HEADERS([arpa/inet.h fcntl.h limits.h netinet/in.h sgtty.h stddef.h stdlib.h string.h sys/file.h sys/ioctl.h sys/socket.h sys/time.h termios.h unistd.h net/if_tun.h linux/if_tun.h linux/if_packet.h net/if_tap.h sys/mtio.h net/nit.h sys/dlpi.h net/if_dl.h net/if_types.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_C_INLINE
AC_TYPE_OFF_T
AC_TYPE_PID_T
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_CHECK_MEMBERS([struct stat.st_rdev])

# Checks for library functions.
AC_FUNC_ERROR_AT_LINE
AC_FUNC_FORK
AC_FUNC_FSEEKO
AC_FUNC_MALLOC
AC_FUNC_REALLOC
AC_CHECK_FUNCS([alarm dup2 gettimeofday localtime_r memset socket strcasecmp strchr strcspn strerror strncasecmp strpbrk strrchr strtol getifaddrs if_nameindex termios sigsetops sigaction nanosleep])

# Check for CPU.
# This requires install-sh, config.sub, config.guess from automake.
AC_CANONICAL_TARGET
AC_DEFINE_UNQUOTED([KLH10S_CENV_CPU_], "$target_cpu", [String form of the CPU name])
AC_DEFINE_UNQUOTED([KLH10S_CENV_SYS_], "$target_vendor-$target_os", [String form of the host OS])

AC_SUBST([KLH10S_CENV_CPU_], [$target_cpu])
AC_SUBST([KLH10S_CENV_SYS_], [$target_vendor-$target_os])

AC_C_BIGENDIAN([
    AC_DEFINE(CENV_CPUF_BIGEND, 1, [])
    AC_DEFINE(CENV_CPUF_LILEND, 0, [])
],[
    AC_DEFINE(CENV_CPUF_BIGEND, 0, [Set to 1 for big-endian CPU])
    AC_DEFINE(CENV_CPUF_LILEND, 1, [Set to 1 for little-endian CPU])
])

# This test and auxiliary file is needed to get around a bug in the NetBSD
# /usr/include files.  <stdio.h> includes <sys/types.h> which includes
# <machine/types.h> which incorrectly exposes a typedef of vaddr_t (normally
# a kernel only type), thus conflicting with KLH10's vaddr_t.
# By including this file ahead of any other source files (see the -include
# in CENVFLAGS) we can nullify the typedef.
# And while we're at it, blast paddr_t for the same reason.
# As of NetBSD 7.0, the problem has moved to <stdlib.h>.

AC_MSG_CHECKING([whether vaddr_t escapes from <std*.h>])
AC_LANG([C])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <stdio.h>
#include <stdlib.h>
/* see if this defines vaddr_t to something different than before */
typedef char vaddr_t;
]])],
[
    # Compiles ok: nothing to do.
    AC_MSG_RESULT([no (all ok)])
],
[
    # Compile error: we need to hide vaddr_t.
    AC_MSG_RESULT([yes (will patch it)])
    CENVFLAGS="$CENVFLAGS -include vaddr-sucks.h"
])

# -------------------------------------------------------------------------
# Check whether we want bridge support
# -------------------------------------------------------------------------

AC_ARG_ENABLE(bridge,
    AC_HELP_STRING([--disable-bridge],[disable bridge networking]),
    [ case "${enableval}" in
	  yes) bridge=true ;;
	  no)  bridge=false ;;
	  *)   AC_MSG_ERROR(bad value ${enableval} for --enable-bridge) ;;
      esac ],
    [ bridge=true ] )
if test x$bridge = xfalse
then
    AC_DEFINE(KLH10_NET_BRIDGE, 0, [Set to 0 to disable bridge networking])
    echo 'Compiling without bridge support'
fi

# -------------------------------------------------------------------------
# Output section. State which @VARIABLES@ are going to be substituted
# -------------------------------------------------------------------------

AC_SUBST([CC])
AC_SUBST([CFLAGS])
AC_SUBST([DESTDIR])
AC_SUBST([PREFIX])
AC_SUBST([LIBS])
AC_SUBST([NETLIBS])
AC_SUBST([MAKEFILE], [Makefile])
AC_SUBST([CENVFLAGS])

# -------------------------------------------------------------------------
# Output section. State which files are going to be constructed
# -------------------------------------------------------------------------

AC_CONFIG_FILES([bld-kl/Makefile:mk/top.mk:mk/top-kl.mk:src/Makefile.mk
                 bld-ks/Makefile:mk/top.mk:mk/top-ks.mk:src/Makefile.mk
                 bld-ks-its/Makefile:mk/top.mk:mk/top-ks-its.mk:src/Makefile.mk
		])

AC_OUTPUT