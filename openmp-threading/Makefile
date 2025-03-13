#===============================================================================
# User Options
#===============================================================================

# Compiler can be set below, or via environment variable
ifeq ($(CC),)
CC    = clang
endif
OPTIMIZE    = yes
OPENMP      = no
OBJSAN      = yes
DEBUG       = no
PROFILE     = no
MPI         = no
AML         = no


LLVM_DIR=/scr/ivan/src/input-gen/
#===============================================================================
# Program name & source code list
#===============================================================================

program = XSBench

source = \
Main.c \
io.c \
Simulation.c \
GridInit.c \
XSutils.c \
Materials.c

obj = $(source:.c=.o)

#===============================================================================
# Sets Flags
#===============================================================================

# Make sure we didn't pick up lower case cc
ifeq (cc,$(CC))
CC    = clang
endif

# Standard Flags
CFLAGS := -std=gnu99 -Wall

# Linker Flags
LDFLAGS = -lm

# Regular gcc Compiler
ifneq (,$(findstring gcc,$(CC)))
  CFLAGS += -flto
  ifeq ($(OPENMP),yes)
	CFLAGS += -fopenmp -DOPENMP
  endif
endif

# LLVM Compiler
ifneq (,$(findstring clang,$(CC)))
  CFLAGS += -flto
  LDFLAGS += -fuse-ld=lld
  ifeq ($(OPENMP),yes)
	CFLAGS += -fopenmp -DOPENMP
  endif
  ifeq ($(OBJSAN),yes)
  CFLAGS += -flto=full
  LDFLAGS += -fsanitize=object -flto -fuse-ld=lld -Wl,-mllvm -Wl,-instrumentor-runtime-bitcode=objsan_ir_rt.bc
  program=XSBench.objsan
	obj = $(source:.c=.objsan.o)
  endif
endif

# Intel Compiler
ifneq (,$(findstring intel,$(CC)))
  CFLAGS += -ipo
  ifeq ($(OPENMP),yes)
	CFLAGS += -fopenmp -DOPENMP
  endif
endif

# Debug Flags
ifeq ($(DEBUG),yes)
  CFLAGS += -g
  LDFLAGS  += -g
endif

# Profiling Flags
ifeq ($(PROFILE),yes)
  CFLAGS += -pg
  LDFLAGS  += -pg
endif

# Optimization Flags
ifeq ($(OPTIMIZE),yes)
  CFLAGS += -O3
endif

# MPI
ifeq ($(MPI),yes)
  CC = mpicc
  CFLAGS += -DMPI
endif

# AML
ifeq ($(AML),yes)
  LDFLAGS += $(shell pkg-config --libs aml)
  CFLAGS += -DAML $(shell pkg-config --cflags aml)
endif

#===============================================================================
# Targets to Build
#===============================================================================

$(program): $(obj) XSbench_header.h Makefile objsan_ir_rt.bc
	$(CC) $(CFLAGS) $(obj) -o $@ $(LDFLAGS)

%.objsan.o: %.c XSbench_header.h Makefile
	$(CC) $(CFLAGS) -c $< -o $@

%.o: %.c XSbench_header.h Makefile
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -rf $(program) $(obj)

edit:
	vim -p $(source) XSbench_header.h

objsan_ir_rt.bc: $(LLVM_DIR)/compiler-rt/lib/objsan/objsan_ir_rt.cpp
	$(CC) -c -emit-llvm -O3 $(LLVM_DIR)/compiler-rt/lib/objsan/objsan_ir_rt.cpp -o objsan_ir_rt.bc

run:
	./$(program)
