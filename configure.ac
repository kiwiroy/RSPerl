#
#	configure.ac for RSPerl
#
# - 2015/07
#	+ updates to work with current linux environments
#	+ To refresh and test:
#		aclocal ; autoconf ; ./configure --with-in-perl
#		aclocal ; autoconf ; ./configure

dnl Configuration script for RSPerl

AC_INIT(DESCRIPTION)

AC_PATH_PROG(PERL, perl, "/usr/bin/perl")
AC_SUBST(PERL)
AC_ARG_WITH(modules, list of Perl modules with C-level code auto load,,)

AC_ARG_WITH(splus,[--with-splus		Compile as an SPlus library (rather than R). Value can be the (fully qualified) name of the Splus script.], USE_SPLUS=1)
AC_ARG_WITH(in-perl, [--with-in-perl	Compile support for running R within Perl], [with_in_perl=$withval; FORCE_WITH_IN_PERL="yes"] , with_in_perl=yes)  # XXX was no. Changed for R CMD check experiments.

if test "${with_in_perl}" != "no" ; then
  R_IN_PERL_LIBRARY="RinPerl"
  PERL_MODULES=${PERL_MODULES-"no"}
  if test "${PERL_MODULES}" = "no" ; then
      echo "No support for any of the Perl modules from calling Perl from R."
  fi
fi

AC_SUBST(R_IN_PERL_LIBRARY)
R_HOME=`echo 'cat(R.home(component = "home"))' | Rscript -`
AC_SUBST(R_HOME)
AC_SUBST(R_SHLIB_CPPFLAGS)
AC_SUBST(R_SHLIB_LDFLAGS)

dnl This is output to RinPerlMakefile and used to create the Makefile.perl
dnl from Makefile.PL.  (This is all very indirect, but it is reasonably coherent.)

dnl Currently, for backward compatability, to install into the standard Perl location
dnl use 
dnl     --with-perl-prefix=
dnl i.e. with no value.  The default value is $R_PACKAGE_DIR (not $R_PACKAGE_DIR)

R_PACKAGE_DIR=$R_HOME/library/RSPerl
AC_ARG_WITH(perl-prefix, [Name of directory passed to perl Makefile.PL PREFIX=<dir>], [SET_PERL_PREFIX="user"; PERL_PREFIX=$withval], [ PERL_PREFIX=${R_PACKAGE_DIR}])
#AC_ARG_WITH(perl-prefix, [Name of directory passed to perl Makefile.PL PREFIX=<dir>], [SET_PERL_PREFIX="user"; PERL_PREFIX=$withval], [ PERL_PREFIX=`perl -e 'print "$INC[0]"'`])
AC_ARG_WITH(perl-lib, [Name of directory passed to perl Makefile.PL LIB=<dir>], [SET_PERL_LIB="user"; PERL_LIB=$withval], [ PERL_LIB=${R_PACKAGE_DIR}/perl])

#if test "${SET_PERL_LIB}" = "user" && test "${SET_PERL_PREFIX}"

dnl did the user specify the prefix as being empty and not specify anything about the lib? If so, unset PERL_LIB
dnl i.e.   --with-perl-prefix=  
if test -z "${PERL_PREFIX}" && test "${SET_PERL_PREFIX}" = "user" && ! test  "${SET_PERL_LIB}" = "user" ; then
 unset PERL_LIB
fi

dnl If the user sets the lib but not the prefix, clear out the prefix.
if ! test "${SET_PERL_PREFIX}" = "user" && test "${SET_PERL_LIB}" = "user" ; then
 unset PERL_PREFIX
fi

if test -n "$PERL_LIB" ; then
  PERL5_PATH=$PERL_LIB
fi

dnl  create the argument list for perl Makefile.PL 
dnl to create the PREFIX=  LIB=
PERL_INSTALL_PREFIX=""
if test -n "$PERL_PREFIX" ; then
  PERL_INSTALL_PREFIX="PREFIX=$PERL_PREFIX"
fi

if test -n "$PERL_LIB" ; then
  PERL_INSTALL_PREFIX="$PERL_INSTALL_PREFIX LIB=$PERL_LIB"
fi


dnl --with-perl-prefix= --with-perl-lib=

