#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([libdnet], [1.12])
AC_CONFIG_AUX_DIR(config)
AC_CONFIG_HEADERS([include/config.h])
AC_CONFIG_SRCDIR([include/dnet.h])
AC_SUBST(ac_aux_dir)
AM_INIT_AUTOMAKE
AC_CONFIG_MACRO_DIRS([m4])

dnl XXX - stop the insanity!@#$
AM_MAINTAINER_MODE

dnl Check for system type.
dnl XXX - we do this to qualify our later feature checks, since some
dnl systems claim to support multiple features, but are quite b0rked.
AC_CANONICAL_HOST

dnl XXX - spoof AC_CYGWIN
case "$host_os" in
  *cygwin*) CYGWIN=yes;;
  *) CYGWIN=no;;
esac

dnl dnl Initialize prefix.
dnl dnl  if test "$prefix" = "NONE"; then
dnl dnl   prefix="/usr/local"
dnl dnl fi

# Checks for programs.
AC_LIBTOOL_DLOPEN
AC_PROG_CXX
AC_PROG_AWK
AC_PROG_CC
AC_PROG_CPP
AC_PROG_INSTALL
AM_PROG_LIBTOOL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AC_PROG_RANLIB

dnl Checks for Python.
dnl XXX - use AM_PATH_PYTHON after automake upgrade
AC_MSG_CHECKING(for Python)
AC_ARG_WITH(python,
[  --with-python=DIR       build Python module (using python in DIR)],
[ case "$withval" in
  yes)
     AC_MSG_RESULT(yes)
     PYTHON="python"
     ;;
  no)
     AC_MSG_RESULT(no)
     ;;
  *)
     AC_MSG_RESULT($withval)
     for subdir in . bin; do
        if test -x $withval/$subdir/python; then
	   owd=`pwd`
	   if cd $withval/$subdir; then withval=`pwd`; cd $owd; fi
	   PYTHON="$withval/python"
	   break
	fi
     done
     if test "x$PYTHON" = "x"; then
        AC_ERROR(python not found in $withval)
     fi
     ;;
  esac 
])
AC_SUBST(PYTHON)
AC_SUBST(TCLINC)
AC_SUBST(TCLLIB)
AM_CONDITIONAL(PYTHON, [test "x$PYTHON" != "x"])
AM_CONDITIONAL(TCL, [test "x$TCLINC" != "x"])

dnl XXX - stupid IRIX cpp
if test -r /usr/include/sgidefs.h ; then
	CPPFLAGS="$CPPFLAGS -D__sgi"
fi

