# Makefile for FINUFFT.
# Barnett 7/30/18

# This is the only makefile; there are no makefiles in subdirectories.
# Users should not need to edit this makefile (doing so would make it hard to
# stay up to date with the repo version). Rather, in order to change
# OS/environment-specific compilers and flags, create the file make.inc, which
# overrides the defaults below (which are for ubuntu linux/GCC system).
# See docs/install.rst, and make.inc.*

# compilers, and linking from C, fortran...
CXX=g++
CC=gcc
FC=gfortran
CLINK=-lstdc++
FLINK=$(CLINK)
# compile flags for baseline single-threaded, double precision case...
# (note -Ofast breaks isfinite() and isnan(), so use -O3 which now is as fast)
CFLAGS   = -fPIC -O3 -funroll-loops -march=native -I/nfs/rtsan02/userdata/home/nhutting/Downloads/libfftw3/usr/include -L/nfs/rtsan02/userdata/home/nhutting/Downloads/libfftw3/usr/lib/x86_64-linux-gnu
# tell examples where to find header files...
CFLAGS   += -I src 
FFLAGS   = $(CFLAGS)
CXXFLAGS = $(CFLAGS) -DNEED_EXTERN_C
# FFTW base name, and math linking...
FFTWNAME=fftw3
LIBS = -lm -L/nfs/rtsan02/userdata/home/nhutting/Downloads/libfftw3
# extra flags for multithreaded: C++/C/Fortran, MATLAB, and octave...
OMPFLAGS = -fopenmp
MOMPFLAGS = -lgomp -D_OPENMP
OOMPFLAGS = -lgomp
# flags for MATLAB MEX compilation...
MFLAGS=-largeArrayDims -I/nfs/rtsan02/userdata/home/nhutting/Downloads/libfftw3/usr/include -L/nfs/rtsan02/userdata/home/nhutting/Downloads/libfftw3/usr/lib/x86_64-linux-gnu
# location of MATLAB's mex compiler...
MEX=/nfs/rtsan01/groupdata/packages/stow/Matlab2015a/bin/mex
# flags for octave mkoctfile...
OFLAGS=
# For experts, location of MWrap executable (see docs/install.rst):
MWRAP=mwrap

# For your OS, override the above by placing make variables in make.inc ...
-include make.inc

# choose the precision (affects library names, test precisions)...
ifeq ($(PREC),SINGLE)
CXXFLAGS += -DSINGLE
CFLAGS += -DSINGLE
# note that PRECSUFFIX is used to choose fftw lib name, and also our demo names
PRECSUFFIX=f
REQ_TOL = 1e-6
CHECK_TOL = 2e-4
else
PRECSUFFIX=
REQ_TOL = 1e-12
CHECK_TOL = 1e-11
endif
# build (since fftw has many) names of libs to link against...
FFTW = $(FFTWNAME)$(PRECSUFFIX)
LIBSFFT = -l$(FFTW) $(LIBS)

# multi-threaded libs & flags needed (see defns above)...
ifneq ($(OMP),OFF)
CXXFLAGS += $(OMPFLAGS)
CFLAGS += $(OMPFLAGS)
FFLAGS += $(OMPFLAGS)
MFLAGS += $(MOMPFLAGS)
OFLAGS += $(OOMPFLAGS)
LIBSFFT += -l$(FFTW)_threads
endif

# decide name of obj files and finufft library we're building...
LIBNAME=libfinufft$(PRECSUFFIX)
DYNAMICLIB = lib/$(LIBNAME).so
STATICLIB = lib-static/$(LIBNAME).a


# ======================================================================

