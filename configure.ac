#                                  -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.
# Order is largely irrevellant, although it must start with AC_INIT and end with AC_OUTPUT
# See http://autotoolset.sourceforge.net/tutorial.html
# and http://www.openismus.com/documents/linux/automake/automake.shtml

AC_PREREQ(2.57)

AC_INIT(be_plugin, 0.0.0-beta-0, bugs@digitalcorpora.org)
AC_CONFIG_MACRO_DIR(m4)

# NOTE 1 - src/test/Makefile is referenced so the test/Makefile is created,
#          but it is not listed as a SUBDIR for Makefile.am so that it is
#          not automatically built.  Compile manually there for testing.
#
# NOTE 2 - src_win/Makefile is not automatically generated. This is necessary
#          so that it can be run even after a 'make distclean' is done
#

AC_CONFIG_FILES([Makefile src/Makefile])

# define CC and CXX as desired for alternate compilers
#CC=clang
#CXX=clang++

AC_CONFIG_HEADER([config.h])
AM_INIT_AUTOMAKE([subdir-objects])
AC_PREFIX_PROGRAM(be_plugin)
AC_PROG_CC
AM_PROG_CC_C_O                  dnl allow per-produce flags
AC_PROG_CXX
AC_PROG_INSTALL
AC_PROG_LIBTOOL

# use C++11 mode if available; HAVE_CXX11 is defined in config.h if so.  Don't
# use the GNU C++11 extensions for portability's sake (noext).
# https://www.gnu.org/software/autoconf-archive/ax_cxx_compile_stdcxx_11.html
m4_include([m4/ax_cxx_compile_stdcxx_11.m4])
AX_CXX_COMPILE_STDCXX_11(noext, optional)

# Endian check is required for MD5 implementation
AC_C_BIGENDIAN

##
## NSIS Windows Installer
AC_CHECK_PROG(has_nsis, makensis, true, false)
if test x"$has_nsis" == "xfalse" ; then
  AC_MSG_NOTICE([nsis is not available for creating the Windows installer.  Try 'apt-get install mingw32-nsis' or 'yum install mingw32-nsis' or 'port install mingw32-nsis' or whatever package manager you happen to be using....])
fi

################################################################
## Includes
m4_include([m4/slg_mingw_support.m4])
m4_include([m4/slg_searchdirs.m4])
m4_include([m4/slg_gcc_all_warnings.m4])
m4_include([src/dfxml/src/dfxml_configure.m4])
m4_include([src/be13_api/be13_configure.m4])

################################################################
## mcheck support
## enable memory analysis support for detecting memory leaks
## and detecting multiply deallocated memory resources
## see http://en.wikipedia.org/wiki/Mtrace
## here is the basic workflow:
##   1. example usage and recompile: "./configure --enable-mcheck"
##   2. export the environment variable for the output file,
##        e.g. "export MALLOC_TRACE=~/malloc_outfile.txt"
##   3. improve readability of malloc_outfile.txt using mtrace,
##        e.g. "mtrace /usr/local/bin/bulk_extractor ~/malloc_outfile.txt
AC_ARG_ENABLE([mcheck], AS_HELP_STRING([--enable-mcheck], \
              [enable memory analysis support for detecting memory leaks \
               and detecting multiply deallocated memory resources]), \
              AC_DEFINE(HAVE_MCHECK, 1, \
              [define the output file using global variable MALLOC_TRACE, \
               for example "MALLOC_TRACE=./mcheck.out; export MALLOC_TRACE". \
               Then post-process unreadable file mcheck.out using the mtrace command line tool, \
               for example "mtrace /usr/local/bin/bulk_extractor mcheck.out".  \
               Unfortunately, only addresses with written code are looked up, \
               in otherwords, use of malloc is looked up, but use of new is not.]))

################################################################
## AddressSanitizer support
AC_ARG_ENABLE([address-sanitizer], AS_HELP_STRING([--enable-address-sanitizer], \
              [enabled AddressSanitizer support for detecting a wide variety of \
               memory allocation and deallocation errors]), \
              [AC_DEFINE(HAVE_ADDRESS_SANITIZER, 1, [enable AddressSanitizer])
              CFLAGS="$CFLAGS -fsanitize=address"
              CXXFLAGS="$CXXFLAGS -fsanitize=address"
              AC_TRY_COMPILE([],[const int i=0;],[AC_MSG_NOTICE([Address Sanitizer Enabled])],
                                                 [AC_MSG_ERROR([Address Sanitizer not available])])
              ])

################################################################
# PTHREAD support
# With special nods to compiling under mingw

