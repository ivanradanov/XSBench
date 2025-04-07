#===============================================================================
# User Options
#===============================================================================

CONFIG    = amd_none
# CONFIG    = amd_asan
# CONFIG    = clang_none
# CONFIG    = clang_objsan

OPTIMIZE    = yes
DEBUG       = no
PROFILE     = no

ROCM_PATH=/opt/rocm
GPU_ARCH=gfx1030
LLVM_DIR=/scr/ivan/src/input-gen/

#===============================================================================
# Program name & source code list
#===============================================================================

program = XSBench

source = \
Main.cpp \
io.cpp \
Simulation.cpp \
GridInit.cpp \
XSutils.cpp \
Materials.cpp

#===============================================================================
# Sets Flags
#===============================================================================

# Standard Flags
CFLAGS += -std=c++14

# Linker Flags
LDFLAGS = -lm

# AMD
ifeq ($(CONFIG),amd_none)
  CC = hipcc
endif

ifeq ($(CONFIG),amd_none)
  CC = hipcc
endif

ifeq ($(CONFIG),clang_none)
  CC = clang++
  program=XSBench.clang_none
	obj = $(source:.cpp=.clang_none.o)
endif

ifeq ($(CONFIG),clang_objsan)
  CC = clang++
  CFLAGS += -fsanitize=object -mllvm -objsan-runtime-bitcode=objsan_ir_rt.bc -I$(LLVM_DIR)/compiler-rt/lib/objsan/include/
  LDFLAGS += -fsanitize=object -Wl,-mllvm -Wl,-objsan-runtime-bitcode=objsan_ir_rt.bc
  program=XSBench.clang_objsan
	source += objsan_preload_impl_hip.cpp \
	  objsan_preload.cpp \
	  objsan_preload_hip.cpp
	obj = $(source:.cpp=.clang_objsan.o)
endif

ifneq (,$(findstring clang,$(CC)))
  COMMON_CFLAGS = -x hip -flto --rocm-path=$(ROCM_PATH) --offload-arch=$(GPU_ARCH) --offload-new-driver -foffload-lto -fgpu-rdc
  COMMON_LDFLAGS = --rocm-path=$(ROCM_PATH) -L$(ROCM_PATH)/lib -fuse-ld=lld -flto -lstdc++ -lamdhip64 --offload-link --offload-new-driver -foffload-lto -fgpu-rdc
  LDFLAGS += $(COMMON_LDFLAGS)
  CFLAGS += $(COMMON_CFLAGS)
endif

# Debug Flags
ifeq ($(DEBUG),yes)
  CFLAGS += -g
  LDFLAGS  += -g
  RT_CFLAGS += -g
  RT_LDFLAGS  += -g
endif

# Profiling Flags
ifeq ($(PROFILE),yes)
  CFLAGS += -pg
  LDFLAGS  += -pg
endif

# Optimization Flags
ifeq ($(OPTIMIZE),yes)
  CFLAGS += -O3
  LDFLAGS += -O3
  RT_CFLAGS += -O3
  RT_LDFLAGS  += -O3
endif

#===============================================================================
# Targets to Build
#===============================================================================

DEVICE_TARGET=amdgcn-amd-amdhsa
DEVICE_OBJSAN_IR_RT = objsan_ir_rt.bc.$(DEVICE_TARGET)
HOST_TARGET=x86_64-unknown-linux-gnu
HOST_OBJSAN_IR_RT = objsan_ir_rt.bc.$(HOST_TARGET)

HIP_PRELOAD=hip_preload.so

all: $(program) #$(HIP_PRELOAD)

$(program): $(obj) XSbench_header.h Makefile # $(HIP_PRELOAD)
	$(CC) $(obj) -o $@ $(LDFLAGS)

%.o: %.cpp XSbench_header.h Makefile
	$(CC) $(CFLAGS) -c $< -o $@

%.clang_objsan.o: %.cpp XSbench_header.h Makefile $(DEVICE_OBJSAN_IR_RT) $(HOST_OBJSAN_IR_RT)
	$(CC) $(CFLAGS) -c $< -o $@
%.clang_none.o: %.cpp XSbench_header.h Makefile
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -rf $(program) $(obj) $(DEVICE_OBJSAN_IR_RT) $(HOST_OBJSAN_IR_RT) objsan_preload*.cpp

edit:
	vim -p $(source) XSbench_header.h

run: $(program) $(HIP_PRELOAD)
	LD_PRELOAD=$(HIP_PRELOAD) ./$(program) -m event

#$(CC) -c -emit-llvm -O3 -target $(DEVICE_TARGET) -mcpu=$(GPU_ARCH) $(LLVM_DIR)/compiler-rt/lib/objsan/objsan_ir_rt.cpp -o $(DEVICE_OBJSAN_IR_RT) -D__OBJSAN_DEVICE__ -D__OBJSAN_DEVICE_DISABLE_PRINTF__ -DNDEBUG
$(DEVICE_OBJSAN_IR_RT): $(LLVM_DIR)/compiler-rt/lib/objsan/objsan_ir_rt.cpp
	$(CC) $(RT_CFLAGS) -c -emit-llvm -O3 -target $(DEVICE_TARGET) -mcpu=$(GPU_ARCH) $(LLVM_DIR)/compiler-rt/lib/objsan/objsan_ir_rt.cpp -o $(DEVICE_OBJSAN_IR_RT).rt -D__OBJSAN_DEVICE__ -D__OBJSAN_DEVICE_DISABLE_PRINTF__ -DNDEBUG
	$(CC) $(RT_CFLAGS) -c -emit-llvm -O3 -target $(DEVICE_TARGET) -mcpu=$(GPU_ARCH) $(LLVM_DIR)/compiler-rt/lib/objsan/objsan_rt.cpp -o $(DEVICE_OBJSAN_IR_RT).storage -D__OBJSAN_DEVICE__ -D__OBJSAN_DEVICE_DISABLE_PRINTF__ -DNDEBUG
	llvm-link $(DEVICE_OBJSAN_IR_RT).storage $(DEVICE_OBJSAN_IR_RT).rt -o $(DEVICE_OBJSAN_IR_RT)

$(HOST_OBJSAN_IR_RT): $(LLVM_DIR)/compiler-rt/lib/objsan/objsan_ir_rt.cpp
	$(CC) -c -emit-llvm -O3 -target $(HOST_TARGET) $(LLVM_DIR)/compiler-rt/lib/objsan/objsan_ir_rt.cpp -o $(HOST_OBJSAN_IR_RT) -D__OBJSAN_HOST__ -DNDEBUG

objsan_preload_impl_hip.cpp: $(LLVM_DIR)/compiler-rt/lib/objsan/preload/objsan_preload_impl_hip.cpp
	cp $(LLVM_DIR)/compiler-rt/lib/objsan/preload/objsan_preload_impl_hip.cpp ./
objsan_preload.cpp: $(LLVM_DIR)/compiler-rt/lib/objsan/preload/objsan_preload.cpp
	cp $(LLVM_DIR)/compiler-rt/lib/objsan/preload/objsan_preload.cpp ./
objsan_preload_hip.cpp: $(LLVM_DIR)/compiler-rt/lib/objsan/preload/objsan_preload_hip.cpp
	cp $(LLVM_DIR)/compiler-rt/lib/objsan/preload/objsan_preload_hip.cpp ./
