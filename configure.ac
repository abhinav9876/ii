
m4_include([config/ax_check_openssl.m4])
AC_LANG(C)

# Specify where the auxiliary files created by configure should go. The config
# directory is picked so that they don't clutter up more useful directories.
AC_CONFIG_AUX_DIR(config)


# Initialize the automake system
AM_INIT_AUTOMAKE([foreign])
AM_MAINTAINER_MODE
AM_CONFIG_HEADER(src/iqperf_config.h)

AC_CANONICAL_HOST

# Checks for tools: c compiler, ranlib (used for creating static libraries),
# symlinks and libtool
AC_PROG_CC
AC_PROG_RANLIB
AC_PROG_LN_S
AC_PROG_LIBTOOL

# Add -Wall if we are using GCC.
if test "x$GCC" = "xyes"; then
  CFLAGS="$CFLAGS -Wall"
fi

# Check if profiling must be disabled
AC_ARG_ENABLE([profiling],
    AS_HELP_STRING([--disable-profiling], [Disable iqperf profiling binary]),
    [:],
    [enable_profiling=yes])
AM_CONDITIONAL([ENABLE_PROFILING], [test x$enable_profiling = xyes])

# Checks for header files.
AC_HEADER_STDC

# Check for systems which need -lsocket and -lnsl
#AX_LIB_SOCKET_NSL

# Check for the math library (needed by cjson on some platforms)
AC_SEARCH_LIBS(floor, [m], [], [
echo "floor()"
exit 1
])

# On illumos we need -lsocket
AC_SEARCH_LIBS(socket, [socket], [], [
echo "socket()"
exit 1
])

# On illumos inet_ntop in in -lnsl
AC_SEARCH_LIBS(inet_ntop, [nsl], [], [
echo "inet_ntop()"
exit 1
])

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST

# Check for poll.h (it's in POSIX so everyone should have it?)
AC_CHECK_HEADERS([poll.h])

# Check for SCTP support
AC_CHECK_HEADERS([sys/socket.h])
AC_CHECK_HEADERS([netinet/sctp.h],
		 AC_DEFINE([HAVE_SCTP], [1], [Have SCTP support.])
		 AC_SEARCH_LIBS(sctp_bindx, [sctp])
		 AC_CHECK_TYPES([struct sctp_assoc_value], [], [],
				[[#include <netinet/sctp.h>]]),
		 [],
		 [#ifdef HAVE_SYS_SOCKET_H
#include <sys/socket.h>
#endif
])

AC_CHECK_HEADER([endian.h],
		AC_DEFINE([HAVE_ENDIAN_H], [1], [Define to 1 if you have the <endian.h> header file.]),
		AC_CHECK_HEADER([sys/endian.h],
				AC_DEFINE([HAVE_SYS_ENDIAN_H], [1], [Define to 1 if you have the <sys/endian.h> header file.]),
				AC_MSG_WARN([Couldn't find endian.h or sys/endian.h files: doing compile-time tests.])
				)
		)

if test "x$with_openssl" = "xno"; then
    AC_MSG_WARN( [Building without OpenSSL; disabling iqperf_auth functionality.] )
else
    # Check for OPENSSL support
    AX_CHECK_OPENSSL(
        [ AC_DEFINE([HAVE_SSL], [1], [OpenSSL Is Available]) ],
	[ if test "x$with_openssl" != "x"; then
	  AC_MSG_FAILURE([--with-openssl was given, but test for OpenSSL failed])
	  fi ]
    )
    LDFLAGS="$LDFLAGS $OPENSSL_LDFLAGS"
    LIBS="$OPENSSL_LIBS $LIBS"
    CPPFLAGS="$OPENSSL_INCLUDES $CPPFLAGS"
fi

# Check for TCP_CONGESTION sockopt (believed to be Linux and FreeBSD only)
AC_CACHE_CHECK([TCP_CONGESTION socket option],
[iqperf3_cv_header_tcp_congestion],
AC_EGREP_CPP(yes,
[#include <netinet/tcp.h>
#ifdef TCP_CONGESTION
  yes
#endif
],iqperf3_cv_header_tcp_congestion=yes,iqperf3_cv_header_tcp_congestion=no))
if test "x$iqperf3_cv_header_tcp_congestion" = "xyes"; then
    AC_DEFINE([HAVE_TCP_CONGESTION], [1], [Have TCP_CONGESTION sockopt.])
fi

# Check for IPv6 flowlabel support (believed to be Linux only)
# We check for IPV6_FLOWLABEL_MGR in <linux/in6.h> even though we
# don't use that file directly (we have our own stripped-down
# copy, see src/flowlabel.h for more details).
AC_CACHE_CHECK([IPv6 flowlabel support],
[iqperf3_cv_header_flowlabel],
AC_EGREP_CPP(yes,
[#include <sys/types.h>
#include <linux/in6.h>
#ifdef IPV6_FLOWLABEL_MGR
  yes
#endif
],iqperf3_cv_header_flowlabel=yes,iqperf3_cv_header_flowlabel=no))
if test "x$iqperf3_cv_header_flowlabel" = "xyes"; then
    AC_DEFINE([HAVE_FLOWLABEL], [1], [Have IPv6 flowlabel support.])
fi

# Check for CPU affinity support.  FreeBSD and Linux do this differently
# unfortunately so we have to check separately for each of them.
# FreeBSD uses cpuset_setaffinity while Linux uses sched_setaffinity.
# Define HAVE_CPU_AFFINITY to indicate the CPU affinity setting as a
# generic concept is available.
AC_CHECK_FUNCS([cpuset_setaffinity sched_setaffinity SetProcessAffinityMask],
	       AC_DEFINE([HAVE_CPU_AFFINITY], [1],
	 	         [Have CPU affinity support.]))

# Check for daemon().  Most systems have this but a few (IRIX) don't.
AC_CHECK_FUNCS([daemon])

# Check for sendfile support.  FreeBSD, Linux, and MacOS all support
# this system call, but they're all different in terms of what headers
# it needs and what arguments it expects.
AC_CHECK_FUNCS([sendfile])

# Check for getline support, used as a part of authenticated
# connections.
AC_CHECK_FUNCS([getline])

# Check for packet pacing socket option (Linux only for now).
AC_CACHE_CHECK([SO_MAX_PACING_RATE socket option],
[iqperf3_cv_header_so_max_pacing_rate],
AC_EGREP_CPP(yes,
[#include <sys/socket.h>
#ifdef SO_MAX_PACING_RATE
  yes
#endif
],iqperf3_cv_header_so_max_pacing_rate=yes,iqperf3_cv_header_so_max_pacing_rate=no))
if test "x$iqperf3_cv_header_so_max_pacing_rate" = "xyes"; then
    AC_DEFINE([HAVE_SO_MAX_PACING_RATE], [1], [Have SO_MAX_PACING_RATE sockopt.])
fi


AC_OUTPUT([Makefile src/Makefile src/version.h examples/Makefile iqperf3.spec])