# objects to compile: spreader...
SOBJS = src/spreadinterp.o src/utils.o
# for NUFFT library and its testers...
OBJS = $(SOBJS) src/finufft1d.o src/finufft2d.o src/finufft3d.o src/dirft1d.o src/dirft2d.o src/dirft3d.o src/common.o contrib/legendre_rule_fast.o src/finufft_c.o fortran/finufft_f.o
# just the dimensions (1,2,3) separately...
OBJS1 = $(SOBJS) src/finufft1d.o src/dirft1d.o src/common.o contrib/legendre_rule_fast.o
OBJS2 = $(SOBJS) src/finufft2d.o src/dirft2d.o src/common.o contrib/legendre_rule_fast.o
OBJS3 = $(SOBJS) src/finufft3d.o src/dirft3d.o src/common.o contrib/legendre_rule_fast.o
# for Fortran interface demos...
FOBJS = fortran/dirft1d.o fortran/dirft2d.o fortran/dirft3d.o fortran/dirft1df.o fortran/dirft2df.o fortran/dirft3df.o fortran/prini.o

HEADERS = src/spreadinterp.h src/finufft.h src/dirft.h src/common.h src/utils.h src/finufft_c.h fortran/finufft_f.h

.PHONY: usage lib examples test perftest fortran matlab octave all mex python python3 clean objclean pyclean mexclean

default: usage

all: test perftest lib examples fortran matlab octave python3

usage:
	@echo "Makefile for FINUFFT library. Specify what to make:"
	@echo " make lib - compile the main library (in lib/ and lib-static/)"
	@echo " make examples - compile and run codes in examples/"
	@echo " make test - compile and run quick math validation tests"
	@echo " make perftest - compile and run performance tests"
	@echo " make fortran - compile and test Fortran interfaces"
	@echo " make matlab - compile MATLAB interfaces"
	@echo " make octave - compile and test octave interfaces"
	@echo " make python3 - compile and test python3 interfaces"	
	@echo " make all - do all the above (around 1 minute; assumes you have MATLAB, etc)"
	@echo " make python - compile and test python (v2) interfaces"
	@echo " make spreadtest - compile and run spreader tests only"
	@echo " make objclean - remove all object files, preserving lib & MEX"
	@echo " make clean - also remove lib, MEX, py, and demo executables"
	@echo "For faster (multicore) making, append the flag -j"
	@echo ""
	@echo "Compile options: 'make [task] PREC=SINGLE' for single-precision"
	@echo " 'make [task] OMP=OFF' for single-threaded (otherwise OpenMP)"
	@echo " You must 'make objclean' before changing such options!"
	@echo ""
	@echo "Also see docs/install.rst"

# implicit rules for objects (note -o ensures writes to correct dir)
%.o: %.cpp %.h
	$(CXX) -c $(CXXFLAGS) $< -o $@
%.o: %.c %.h
	$(CC) -c $(CFLAGS) $< -o $@
%.o: %.f %.h
	$(FC) -c $(FFLAGS) $< -o $@

# included code dependency...
src/spreadinterp.o: src/ker_horner_allw_loop.c src/ker_lowupsampfac_horner_allw_loop.c

# build the library...
lib: $(STATICLIB) $(DYNAMICLIB)
ifeq ($(OMP),OFF)
	echo "$(STATICLIB) and $(DYNAMICLIB) built, single-thread versions"
else
	echo "$(STATICLIB) and $(DYNAMICLIB) built, multithreaded versions"
endif
$(STATICLIB): $(OBJS) $(HEADERS)
	ar rcs $(STATICLIB) $(OBJS)
$(DYNAMICLIB): $(OBJS) $(HEADERS)
	$(CXX) -shared $(OMPFLAGS) $(LIBSFFT) $(OBJS) -o $(DYNAMICLIB)
# here $(OMPFLAGS) $(LIBSFFT) is needed for mac osx.
# see: http://www.cprogramming.com/tutorial/shared-libraries-linux-gcc.html

# examples in C++ and C... (separate codes for double vs single prec)
EX = examples/example1d1$(PRECSUFFIX)
EXC = examples/example1d1c$(PRECSUFFIX)
examples: $(EX) $(EXC)
	./$(EX)
	./$(EXC)