dnl If PERL_PREFIX is set which will happen if they specify a value or omit the argument altogether
dnl (and will not happen only if they specify --with-perl-prefix=   with no value given.)
# if test -n "$PERL_PREFIX" ; then
#  PERL_INSTALL_PREFIX="PREFIX=$PERL_PREFIX"
#  if ! test "${SET_PERL_PREFIX}" = "user" ; then
#    PERL5_PATH=${PERL_PREFIX}/perl
#    PERL_INSTALL_PREFIX="${PERL_INSTALL_PREFIX} LIB=${PERL_PREFIX}/perl"
#  fi
# fi

dnl Tell 
echo "*****************************************************"
echo ""
if  test -n "${PERL5_PATH}" ; then
  echo "       Set PERL5LIB to $PERL5_PATH"
else
  echo "Installing Perl module elements into standard Perl location." 
  echo "You may have to do this manually if the R configuration doesn't have permission to write to those directories."
  echo "Logged in as the relevant user, do "
  echo "    cd `pwd`/src "
  echo "    make -f Makefile.perl install"
fi
echo ""
echo "*****************************************************"

AC_SUBST(PERL5_PATH)

AC_SUBST(R_HOME)
AC_SUBST(R_LIBRARY_DIR)  # Where this package is being installed into.

AC_SUBST(PERL_INSTALL_PREFIX)

if test -n "${USE_SPLUS}" ; then
  # Allows the user to say --
  # This could be fooled, but unlikely
 if test -x ${with_splus} ; then
   SPLUS="${with_splus}"
 else
   SPLUS=Splus
 fi

   # Get the major version of the Splus being run.
 SPLUS_VERSION=`echo 'cat(version$major,"\n",sep="")' | ${SPLUS} | perl -e 'while(<STDIN>){ $x = $_;} printf $x;'`
  

   # If this is version 3, we are in trouble.
 if test ${SPLUS_VERSION} -lt 5 ; then
    echo "This package does not work with SPlus 3, but only SPlus 5 or 6" 
    exit 1
 fi

fi  
dnl End of USE_SPLUS




dnl -----------------------------------------------------------------------------------

R_SHLIB_LDFLAGS=`$R_HOME/bin/R CMD config --ldflags`
R_SO_DIR=`echo $R_SHLIB_LDFLAGS | sed -e 's/-L\(.*\) -lR.*/\1/'`
R_SHLIB_CPPFLAGS=`$R_HOME/bin/R CMD config --cppflags`
OTHERLDFLAGS=`echo $R_SHLIB_LDFLAGS | sed -e 's/-L.* -lR\(.*\)/\1/'`


dnl Used to test 
dnl    if test  "$R_SO_DIR" = "$R_SHLIB_FLAGS" ; then ...
echo "Testing: $R_SHLIB_LDFLAGS"
if test "$R_SHLIB_LDFLAGS" = "" ; then
  echo ""
  if test -n "$FORCE_WITH_IN_PERL" ; then
     echo "You have explicitly asked to have R embedded in Perl, but"
     echo "R was not built as a shared library and so cannot be embedded"
     echo "Rebuild R from source, e.g. with ./configure --enable-R-shlib"
     echo "and try to install this package after that has completed"
     exit 2
  else 
      echo "Cannot provide support for R to be embedded within Perl unless R is compiled as a R shared library."
      R_SO_DIR=""
      with_in_perl="no"
      R_IN_PERL_LIBRARY=""
      if test "${R_SHLIB_CPPFLAGS}" = "" ; then
         if test -r "$R_HOME/include/Rinternals.h" ; then 
	          R_SHLIB_CPPFLAGS="-I $R_HOME/include"
         else
	   echo "Cannot find Rinternals.h needed to build R.xs, (i.e. R.c)"
	   exit 4
	 fi
      fi
  fi
fi

dnl -----------------------------------------------------------------------------------



dnl Find the perl executable.

dnl PERL_MODULES=${PERL_MODULES-"no"}

if test -z "${PERL}" ; then
 AC_PATH_PROG(PERL, perl)
fi

if test -z "${PERL}" ; then
 echo "You don't seem to have Perl installed - very bad news when trying to run it from within R!"
 echo "Change your path to include the perl executable and re-install this package" 
 exit 1
else
 echo "Using '$PERL' as the perl executable"
fi

