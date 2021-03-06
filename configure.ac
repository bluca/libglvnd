dnl configure.ac
dnl Process this file with autoconf to produce a configure script.

AC_PREREQ([2.63])
AC_INIT([libglvnd], [0.0.0], [kbrenneman@nvidia.com])
AC_CONFIG_SRCDIR([config.h.in])
AC_CONFIG_HEADERS([config.h])

AC_CONFIG_AUX_DIR([bin])
AC_CONFIG_MACRO_DIR([m4])
AC_CANONICAL_SYSTEM

AC_USE_SYSTEM_EXTENSIONS

AM_INIT_AUTOMAKE([1.11 foreign silent-rules])
AM_SILENT_RULES([yes])

dnl Add an --enable-debug option
AX_CHECK_ENABLE_DEBUG(no, DEBUG)

dnl Checks for programs.
AC_PROG_AWK
AC_PROG_CXX

AC_PROG_CC
AM_PROG_AS
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AC_PROG_LIBTOOL
AC_CHECK_PROGS([PYTHON2], [python2 python])
AC_CHECK_PROGS([PERL], [perl])

dnl
dnl Auxiliary ELF symbol filtering support
dnl
AC_ARG_ENABLE([dt-auxiliary],
    [AS_HELP_STRING([--enable-dt-auxiliary],
    [enable auxiliary ELF symbol filtering @<:@default=disabled@:>@])],
    [enable_dt_auxiliary="$enableval"],
    [enable_dt_auxiliary=no]
)
AC_MSG_CHECKING([whether to enable auxiliary ELF symbol filtering])

if test "x$enable_dt_auxiliary" = xyes; then
    (
        LDFLAGS='-shared -Wl,--auxiliary=libc.so.6'
        AC_LINK_IFELSE([AC_LANG_SOURCE([])],
                        [dt_auxiliary_supported=yes],
                        [dt_auxiliary_supported=no])
        if test "x$dt_auxiliary_supported" = xyes; then
            AC_MSG_RESULT([yes, supported])
        else
            AC_MSG_RESULT([no, not supported])
        fi
        test "x$dt_auxiliary_supported" = xyes
    ) && dt_auxiliary_supported=yes
else
    dt_auxiliary_supported=no
    AC_MSG_RESULT([no])
fi

AM_CONDITIONAL([USE_DT_AUXILIARY], [test "x$dt_auxiliary_supported" = xyes])

dnl
dnl Arch/platform-specific settings. Copied from mesa
dnl
AC_ARG_ENABLE([asm],
    [AS_HELP_STRING([--disable-asm],
        [disable assembly usage @<:@default=enabled on supported plaforms@:>@])],
    [enable_asm="$enableval"],
    [enable_asm=yes]
)
asm_arch=""
AC_MSG_CHECKING([whether to enable assembly])
test "x$enable_asm" = xno && AC_MSG_RESULT([no])
# disable if cross compiling on x86/x86_64 since we must run gen_matypes
if test "x$enable_asm" = xyes && test "x$cross_compiling" = xyes; then
    case "$host_cpu" in
    i?86 | x86_64)
        enable_asm=no
        AC_MSG_RESULT([no, cross compiling])
        ;;
    esac
fi
# check for supported arches
if test "x$enable_asm" = xyes; then
    case "$host_cpu" in
    i?86)
        case "$host_os" in
        linux* | *freebsd* | dragonfly* | *netbsd*)
            test "x$enable_64bit" = xyes && asm_arch=x86_64 || asm_arch=x86
            ;;
        gnu*)
            asm_arch=x86
            ;;
        esac
        ;;
    x86_64)
        case "$host_os" in
        linux* | *freebsd* | dragonfly* | *netbsd*)
            test "x$enable_32bit" = xyes && asm_arch=x86 || asm_arch=x86_64
            ;;
        esac
        ;;
    armv7l)
        asm_arch=armv7l
        ;;
    sparc*)
        case "$host_os" in
        linux*)
            asm_arch=sparc
            ;;
        esac
        ;;
    esac

    case "$asm_arch" in
    x86)
        DEFINES="$DEFINES -DUSE_X86_ASM -DUSE_MMX_ASM -DUSE_3DNOW_ASM -DUSE_SSE_ASM"
        AC_MSG_RESULT([yes, x86])
        ;;
    x86_64)
        DEFINES="$DEFINES -DUSE_X86_64_ASM"
        AC_MSG_RESULT([yes, x86_64])
        ;;
    armv7l)
        DEFINES="$DEFINES -DUSE_ARMV7_ASM"
        AC_MSG_RESULT([yes, armv7l])
        ;;
    sparc)
        DEFINES="$DEFINES -DUSE_SPARC_ASM"
        AC_MSG_RESULT([yes, sparc])
        ;;
    *)
        AC_MSG_RESULT([no, platform not supported])
        ;;
    esac
