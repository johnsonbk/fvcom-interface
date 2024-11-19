##############
Building FVCOM
##############

The FVCOM source code is `available in a repository on Github.com
<https://github.com/FVCOM-GitHub/FVCOM>`_.

.. code-block::

   git clone https://github.com/FVCOM-GitHub/FVCOM.git
   cd FVCOM/src

Installing dependencies using MacPorts
======================================

.. note::

   FVCOM requires the `"Legacy C Library" version of Julian
   <https://pds-rings.seti.org/toolkits/>`_. Installing the python-based
   version of Julian using ``pip install rms-julian`` will result in the
   following build error:

   .. error::

      ``Fatal Error: Cannot open included file 'fjulian.inc'``


.. code-block::

   su ${ADMIN}
   sudo port selfupdate
   sudo port upgrade outdated
   sudo port install proj
   sudo port install metis
   sudo port install PetSc
   sudo port install esmf

Installing remaining dependency: Julian
=======================================

This modification of ``./makemac.com`` works until the final line, which tries 
to link the static C library into the fortran executable.

.. code-block::

   cc -c \
   dates.c \
   format.c \
   juldates.c \
   leapsecs.c \
   parse.c \
   seconds.c \
   tai_et.c \
   utc_tai.c
   gfortran -c fjulian.for

   cc -c fortran.c rlerrors.c rlmemory.c
   gfortran -c fstrings.for

   ar cr julian.a \
   dates.o \
   format.o \
   juldates.o \
   leapsecs.o \
   parse.o \
   seconds.o \
   tai_et.o \
   utc_tai.o \
   fjulian.o \
   fortran.o \
   rlerrors.o \
   rlmemory.o \
   fstrings.o \

   ranlib julian.a

   rm *.o

   gfortran -f -o tconvert tconvert.for julian.a

This is the error message I recieve when trying to link the static C library to
the ``tconvert`` Fortran executable using ``gfortran``.

.. error::

    .. code-block::

        gfortran tconvert.o -L/opt/local/julian_133 -ljulian -o tconvert
        ld: library not found for -ljulian
        collect2: error: ld returned 1 exit status

Trying to compile FVCOM
=======================

Regardless, the static library ``julian.a`` seemed to compile alright. Can I 
link it to FVCOM and compile it?

.. code-block::

   vim make.inc

   TOPDIR       = /Users/$(USER)/work/git/FVCOM/src
   IOINCS       =  -I/opt/local/include -I/opt/local/julian_133
   IOLIBS       =  -L/opt/local/lib -lnetcdff -lnetcdf -L/opt/local/julian_133 -ljulian

The edit to ``make.inc`` adds the paths to the location FVCOM is installed and
the lib and include directories. There are two sets of lib and include
directories. The ``/opt/local/lib`` and ``/opt/local/include`` directories are 
where the MacPorts-installed dependencies are located. The
``/opt/local/julian_133`` directory is where Julian is installed.

However, the `compile attempt <using the FVCOM installation instructions 
https://github.com/FVCOM-GitHub/FVCOM>`_ is unsuccessful.

.. code-block::

   make clean
   make

.. error::

   .. code-block::

      mpif90  -c  -O3  -I/opt/local/include -I/opt/local/julian_133        mod_time.f90
      fjulian.inc:1:1:
      Error: Unclassifiable statement at (1)
      fjulian.inc:18:57:
      [ ... ]
      mod_time.f90:663:21:
      663 |      mjd%mjd = ANINT(FJul_MJDofTAI(tai, FJUL_UTC_TYPE),itime)
          |                     1
      Error: Function 'fjul_mjdoftai' at (1) has no IMPLICIT type; did you mean 'fjul_taiofdutc'?
      mod_time.f90:695:12:
      695 |      tai  = FJul_TAIofMJD(rMJD, FJUL_UTC_TYPE)
          |            1
      Error: Function 'fjul_taiofmjd' at (1) has no IMPLICIT type; did you mean 'fjul_dutcoftai'?
      make: *** [mod_time.o] Error 1

Guidance from Gemini
====================

Gemini has the following advice when using the search phrase, "linking static c
library to fortran executable gfortran."

   Function Names:
   Make sure the function names in your C code match the names you use in your
   Fortran code. Fortran compilers may modify the names of external functions.
   
   Calling Conventions:
   C and Fortran may use different calling conventions. You may need to use
   special directives or wrappers to ensure that the calling conventions match.
   
   Data Types:
   Be careful when passing data between C and Fortran. Ensure that the data
   types match or use appropriate conversion functions.
   
   Include Paths:
   If your C library has header files, you may need to include them in your
   Fortran code using the #include directive.