AC_SUBST(PERL)


if test -n "${with_modules}" ; then
  if test "${with_modules}" = "no"; then
    PERL_MODULES=""
  else
    if ! test "$with_modules" = "yes" ; then
      PERL_MODULES=$with_modules
    fi
  fi
else
  if test "${PERL_MODULES}"  = "no" ; then
   PERL_MODULES=""
   with_modules="no"
  fi
fi


echo "Perl modules (${with_modules}): ${PERL_MODULES}"

 dnl This iterates over the modules and checks that there is 
 dnl some C code to load for that module.
 dnl This will have already been done if $with_modules = "all"
 dnl but we can do it again without harm and to be certain.

if test "${with_modules}" != "no" ; then
 if test -z "${PERL_MODULES}" || test "${PERL_MODULES}" = "yes" ; then
   echo "Searching for Perl modules with native/compiled code"
   PERL_MODULES=`${PERL} -s modules.pl -modules`
   PERL_MODULE_SO_DIRS=`${PERL} modules.pl`
 else 
   PERL_MODULE_SO_DIRS=`${PERL} modules.pl ${PERL_MODULES}`
 fi
fi

# Now have to get our own package for calling R from Perl
# into this list if we want to be able to call back to R
# from Perl code (called from R).

#if test -n "${R_IN_PERL_LIBRARY}" ; then
  echo "Adding R package to list of Perl modules to enable callbacks to R from Perl"
  PERL_MODULES="${PERL_MODULES} R"
dnl  PERL_MODULE_SO_DIRS="${PERL_MODULE_SO_DIRS} ${R_PACKAGE_DIR}/share/blib/arch/auto/R/R.so"
#fi



 dnl Now we have to create the C code that will perform the loading
 dnl of the special modules.
 dnl Perl has a module (Embed) that generates this automatically
 dnl for us if we give the list of modules (i.e. ${PERL_MODULES})
if test -n "${PERL_MODULES}" ; then
  echo "Creating the C code for dynamically loading modules with native code for Perl: ${PERL_MODULES}"
  ${PERL} -MExtUtils::Embed -e xsinit -- -o src/xsinit.c -std ${PERL_MODULES}
fi


if test -n "$PERL_MODULES" ; then
  echo "modules:  ${PERL_MODULES}; linking: $PERL_MODULE_SO_DIRS"
    dnl arrange to trim the string before splitting to get rid of extraneous elements.
  R_PERL_MODULES=`$PERL -e '$x = $ARGV[[0]]; $x =~ s/^\s+//; $x =~ s/\s+$//; print "c(", join(", ", map { "\"" . $_ . "\""} split("[[ ]]+", $x)), ")"'  "$PERL_MODULES" `
else
  R_PERL_MODULES="character(0)"
fi
AC_SUBST(R_PERL_MODULES)