fi

dnl Various conditionals.
AM_CONDITIONAL([GCC], [test x$GCC = xyes ])

dnl
dnl mapi top-relative paths: defined here so mapi can be used elsewhere
dnl

AC_SUBST([MAPI_PREFIX], [src/GLdispatch/vnd-glapi/mapi])
AC_SUBST([MAPI_MESA_PREFIX], [src/GLdispatch/mesa])

AM_CONDITIONAL(HAVE_X86_ASM, echo "$DEFINES" | grep 'X86_ASM' >/dev/null 2>&1)
AM_CONDITIONAL(HAVE_X86_64_ASM, echo "$DEFINES" | grep 'X86_64_ASM' >/dev/null 2>&1)
AM_CONDITIONAL(HAVE_SPARC_ASM, echo "$DEFINES" | grep 'SPARC_ASM' >/dev/null 2>&1)

dnl Checks for libraries.
AX_PTHREAD()

PKG_CHECK_MODULES([X11], [x11])
PKG_CHECK_MODULES([XEXT], [xext])
PKG_CHECK_MODULES([XORG], [xorg-server >= 1.11.0])

dnl Checks for header files.
AC_PATH_X
AC_CHECK_HEADERS([fcntl.h inttypes.h stddef.h stdlib.h string.h sys/param.h sys/time.h unistd.h])

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_INLINE
AC_C_TYPEOF
AC_TYPE_OFF_T
AC_TYPE_PID_T
AC_TYPE_SIZE_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT8_T
AC_CHECK_TYPES([ptrdiff_t])

dnl Checks for library functions.
AC_FUNC_MALLOC
AC_FUNC_REALLOC
AC_FUNC_STRNLEN
AC_CHECK_FUNCS([getpagesize gettimeofday memmove memset strdup strerror])

dnl TLS detection
AC_ARG_ENABLE([tls],
    [AS_HELP_STRING([--disable-tls],
        [disable TLS usage @<:@default=enabled on supported plaforms@:>@])],
    [enable_tls="$enableval"],
    [enable_tls=yes]
)

AC_MSG_CHECKING([for initial-exec TLS])
if test "x$enable_tls" = "xyes"; then
    AC_COMPILE_IFELSE([AC_LANG_SOURCE([
       __thread int foo __attribute__((tls_model("initial-exec")));
    ])],
    [HAVE_INIT_TLS=yes],[HAVE_INIT_TLS=no])
else
    HAVE_INIT_TLS=no
fi
AC_MSG_RESULT($HAVE_INIT_TLS)

# Figure out what implementation to use for the entrypoint stubs.
# This will set an automake condition, which is then used in
# src/GLdispatch/vnd-glapi/entry_files.mk.

AC_MSG_CHECKING([for entrypoint stub type])
case "x$asm_arch" in
xx86)
    # The x86 TLS stubs don't work, so use the TSD stubs instead.
    gldispatch_entry_type=x86_tsd
    gldispatch_use_tls=no
    ;;
xx86_64)
    # For x86-64, both the TLS and TSD stubs work.
    if test "x$HAVE_INIT_TLS" = "xyes" ; then
        gldispatch_entry_type=x86_64_tls
        gldispatch_use_tls=yes
    else
        gldispatch_entry_type=x86_64_tsd
        gldispatch_use_tls=no
    fi
    ;;
xarmv7l)
    # For ARMv7, only the TSD stubs have been implemented yet.
    gldispatch_entry_type=armv7_tsd
    gldispatch_use_tls=no
    ;;
*)
    # The C stubs will work with either TLS or TSD.
    gldispatch_entry_type=pure_c
    gldispatch_use_tls=$HAVE_INIT_TLS
    ;;
esac
AC_MSG_RESULT([$gldispatch_entry_type, TLS=$gldispatch_use_tls])

AS_IF([test "x$gldispatch_use_tls" = "xyes"],
      [AC_DEFINE([GLDISPATCH_USE_TLS], 1,
      [Define to 1 if libGLdispatch should use a TLS variable for the dispatch table.])])