if test  x"$mingw" = x"yes";  then
  CFLAGS="$CFLAGS -mthreads -pthread "
  CPPFLAGS="-DPTW32_STATIC_LIB $CPPFLAGS"
  CXXFLAGS="$CXXFLAGS -mthreads -pthread "
  LDFLAGS="$LDFLAGS -mthreads -pthread "
  AC_DEFINE(HAVE_PTHREAD,1,[Defined to POSIX threads for mingw])
else
  m4_include([m4/ax_pthread.m4])
  AC_MSG_CHECKING([Running pthread test from http://www.gnu.org/software/autoconf-archive/ax_pthread.html..])
  AX_PTHREAD([
              echo Using settings from AX_PTHREAD
              LIBS="$PTHREAD_LIBS $LIBS"
              CFLAGS="  $PTHREAD_CFLAGS $CFLAGS"
              CXXFLAGS="$PTHREAD_CFLAGS $CXXFLAGS "
              CC="$PTHREAD_CC"],
            [AC_MSG_ERROR(bulk_extractor requires pthreads)])
fi

AC_MSG_NOTICE([After pthread test, CFLAGS: $CFLAGS])
AC_MSG_NOTICE([After pthread test, CXXFLAGS: $CXXFLAGS])
AC_MSG_NOTICE([After pthread test, LDFLAGS: $LDFLAGS])

AC_CHECK_HEADERS([pthread.h])
AC_CHECK_LIB([pthreadGC2],[pthread_create])

################################################################
## more to satisfy dependencies in be13_api

## Check for headers used by bulk Extractor
## do not put pthread here
AC_CHECK_HEADERS([alloca.h dirent.h dlfcn.h err.h errno.h fcntl.h inttypes.h libgen.h limits.h malloc.h mmap.h pwd.h signal.h stdarg.h stdint.h stdio.h strings.h string.h stdlib.h sys/cdefs.h sys/disk.h sys/fcntl.h sys/ioctl.h sys/mman.h sys/mmap.h sys/mount.h sys/param.h sys/socket.h sys/stat.h sys/types.h sys/time.h sys/resource.h sys/sysctl.h time.h unistd.h windows.h CoreServices/CoreServices.h])
AC_CHECK_FUNCS([err errx getuid getpwuid gethostname getrusage gmtime_r isxdigit ishexnumber le64toh localtime_r _lseeki64 inet_ntop pread64 pread printf mmap munmap MD5 mkstemp mktemp random srandom srandomdev sleep SleepEx strptime usleep vasprintf warn warnx])
AC_CHECK_FUNCS([CreateProcess LoadLibrary IncrementAtomic InterlockedIncrement])

## dlopen is now itself in a different library 
## Explicitly check for dlopen library before checking for dlopen
##       -ldl (see http://stackoverflow.com/questions/956640/linux-c-error-undefined-reference-to-dlopen)
AC_CHECK_LIB([dl],[dlopen])		              
AC_CHECK_FUNCS([dlopen dlopen_preflight])

# Test for sin_len
AC_CHECK_HEADERS([arpa/inet.h netinet/in.h wsipx.h])
AC_CHECK_HEADERS([netinet/ip.h], [], [],
[[
#include <sys/types.h>
#ifdef HAVE_NETINET_IN_H
#include <netinet/in.h>
#endif
]])

AC_CHECK_MEMBER([struct sockaddr_in.sin_len],
		[AC_DEFINE(HAVE_SOCKADDR_IN_SIN_LEN, 1, [Do we have sockaddr.sin_len?])],
		[],
		[
#include <sys/types.h>
#include <arpa/inet.h>
#ifdef HAVE_NETINET_IN_H
#include <netinet/in.h>
#endif
		])




################################################################
##
## Multithreading stuff

# On mingw, be sure to use the static version and be sure we are using mthread option
# (which should be a no-op on later version of G++ anyway)

AC_CHECK_FUNCS([pthread_win32_process_attach_np pthread_win32_process_detach_np pthread_win32_thread_attach_np pthread_win32_thread_detach_np ])

# end PTHREAD SUPPORT
################################################################

# Specific functions, may not be required
AC_TYPE_INT64_T
AC_SYS_LARGEFILE
AC_SYS_POSIX_TERMIOS


#################################################################
############### drop optimization flags and add -g if requested ################
#if test x"${AFF_NOOPT}" != "x" ; then
#   with_noopt="yes";
#fi
#
#if test "${with_opt}" = "no" ; then
#  CFLAGS=`echo   -g "$CFLAGS"   | sed s/-O[[0-9]]//`             # note the double quoting! 
#  CXXFLAGS=`echo -g "$CXXFLAGS" | sed s/-O[[0-9]]//`
#else
#  # If we are not stripping the optimizer, add fortify source
#  CFLAGS="  $CFLAGS   -D_FORTIFY_SOURCE=2 "
#  CXXFLAGS="$CXXFLAGS -D_FORTIFY_SOURCE=2 "
#
#  # and increase optimizer from -O2 to -O3 if not explicitly forbidden
#  if test "${with_o3}" != "no" ; then
#      CFLAGS=`echo   -g "$CFLAGS"   | sed s/-O2/-O3/`             # note the double quoting! 
#      CXXFLAGS=`echo -g "$CXXFLAGS" | sed s/-O2/-O3/`
#  fi
#fi

################################################################
## support required by libxml2

# zzzzzzzzzzzzz
# note: a recommended approach is to use PKG_CHECK_MODULES:
#          PKG_CHECK_MODULES(XML, libxml-2.0)
# but this requires compiling with per-target flags,
# requiring 'AM_PROG_CC_C_O' and the addition of
# XML_CFLAGS to hashdb_CPPFLAGS and XML_LIBS to hashdb_LDFLAGS,
# which may be making mingw not work by disrupting names
# and blocking the ability to find libraries.
#
# So instead, check for headers and libraries in the hardcoded path
# expected for a standard Fedora installation
# or hardwaire LDFLAGS for your path:
# zzzzzzzzzzzzz

# the win32 configuration of libxml2 requires zlib and iconv libraries
if test  x"$mingw" = x"yes";  then
  AC_CHECK_LIB([z], [gzdopen],[LIBS="-lz $LIBS"], [AC_MSG_ERROR([Could not find zlib library])])
  AC_CHECK_LIB([iconv], [iconv],[LIBS="-liconv $LIBS"], [AC_MSG_ERROR([Could not find win-iconv library])])
fi

# find the libxml/parser.h header
PKG_CHECK_MODULES(XML, libxml-2.0 >= 2.4, [], [AC_MSG_ERROR([libxml2 failed])])

# find the xml2 library
AC_CHECK_LIB([xml2], [xmlSAXUserParseFile],[LIBS="-lxml2 $LIBS"],
    [AC_MSG_ERROR([Could not find libxml2 library required to build the hashid scanner])])

################################################################
## support required by boost
AX_BOOST_BASE([1.53],
  [AX_BOOST_SYSTEM
  AX_BOOST_FILESYSTEM
  AX_BOOST_IOSTREAMS
  AX_BOOST_THREAD
  LIBS="$BOOST_SYSTEM_LIB $BOOST_FILESYSTEM_LIB $BOOST_IOSTREAMS_LIB $BOOST_THREAD_LIB $LIBS"],
  [AC_MSG_ERROR([Boost 1.53 or newer is required])])


################################################################
## libboost_thread needs clock_gettime from librt
AC_CHECK_LIB([rt], [clock_gettime])

################################################################
## OpenSSL Support is required for hash_t and bloom
AC_CHECK_HEADERS([openssl/aes.h openssl/bio.h openssl/evp.h openssl/hmac.h openssl/md5.h openssl/pem.h openssl/rand.h openssl/rsa.h openssl/sha.h openssl/pem.h openssl/x509.h])
# OpenSSL has been installed under at least two different names...
AC_CHECK_LIB([crypto],[EVP_get_digestbyname])	
AC_CHECK_LIB([ssl],[SSL_library_init])
AC_CHECK_FUNCS([EVP_get_digestbyname],,
	AC_MSG_ERROR([SSL/OpenSSL support required]))

################################################################
## support required by bloom.h
#AC_CHECK_FUNCS([printf getrusage err errx warn warnx mmap],,[AC_MSG_WARN([Not all requested functions are available: printf, getrusage...])])
#AC_CHECK_HEADERS([err.h sys/mman.h sys/resource.h unistd.h],,[AC_MSG_WARN([Not all requested headers are available: err.h, sys/mman.h, ...])])
#PTHREAD: AC_CHECK_HEADERS([pthread.h],,[AC_MSG_ERROR([pthread headers are required])])
# note that crypto requires zlib
AC_CHECK_LIB([crypto],[EVP_Digest],[],[AC_MSG_ERROR([Could not find crypto library])])
LIBS="-lcrypto $LIBS"

#PTHREAD: AC_CHECK_LIB([pthread],[pthread_mutexattr_settype],[],[AC_MSG_ERROR([Could not find pthread library])])
#LIBS="-lssl -lcrypto -lpthread $LIBS"
#LIBS="-lcrypto -lpthread $LIBS"
## end support required by bloom.h

# Should we disable optimization?
AC_ARG_WITH([opt], AC_HELP_STRING([--without-opt], [Drop all -O C flags]))
# Or maybe just tone it down a bit?
AC_ARG_WITH([o3], AC_HELP_STRING([--without-o3], [Do not force O3 optimization; use default level]))

################################################################
## regex support
## there are several options
## tre is better than regex
AC_CHECK_LIB([stdc++],[main])
AC_CHECK_LIB([sicudt],[main])
AC_CHECK_HEADER([unicode/ucnv.h])
AC_SEARCH_LIBS([uset_close], [sicuuc])
AC_CHECK_LIB([sicuuc],[main])
#AC_CHECK_HEADER([lightgrep/api.h])
#AC_CHECK_LIB([lightgrep], [lg_create_pattern])
AC_CHECK_HEADERS([regex.h tre/tre.h])
AC_CHECK_LIB([regex],[regcomp])
AC_CHECK_LIB([tre],[tre_regcomp])
AC_CHECK_FUNCS([regcomp tre_regcomp tre_version])

################################################################
## hashdb required

if test x"$mingw" = x"yes";  then
  # the win32 configuration of libxml2 requires zlib and iconv libraries
  AC_CHECK_LIB([z], [gzdopen],[LIBS="-lz $LIBS"], [AC_MSG_ERROR([Could not find zlib library])])
  AC_CHECK_LIB([iconv], [iconv],[LIBS="-liconv $LIBS"], [AC_MSG_ERROR([Could not find win-iconv library])])
fi

AC_CHECK_LIB([xml2], [xmlSAXUserParseFile],,[missing -llibxml2])


AC_LANG_PUSH(C++)
AC_CHECK_HEADER([hashdb.hpp], [], [AC_MSG_ERROR([Could not find hashdb header file])])
AC_CHECK_LIB([hashdb], [hashdb_version], [], [AC_MSG_ERROR([Could not find hashdb library])])
AC_LANG_POP()

################################################################
## support required by be13_api required by DFXML
AC_CHECK_HEADERS([sys/cdefs.h sys/resource.h pwd.h sys/utsname.h])
AC_CHECK_FUNCS([localtime_r getuid gethostname getwpuid getrusage mkstemp vasprintf])

################################################################
## libtool required for preparing the hashdb library
AC_CHECK_PROG(has_libtool, libtool, true, false)
if test x"$has_libtool" == "xfalse" ; then
  AC_MSG_ERROR([libhashdb requires libtool, please install libtool such as The GNU Portable Library Tool])
fi

################################################################
## diagnostic defines
AC_TRY_COMPILE([#pragma GCC diagnostic ignored "-Wshadow"],[int a=3;],
  [AC_DEFINE(HAVE_DIAGNOSTIC_SHADOW,1,[define 1 if GCC supports -Wshadow])]
)
AC_TRY_COMPILE([#pragma GCC diagnostic ignored "-Wundef"],[int a=3;],
  [AC_DEFINE(HAVE_DIAGNOSTIC_UNDEF,1,[define 1 if GCC supports -Wundef])]
)
AC_TRY_COMPILE([#pragma GCC diagnostic ignored "-Wcast-qual"],[int a=3;],
  [AC_DEFINE(HAVE_DIAGNOSTIC_CAST_QUAL,1,[define 1 if GCC supports -Wcast-qual])]
)
AC_TRY_COMPILE([#pragma GCC diagnostic ignored "-Weffcpp"],[int a=3;],
  [AC_DEFINE(HAVE_DIAGNOSTIC_EFFCPP,1,[define 1 if GCC supports -Weffc++])]
)
AC_TRY_COMPILE([#pragma GCC diagnostic ignored "-Wsuggest-attribute=noreturn"],[int a=3;],
  [AC_DEFINE(HAVE_DIAGNOSTIC_SUGGEST_ATTRIBUTE,1,[define 1 if GCC supports -Wsuggest-attribute=noreturn])]
)
#AC_TRY_COMPILE([#pragma GCC diagnostic ignored "-Wredundant-decls"],[int a=3;],
#  [AC_DEFINE(HAVE_DIAGNOSTIC_REDUNDANT_DECLS,1,[define 1 if GCC supports -Wredundant-decls])]
#)

################################################################
# do we have a broken strchr?
# This prevents errors in base64_forensics.cpp
AC_LANG_PUSH(C++)
AC_TRY_COMPILE([#include <string.h>],
               [static const char Base64[] = "ABCDEFGH";int ch=3;char *pos = strchr(Base64, ch);],
               conforming_strchr=yes,
               conforming_strchr=no)
if test "$conforming_strchr" = yes; then
  AC_MSG_NOTICE([strchr is conforming])
  AC_DEFINE(HAVE_CONFORMING_STRCHR, 1,[define to 1 if strchr takes const char*])
else
  AC_MSG_NOTICE([strchr is not conforming])
fi
AC_LANG_POP()

#################################################################
##
## Do we have the CPUID instruction?
#AC_TRY_COMPILE([#define cpuid(id) __asm__( "cpuid" : "=a"(eax), "=b"(ebx), "=c"(ecx), "=d"(edx) : "a"(id), "b"(0), "c"(0), "d"(0))],
#			[unsigned long eax, ebx, ecx, edx;cpuid(0);],
#			have_cpuid=yes,
#			have_cpuid=no)
#if test "$have_cpuid" = yes; then
# AC_DEFINE(HAVE_ASM_CPUID, 1, [define to 1 if __asm__ CPUID is available])
#fi

################################################################
############## drop optimization flags and add -g if requested ################
if test x"${AFF_NOOPT}" != "x" ; then
   with_noopt="yes";
fi

if test "${with_opt}" = "no" ; then
  CFLAGS=`echo   -g "$CFLAGS"   | sed s/-O[[0-9]]//`             # note the double quoting! 
  CXXFLAGS=`echo -g "$CXXFLAGS" | sed s/-O[[0-9]]//`
else
  # If we are not stripping the optimizer, add fortify source
  CFLAGS="  $CFLAGS   -D_FORTIFY_SOURCE=2 "
  CXXFLAGS="$CXXFLAGS -D_FORTIFY_SOURCE=2 "

  # and increase optimizer from -O2 to -O3 if not explicitly forbidden
  if test "${with_o3}" != "no" ; then
      CFLAGS=`echo   -g "$CFLAGS"   | sed s/-O2/-O3/`             # note the double quoting! 
      CXXFLAGS=`echo -g "$CXXFLAGS" | sed s/-O2/-O3/`
  fi
fi

################################################################
############## A hash type is required, default MD5
AC_ARG_VAR(HASH_TYPE, [Set hash type to one of md5, sha1, sha256, sha512, straight64, default md5])
if test x"${HASH_TYPE}" = "x" -o x"${HASH_TYPE}" = "xmd5"; then
  AC_DEFINE(USE_HASH_TYPE_MD5, 1, [define to compile for md5 type])
elif test x"${HASH_TYPE}" = "xsha1"; then
  AC_DEFINE(USE_HASH_TYPE_SHA1, 1, [define to compile for sha1 type])
elif test x"${HASH_TYPE}" = "xsha256"; then
  AC_DEFINE(USE_HASH_TYPE_SHA256, 1, [define to compile for sha256 type])
elif test x"${HASH_TYPE}" = "xsha512"; then
  AC_DEFINE(USE_HASH_TYPE_SHA512, 1, [define to compile for sha512 type])
elif test x"${HASH_TYPE}" = "xstraight64"; then
  AC_DEFINE(USE_HASH_TYPE_STRAIGHT64, 1, [define to compile for straight64 type])
else
  AC_MSG_ERROR([The HASH_TYPE value provided is invalid])
fi

AC_MSG_NOTICE([*************************************])
AC_MSG_NOTICE([*************************************])
AC_MSG_NOTICE([  PACKAGE_NAME:     $PACKAGE_NAME])
AC_MSG_NOTICE([  PACKAGE_VERSION:  $PACKAGE_VERSION])
AC_MSG_NOTICE([  CC:               $CC])
AC_MSG_NOTICE([  CXX:              $CXX])
AC_MSG_NOTICE([  CPPFLAGS:         $CPPFLAGS])
AC_MSG_NOTICE([  CFLAGS:           $CFLAGS])
AC_MSG_NOTICE([  CXXFLAGS:         $CXXFLAGS])
AC_MSG_NOTICE([  LIBS:             $LIBS])
AC_MSG_NOTICE([  LDFLAGS:          $LDFLAGS])

AC_OUTPUT

## Finally, record the values of CFLAGS, CPPFLAGS, and CXXFLAGS for DFXML
echo "#define CPPFLAGS \"$CPPFLAGS\"" >> config.h
echo "#define CFLAGS   \"$CFLAGS\""   >> config.h
echo "#define CXXFLAGS \"$CXXFLAGS\"" >> config.h
echo "#define LIBS     \"$LIBS\""     >> config.h
echo "#define LDFLAGS  \"$LDFLAGS\""  >> config.h

## for git version, see dfxml_configure.m4
if test x"$GIT_COMMIT" != "x" ; then
  echo "#define GIT_COMMIT  \"$GIT_COMMIT\""  >> config.h
fi