$(EX): $(EX).o $(STATICLIB)
	$(CXX) $(CXXFLAGS) $(EX).o $(STATICLIB) $(LIBSFFT) -o $(EX)
$(EXC): $(EXC).o $(STATICLIB)
	$(CC) $(CFLAGS) $(EXC).o $(STATICLIB) $(LIBSFFT) $(CLINK) -o $(EXC)

# validation tests... (most link to .o allowing testing pieces separately)
test: $(STATICLIB) test/testutils test/finufft1d_test test/finufft2d_test test/finufft3d_test test/dumbinputs test/finufft2dmany_test
	(cd test; \
	export FINUFFT_REQ_TOL=$(REQ_TOL); \
	export FINUFFT_CHECK_TOL=$(CHECK_TOL); \
	./check_finufft.sh)
test/testutils: test/testutils.cpp src/utils.o src/utils.h $(HEADERS)
	$(CXX) $(CXXFLAGS) test/testutils.cpp src/utils.o -o test/testutils
test/finufft1d_test: test/finufft1d_test.cpp $(OBJS1) $(HEADERS)
	$(CXX) $(CXXFLAGS) test/finufft1d_test.cpp $(OBJS1) $(LIBSFFT) -o test/finufft1d_test
test/finufft2d_test: test/finufft2d_test.cpp $(OBJS2) $(HEADERS)
	$(CXX) $(CXXFLAGS) test/finufft2d_test.cpp $(OBJS2) $(LIBSFFT) -o test/finufft2d_test
test/finufft3d_test: test/finufft3d_test.cpp $(OBJS3) $(HEADERS)
	$(CXX) $(CXXFLAGS) test/finufft3d_test.cpp $(OBJS3) $(LIBSFFT) -o test/finufft3d_test
test/dumbinputs: test/dumbinputs.cpp $(STATICLIB) $(HEADERS)
	$(CXX) $(CXXFLAGS) test/dumbinputs.cpp $(STATICLIB) $(LIBSFFT) -o test/dumbinputs
test/finufft2dmany_test: test/finufft2dmany_test.cpp $(OBJS2) $(HEADERS)
	$(CXX) $(CXXFLAGS) test/finufft2dmany_test.cpp $(OBJS2) $(LIBSFFT) -o test/finufft2dmany_test

# performance tests...
perftest: test/spreadtestnd test/finufft1d_test test/finufft2d_test test/finufft3d_test
# here the tee cmd copies output to screen. 2>&1 grabs both stdout and stderr...
	(cd test; ./spreadtestnd.sh 2>&1 | tee results/spreadtestnd_results.txt)
	(cd test; ./nuffttestnd.sh 2>&1 | tee results/nuffttestnd_results.txt)
test/spreadtestnd: test/spreadtestnd.cpp $(SOBJS) $(HEADERS)
	$(CXX) $(CXXFLAGS) test/spreadtestnd.cpp $(SOBJS) -o test/spreadtestnd

# spreader only test (useful for development work on spreader)...
spreadtest: test/spreadtestnd
	test/spreadtestnd 1 8e6 8e6 1e-6 1 0
	test/spreadtestnd 2 8e6 8e6 1e-6 1 0
	test/spreadtestnd 3 8e6 8e6 1e-6 1 0

