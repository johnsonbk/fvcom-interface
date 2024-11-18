##############
Building FVCOM
##############

The FVCOM source code is `available in a repository on Github.com
<https://github.com/FVCOM-GitHub/FVCOM>`_.

.. code-block::

   git clone https://github.com/FVCOM-GitHub/FVCOM.git
   cd FVCOM/src
   vim make.inc

   TOPDIR        = /Users/$(USER)/work/git/FVCOM/src
   IOINCS       =  -I/opt/local/include
   IOLIBS       =  -L/opt/local/lib  -lnetcdff -lnetcdf

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