AM_CONDITIONAL([GLDISPATCH_USE_TLS], [test "x$gldispatch_use_tls" = "xyes"])
AM_CONDITIONAL([GLDISPATCH_TYPE_X86_TLS], [test "x$gldispatch_entry_type" = "xx86_tls"])
AM_CONDITIONAL([GLDISPATCH_TYPE_X86_TSD], [test "x$gldispatch_entry_type" = "xx86_tsd"])
AM_CONDITIONAL([GLDISPATCH_TYPE_X86_64_TLS], [test "x$gldispatch_entry_type" = "xx86_64_tls"])
AM_CONDITIONAL([GLDISPATCH_TYPE_X86_64_TSD], [test "x$gldispatch_entry_type" = "xx86_64_tsd"])
AM_CONDITIONAL([GLDISPATCH_TYPE_ARMV7_TSD], [test "x$gldispatch_entry_type" = "xarmv7_tsd"])
AM_CONDITIONAL([GLDISPATCH_TYPE_PURE_C], [test "x$gldispatch_entry_type" = "xpure_c"])


AC_MSG_CHECKING([for constructor attributes])
AC_COMPILE_IFELSE([AC_LANG_SOURCE([
void __attribute__ ((constructor)) foo(void)
{
}
void __attribute__ ((destructor)) bar(void)
{
}
])],
[USE_ATTRIBUTE_CONSTRUCTOR=yes],[USE_ATTRIBUTE_CONSTRUCTOR=no])
AC_MSG_RESULT($USE_ATTRIBUTE_CONSTRUCTOR)
AS_IF([test "x$USE_ATTRIBUTE_CONSTRUCTOR" = "xyes"],
      [AC_DEFINE([USE_ATTRIBUTE_CONSTRUCTOR], 1,
      [Define to 1 if the compiler supports constructor attributes.])])

AC_MSG_CHECKING([for pthreads rwlocks])
AC_COMPILE_IFELSE([AC_LANG_SOURCE([
#include <pthread.h>
void foo(void)
{
    pthread_rwlock_t lock;
    pthread_rwlock_init(&lock, NULL);
}
])],
[HAVE_PTHREAD_RWLOCK_T=yes],[HAVE_PTHREAD_RWLOCK_T=no])
AC_MSG_RESULT($HAVE_PTHREAD_RWLOCK_T)
AS_IF([test "x$HAVE_PTHREAD_RWLOCK_T" = "xyes"],
      [AC_DEFINE([HAVE_PTHREAD_RWLOCK_T], 1,
      [Define to 1 if the compiler supports pthreads rwlocks.])])

AC_MSG_CHECKING([for sync intrinsics])
AC_COMPILE_IFELSE([AC_LANG_SOURCE([
int foo(int volatile *val, int oldVal, int newVal)
{
    return __sync_add_and_fetch(val, 1);
    return __sync_lock_test_and_set(val, newVal);
    return __sync_val_compare_and_swap(val, oldVal, newVal);
}
])],
[HAVE_SYNC_INTRINSICS=yes],[HAVE_SYNC_INTRINSICS=no])
AC_MSG_RESULT($HAVE_SYNC_INTRINSICS)
AS_IF([test "x$HAVE_SYNC_INTRINSICS" = "xyes"],
      [AC_DEFINE([HAVE_SYNC_INTRINSICS], 1,
      [Define to 1 if the compiler supports __sync intrinsic functions.])])

# See if the linker supports the --no-undefiend flag.
AX_CHECK_LINK_FLAG([-Xlinker --no-undefined],
      [AC_SUBST([LINKER_FLAG_NO_UNDEFINED], ["-Xlinker --no-undefined"])],
      [AC_SUBST([LINKER_FLAG_NO_UNDEFINED], [""])])

dnl default CFLAGS
CFLAGS="$CFLAGS -Wall -Werror -std=gnu99 -include config.h -fvisibility=hidden $DEFINES"

AC_CONFIG_FILES([Makefile
                 libglvnd.pc
                 include/Makefile
                 src/Makefile
                 src/GL/Makefile
                 src/OpenGL/Makefile
                 src/GLESv1/Makefile
                 src/GLESv2/Makefile
                 src/GLX/Makefile
                 src/x11glvnd/Makefile
                 src/GLdispatch/Makefile
                 src/GLdispatch/vnd-glapi/Makefile
                 src/util/Makefile
                 src/util/glvnd_pthread/Makefile
                 src/util/trace/Makefile
                 tests/Makefile
                 tests/GLX_dummy/Makefile])
AC_OUTPUT