# --------------- LANGUAGE INTERFACES -----------------------
# fortran interface...
F1=fortran/nufft1d_demo$(PRECSUFFIX)
F2=fortran/nufft2d_demo$(PRECSUFFIX)
F3=fortran/nufft3d_demo$(PRECSUFFIX)
F4=fortran/nufft2dmany_demo$(PRECSUFFIX)
fortran: $(FOBJS) $(OBJS) $(HEADERS)
	$(FC) $(FFLAGS) $(F1).f $(FOBJS) $(OBJS) $(LIBSFFT) $(FLINK) -o $(F1)
	$(FC) $(FFLAGS) $(F2).f $(FOBJS) $(OBJS) $(LIBSFFT) $(FLINK) -o $(F2)
	$(FC) $(FFLAGS) $(F3).f $(FOBJS) $(OBJS) $(LIBSFFT) $(FLINK) -o $(F3)
	$(FC) $(FFLAGS) $(F4).f $(FOBJS) $(OBJS) $(LIBSFFT) $(FLINK) -o $(F4)
	time -p $(F1)
	time -p $(F2)
	time -p $(F3)
	time -p $(F4)

# matlab .mex* executable...
matlab: $(STATICLIB) $(HEADERS) matlab/finufft_m.cpp
ifeq ($(PREC),SINGLE)
	@echo "MATLAB interface only supports double precision; doing nothing"
else
	$(MEX) matlab/finufft.cpp $(STATICLIB) matlab/finufft_m.cpp $(MFLAGS) $(LIBSFFT) -output matlab/finufft
endif

# octave .mex executable... (also creates matlab/finufft.o for some reason)
octave: $(STATICLIB) $(HEADERS) matlab/finufft_m.cpp
ifeq ($(PREC),SINGLE)
	@echo "Octave interface only supports double precision; doing nothing"
else
	(cd matlab; mkoctfile --mex finufft.cpp ../$(STATICLIB) finufft_m.cpp $(OFLAGS) $(LIBSFFT) -output finufft)
	@echo "Running octave interface test; please wait a few seconds..."
	(cd matlab; octave check_finufft.m)
endif

# for experts; force rebuilds fresh MEX (matlab/octave) gateway via mwrap...
# (needs mwrap)
mex: matlab/finufft.mw
	(cd matlab;\
	$(MWRAP) -list -mex finufft -cppcomplex -mb finufft.mw ;\
	$(MWRAP) -mex finufft -c finufft.cpp -cppcomplex finufft.mw )

# python(3) interfaces...
python: $(STATICLIB)
ifeq ($(PREC),SINGLE)
	@echo "python interface only supports double precision; doing nothing"
else
	pip install .
	python python_tests/demo1d1.py
	python python_tests/run_accuracy_tests.py
endif
python3: $(STATICLIB)
ifeq ($(PREC),SINGLE)
	@echo "python3 interface only supports double precision; doing nothing"
else
	pip3 install .
	python3 python_tests/demo1d1.py
	python3 python_tests/run_accuracy_tests.py
endif


# ------------- Various obscure/devel tests -----------------
# This was for a CCQ application; zgemm was 10x faster!
manysmallprobs: $(STATICLIB) $(HEADERS) test/manysmallprobs.cpp
	$(CXX) $(CXXFLAGS) test/manysmallprobs.cpp $(STATICLIB) -o test/manysmallprobs $(LIBSFFT)
	(export OMP_NUM_THREADS=2; time test/manysmallprobs; unset OMP_NUM_THREADS)


# ------------- Cleaning up (including all versions of lib, and interfaces)...
clean: objclean pyclean
	rm -f lib-static/*.a lib/*.so
	rm -f matlab/*.mex*
	rm -f test/spreadtestnd test/finufft?d_test test/testutils test/manysmallprobs test/results/*.out fortran/nufft?d_demo fortran/nufft?d_demof examples/example1d1 examples/example1d1c examples/example1d1f examples/example1d1cf

# this is needed before changing precision or threading...
objclean:
	rm -f $(OBJS) $(SOBJS)
	rm -f fortran/*.o examples/*.o matlab/*.o

pyclean:
	rm -f finufftpy/*.pyc finufftpy/__pycache__/* python_tests/*.pyc python_tests/__pycache__/*

# for experts; only do this if you have mwrap to rebuild the interfaces!
mexclean:
	rm -f matlab/finufft.cpp matlab/finufft?d?.m matlab/finufft.mex*