OLD_CFLAGS=${CFLAGS}
CFLAGS="`${PERL} -MExtUtils::Embed -e ccopts` -DPERL_POLLUTE"
AC_TRY_COMPILE([#include <EXTERN.h>
#include <perl.h>],
  [dTHX;],, LANGUAGE_DEFS="${LANGUAGE_DEFS} -DdTHX='int dthx_unused'"; echo "Adding definition for dTHX")
CFLAGS=${OLD_CFLAGS}

if test -z "${USE_SPLUS}" ; then
LANGUAGE_DEFS="${LANGUAGE_DEFS} -D_R_=1 -DUSE_R=1 -DUSE_TOPLEVEL_EXEC=1"
PKG_SYS_FILE='system.file("scripts", name, package="RSPerl")'
else
 LANGUAGE_DEFS="${LANGUAGE_DEFS} -D_S_=1 -DUSE_S=1 -D_S4_=1 -D_SPLUS${SPLUS_VERSION}_ -DNO_SPLUS_THREAD_DEF=1"
 INSTALL_DIR=`pwd`
 PKG_SYS_FILE="paste(\"${INSTALL_DIR}/inst/scripts/\", name,sep=\"\")"
fi

LD_PATH=$R_PACKAGE_DIR/libs

AC_SUBST(LANGUAGE_DEFS)
AC_SUBST(LD_PATH)
AC_SUBST(PERL_MODULE_SO_DIRS)



LD_PATH_VAR="LD_LIBRARY_PATH"

if test `uname` = "Darwin" ; then 
 LD_PATH_VAR="DYLD_LIBRARY_PATH"
 PERL_LD_MAKE_ARG="LD=/usr/bin/gcc"
fi

AC_SUBST(PERL_LD_MAKE_ARG)
AC_SUBST(OTHERLDFLAGS)

AC_SUBST(R_SO_DIR)

AC_SUBST(LD_PATH_VAR)


echo "Support R in Perl: ${with_in_perl}"
if test "${with_in_perl}" = "yes" ; then
   LANGUAGE_DEFS="${LANGUAGE_DEFS} -DWITH_R_IN_PERL=1"
   R_IN_PERL_DEFINES="-DWITH_R_IN_PERL=1"
   NO_SUPPORT_R_IN_PERL="0"
else
   NO_SUPPORT_R_IN_PERL="1"
fi

AC_SUBST(NO_SUPPORT_R_IN_PERL)
AC_SUBST(R_IN_PERL_DEFINES)

if test "${with_in_perl}" = "no" && test -z "${PERL_MODULES}" ; then
  LANGUAGE_DEFS="${LANGUAGE_DEFS} -DNO_XS_GENERATED"
fi


AC_SUBST(R_PACKAGE_DIR)


AC_OUTPUT(src/Makevars inst/scripts/RSPerl.csh inst/scripts/RSPerl.bsh src/RinPerlMakefile src/Makefile.PL cleanup src/R.pm R/perl5lib.R)
# cleanup is now not used.
chmod +x cleanup

if test -n "${USE_SPLUS}" ; then
 dnl We need to move the PseudoThreadAliases.h 
 dnl and modify it to remove the Thread typedef.
 cd src
 ${SPLUS} CHAPTER
 SHOME=`${SPLUS} SHOME`
 if test -f ${SHOME}/include/PseudoThreadAliases.h  ; then
   ${PERL} -ne 'print $_ unless m/^typedef.*Thread;$/' $SHOME/include/PseudoThreadAliases.h > PseudoThreadAliases.h
 fi
 echo "include Makevars" >> makefile
 echo 'CFLAGS:= $(PKG_CPPFLAGS) $(CFLAGS)' >> makefile
 echo 'LOCAL_LIBS=$(PKG_LIBS) ' >> makefile
fi


  cd src
  if test ! -d ${R_PACKAGE_DIR}/share ; then
    mkdir ${R_PACKAGE_DIR}/share
  fi


if test -n "${R_IN_PERL_LIBRARY}" ; then
  echo "making target all in RinPerlMakefile"
  PERL=$PERL R_INCLUDE_DIR=$R_HOME/include R_SHARE_DIR=$R_HOME/share R_HOME=$R_HOME ${MAKE-make} -f RinPerlMakefile 
  echo "calling make -f Makefile.perl install"
  PERL=$PERL R_INCLUDE_DIR=$R_HOME/include R_SHARE_DIR=$R_HOME/share R_HOME=$R_HOME ${MAKE-make} -f Makefile.perl install

else
   #XXX Need to install R.pm, RReferences.pm but not build the library.
   # So the install needs to bypasss building the C code for the Perl library.
  echo "just installing the relevant Perl pieces and not building the embeddable R Perl module"
  PERL=$PERL R_INCLUDE_DIR=$R_HOME/include R_SHARE_DIR=$R_HOME/share R_HOME=$R_HOME ${MAKE-make} -f RinPerlMakefile Makefile.perl
  PERL=$PERL R_INCLUDE_DIR=$R_HOME/include R_SHARE_DIR=$R_HOME/share R_HOME=$R_HOME ${MAKE-make} -f Makefile.perl install

fi

   # Why are we doing this again? Is there a chance that
   # either of make calls above could delete it. Oh well, it 
   # doesn't hurt.
  if test ! -d ${R_PACKAGE_DIR}/share ; then
    mkdir ${R_PACKAGE_DIR}/share
  fi

  chmod +w blib/lib/R.pm

  if test ! -d ${R_PACKAGE_DIR}/examples ; then
	mkdir ${R_PACKAGE_DIR}/examples
  fi
  cp test.pl ${R_PACKAGE_DIR}/examples



echo "Finished configuration"

