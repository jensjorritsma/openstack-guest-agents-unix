AC_PREREQ([2.59])
AC_INIT([nova-agent], [1.39.0], [chris.behrens@rackspace.com])
AC_CONFIG_SRCDIR([src/nova-agent.c])
AC_CONFIG_HEADERS([include/config.h])

AM_INIT_AUTOMAKE([foreign -Wall -Werror tar-ustar])

AC_PREFIX_DEFAULT([/usr])

AC_ARG_VAR(INSTALL_D, "INSTALL_D allows to feed -D at install")

AC_ARG_ENABLE([coverage],
  [AS_HELP_STRING([--enable-coverage],
    [Toggle coverage @<:@default=off@:>@])],
  [ac_coverage="$enableval"],
  [ac_coverage="no"])

AS_IF([test "$ac_coverage" = "yes"],
  [
    GCOV_CFLAGS="-fprofile-arcs -ftest-coverage -lgcov -O0"
  ])


AC_PROG_CC
AM_PROG_CC_C_O
# Automake 1.12 requires AM_PROG_AR, but older versions don't know it.
m4_ifdef([AM_PROG_AR], [AM_PROG_AR])
AC_PROG_LIBTOOL
AC_CONFIG_MACRO_DIR([m4])

missing() {
    what="$1"
    echo ""
    echo "The $what is missing and is required for building"
    echo ""
}

if test "$PYTHON_CFLAGS" = "" -a "$PYTHON_LIB" = ""; then
    AC_MSG_CHECKING([for python build requisites])

    for python in "python2.7" "python2.6" "python2.5" "python2.4" ; do
        for dir in $prefix "/usr/local" "/usr"; do
            if test -f ${dir}/include/${python}/Python.h ; then

                lib_dir_paths=( "lib" "lib64" )
                gcc_multiarch=`gcc  --print-multiarch` ; is_gcc_multiarch=$?
                if [[ $is_gcc_multiarch -eq 0 ]]; then
                  lib_dir_paths[[2]]="lib/${gcc_multiarch}"
                fi
                for libdir in "${lib_dir_paths[[@]]}" ; do
                    so_path=${dir}/${libdir}
                    if test -f ${so_path}/lib${python}.so ; then
                        # We're done.  Found header and library
                        PYTHON_VER=${python}
                        PYTHON_CFLAGS="-I${dir}/include/${python}"
                        PYTHON_LIB="-l${python}"
                        PYTHON_LDFLAGS="-L${so_path}"
                        break
                    fi
                done
            fi
        done
    done

    if test "x${PYTHON_CFLAGS}" = "x" ; then
        AC_MSG_RESULT([not found])
    else
        AC_MSG_RESULT([adding ${PYTHON_CFLAGS} and ${PYTHON_LIB}])
    fi
fi

CFLAGS="$CFLAGS -fno-strict-aliasing $PYTHON_CFLAGS"
# Make sure to add this, so we don't need to manually add it on FreeBSD
LDFLAGS="$LDFLAGS $PYTHON_LDFLAGS -L/usr/local/lib"

AC_MSG_CHECKING([that the python library can be linked])

SAVE_CFLAGS=$CFLAGS
SAVE_LIBS=$LIBS

CFLAGS="$CFLAGS $PYTHON_CFLAGS"
LIBS="$LIBS $PYTHON_LIB"

AC_TRY_LINK([#include <Python.h>], [
int main(int argc, char **argv)
{
    Py_Main(argc, argv);
}
], [AC_MSG_RESULT([yes])], [

    AC_MSG_RESULT([no])
    AC_MSG_RESULT([])
    AC_MSG_RESULT([A working python header and/or library could not be found])
    AC_MSG_RESULT([Re-run configure specifying PYTHON_CFLAGS and PYTHON_LIB])
    AC_MSG_RESULT([and/or CFLAGS and LDFLAGS correctly])

    exit 1

    ])

LIBS=$SAVE_LIBS
CFLAGS=$SAVE_CFLAGS

AC_CHECK_PROG([PATCHELF], [patchelf], [yes])
if test "$PATCHELF" != "yes"; then
    missing "patchelf"
    exit 1
fi

# Check for certain headers
AC_CHECK_HEADERS(crypt.h)

AC_CHECK_LIB([crypt], [crypt])

AC_CHECK_LIB([pthread], [pthread_create], [], [
    missing "the pthread library"
    exit 1
])

DESTDIR="/"

AC_SUBST([GCOV_CFLAGS])
AC_SUBST([PYTHON_VER])
AC_SUBST([PYTHON_CFLAGS])
AC_SUBST([PYTHON_LIB])
AC_SUBST([DESTDIR])

AC_CONFIG_FILES([Makefile
                 src/Makefile
                 lib/Makefile
                 plugins/Makefile
                 commands/Makefile
                 tests/Makefile
                 scripts/installer.sh
                 scripts/generic/nova-agent
                 scripts/gentoo/nova-agent
                 scripts/freebsd/nova-agent
                 scripts/systemd/nova-agent.service
                 ])

AC_OUTPUT