dnl XXX - we need MingW32 under Cygwin for win32
if test "$CYGWIN" = yes ; then
	if test -d /usr/include/mingw ; then
		CPPFLAGS="$CPPFLAGS -mno-cygwin"
 		CFLAGS="$CFLAGS -mno-cygwin"
		AC_DEFINE(WIN32_LEAN_AND_MEAN, 1, 
		    [Define for faster code generation.])
		AC_CHECK_LIB(ws2_32, main)
		AC_CHECK_LIB(iphlpapi, main)
		AC_DEFINE(snprintf, _snprintf, 
		    [Use MingW32's internal snprintf])
	else
		AC_MSG_ERROR([need MingW32 package to build under Cygwin])
	fi
	AC_MSG_CHECKING(for WinPcap developer's pack)
	AC_ARG_WITH(wpdpack,
	[  --with-wpdpack=DIR      use WinPcap developer's pack in DIR],
	[ AC_MSG_RESULT($withval)
	  if test -f $withval/include/packet32.h -a -f $withval/lib/packet.lib; then
	     owd=`pwd`
	     if cd $withval; then withval=`pwd`; cd $owd; fi
	     CFLAGS="$CFLAGS -I$withval/include"
	     LIBS="$LIBS -L$withval/lib -lpacket"
	  else
	     AC_MSG_ERROR(packet32.h or packet.lib not found in $withval)
	  fi ],
	[ for dir in ${prefix} ${HOME}/WPdpack ; do
	     if test -f ${dir}/include/packet32.h -a -f ${dir}/lib/packet.lib; then
	        CFLAGS="$CFLAGS -I${dir}/include"
		LIBS="$LIBS -L${dir}/lib -lpacket"
		have_pcap=yes
		break;
	     fi
	  done
	  if test "$have_pcap" != yes; then
	     AC_MSG_ERROR(WinPcap developer's pack not found)
	  fi
	  AC_MSG_RESULT(yes) ])
fi

dnl Checks for libraries.
if test "$CYGWIN" != yes ; then
	AC_LBL_LIBRARY_NET
	AC_CHECK_LIB(nm, open_mib)
fi

dnl Checks for Check.
AC_MSG_CHECKING(for Check)
AC_ARG_WITH(check,
[  --with-check=DIR        use Check (http://check.sf.net) in DIR],
[ case "$withval" in
  yes|no)
     AC_MSG_RESULT(no)
     ;;
  *)
     AC_MSG_RESULT($withval)
     if test -f $withval/include/check.h -a -f $withval/lib/libcheck.a; then
        owd=`pwd`
        if cd $withval; then withval=`pwd`; cd $owd; fi
        CHECKINC="-I$withval/include"
        CHECKLIB="-L$withval/lib -lcheck"
     elif test -f $withval/src/check.h -a -f $withval/src/libcheck.a; then
        owd=`pwd`
        if cd $withval; then withval=`pwd`; cd $owd; fi
        CHECKINC="-I$withval/src"
        CHECKLIB="-L$withval/src -lcheck"
     else
        AC_ERROR(check.h or libcheck.a not found in $withval)
     fi
     ;;
  esac ],
[ if test -f ${prefix}/include/check.h -a -f ${prefix}/lib/libcheck.a; then
     CHECKINC="-I${prefix}/include"
     CHECKLIB="-L${prefix}/lib -lcheck"
     AC_MSG_RESULT(yes)
  else
     AC_MSG_RESULT(no)
  fi
])
AC_SUBST(CHECKINC)
AC_SUBST(CHECKLIB)
AM_CONDITIONAL(HAVE_CHECK, test "x$CHECKLIB" != "x")

# Checks for libraries.

# Checks for header files.
AC_HEADER_STDC
if test "$CYGWIN" = yes ; then
	AC_CHECK_HEADERS(Iphlpapi.h winsock2.h)
else
	AC_CHECK_HEADERS(fcntl.h unistd.h)
	AC_CHECK_HEADERS(sys/ioctl.h sys/mib.h sys/ndd_var.h sys/socket.h sys/sockio.h \
	    sys/sysctl.h sys/time.h)
	AC_CHECK_HEADERS(net/bpf.h net/if.h net/if_var.h \
	    net/if_arp.h net/if_dl.h net/pfilt.h \
	    net/pfvar.h net/radix.h net/raw.h net/route.h netinet/in_var.h \
	    net/if_tun.h linux/if_tun.h netinet/ip_fw.h linux/ip_fw.h \
	    linux/ip_fwchains.h linux/netfilter_ipv4/ipchains_core.h)
	AC_CHECK_HEADERS(ip_fil_compat.h netinet/ip_fil_compat.h ip_compat.h \
	    netinet/ip_compat.h ip_fil.h netinet/ip_fil.h)
	AC_CHECK_HEADERS(hpsecurity.h stropts.h)
fi
AC_CHECK_HEADERS([arpa/inet.h fcntl.h inttypes.h netdb.h netinet/in.h nlist.h stdlib.h string.h sys/ioctl.h sys/param.h sys/socket.h sys/time.h unistd.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_C_INLINE
AC_DNET_SOCKADDR_IN6
AC_TYPE_OFF_T
AC_TYPE_PID_T
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT64_T
AC_TYPE_UINT8_T
if test "$ac_cv_header_sys_socket_h" = yes ; then
	AC_DNET_SOCKADDR_SA_LEN
fi
if test "$ac_cv_header_net_if_arp_h" = yes ; then
	AC_DNET_ARPREQ_ARP_DEV
fi
if test "$ac_cv_header_net_route_h" = yes ; then
	AC_DNET_ROUTE_RT_MSGHDR
fi
AC_PROG_GCC_TRADITIONAL
if test "$GCC" = yes ; then
   	CFLAGS="$CFLAGS -Wall"
fi

# Checks for library functions.
dnl AC_FUNC_MALLOC
AC_FUNC_MEMCMP
dnl AC_FUNC_REALLOC
AC_CHECK_FUNCS([gethostbyname gettimeofday memmove memset pow socket strchr strdup strerror strncasecmp strpbrk strstr strtol strtoul])
AC_REPLACE_FUNCS(err strlcat strlcpy strsep)

dnl Checks for other system-specific jonks.
if test "$CYGWIN" != yes ; then
	AC_DNET_BSD_BPF
	AC_DNET_LINUX_PROCFS
	AC_DNET_LINUX_PF_PACKET
	AC_DNET_STREAMS_MIB2
	AC_DNET_STREAMS_ROUTE
	AC_DNET_IOCTL_ARP
	AC_DNET_RAWIP_HOST_OFFLEN
	AC_DNET_RAWIP_COOKED
fi

dnl Check for arp interface.
if test "$ac_cv_header_Iphlpapi_h" = yes ; then
	AC_LIBOBJ([arp-win32])
elif test "$ac_cv_dnet_ioctl_arp" = yes ; then
	AC_LIBOBJ([arp-ioctl])
elif test "$ac_cv_dnet_route_h_has_rt_msghdr" = yes ; then
	AC_LIBOBJ([arp-bsd])
else
	AC_LIBOBJ([arp-none])
fi

dnl Check for Ethernet interface.
if test "$ac_cv_header_Iphlpapi_h" = yes ; then
	AC_LIBOBJ([eth-win32])
elif test "$ac_cv_header_net_pfilt_h" = yes ; then
	AC_LIBOBJ([eth-pfilt])
elif test "$ac_cv_dnet_bsd_bpf" = yes ; then
	AC_LIBOBJ([eth-bsd])
elif test "$ac_cv_dnet_linux_pf_packet" = yes ; then
	AC_LIBOBJ([eth-linux])
elif test "$ac_cv_header_net_raw_h" = yes ; then
	AC_LIBOBJ([eth-snoop])
elif test "$ac_cv_header_sys_ndd_var_h" = yes ; then
	AC_LIBOBJ([eth-ndd])
else
	AC_LIBOBJ([eth-none])
fi

dnl Check for firewall interface.
if test "$ac_cv_header_Iphlpapi_h" = yes ; then
        AC_LIBOBJ([fw-pktfilter])
elif test "$ac_cv_header_net_pfvar_h" = yes ; then
	AC_LIBOBJ([fw-pf])
elif test "$ac_cv_header_netinet_ip_fw_h" = yes ; then
     dnl XXX - ipfw2 support later...
     dnl case "$host_os" in
     dnl *freebsd5*)
     dnl	AC_LIBOBJ([fw-none]) ;;
     dnl *)
	AC_LIBOBJ([fw-ipfw]) ;;
     esac
elif test "$ac_cv_header_netinet_ip_fil_h" = yes ; then
	AC_LIBOBJ([fw-ipf])
elif test "$ac_cv_header_linux_ip_fw_h" = yes ; then
	AC_LIBOBJ([fw-ipchains])
elif test "$ac_cv_header_linux_ip_fwchains_h" = yes ; then
	AC_LIBOBJ([fw-ipchains])
elif test "$ac_cv_header_linux_netfilter_ipv4_ipchains_core_h" = yes ; then
	AC_LIBOBJ([fw-ipchains])
else
	AC_LIBOBJ([fw-none])
fi

dnl Check for network interface interface.
if test "$ac_cv_header_Iphlpapi_h" = yes ; then
	AC_LIBOBJ([intf-win32])
else
	AC_LIBOBJ([intf])
fi

dnl Check for raw IP interface.
if test "$ac_cv_header_Iphlpapi_h" = yes ; then
	AC_LIBOBJ([ip-win32])
elif test "$ac_cv_dnet_rawip_cooked" = yes ; then
	AC_LIBOBJ([ip-cooked])
else
	AC_LIBOBJ([ip])
fi

dnl Check for routing interface.
if test "$ac_cv_header_Iphlpapi_h" = yes ; then
	AC_LIBOBJ([route-win32])
elif test "$ac_cv_dnet_route_h_has_rt_msghdr" = yes ; then
	AC_LIBOBJ([route-bsd])
elif test "$ac_cv_dnet_linux_procfs" = yes ; then
	AC_LIBOBJ([route-linux])
elif test "$ac_cv_header_hpsecurity_h" = yes ; then
	AC_LIBOBJ([route-hpux])
else
	AC_LIBOBJ([route-none])
fi

dnl Check for tun interface.
if test "$ac_cv_header_linux_if_tun_h" = yes ; then
	AC_LIBOBJ([tun-linux])
elif test "$ac_cv_header_net_if_tun_h" = yes ; then
	if test "$ac_cv_header_stropts_h" = yes ; then
		AC_LIBOBJ([tun-solaris])
	else
		AC_LIBOBJ([tun-bsd])
	fi
elif test -c "/dev/tun0" ; then
	AC_LIBOBJ([tun-bsd])
else
	AC_LIBOBJ([tun-none])
fi

AC_CONFIG_FILES([Makefile
		 dnet-config
                 include/Makefile
                 include/dnet/Makefile
                 man/Makefile
                 python/Makefile
		 python/setup.py
                 src/Makefile
                 test/Makefile
                 test/check/Makefile
                 test/dnet/Makefile],
	[chmod 755 dnet-config])
AC_OUTPUT