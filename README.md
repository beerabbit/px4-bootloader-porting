# px4-bootloader-porting

STM32H7* Board용으로 PX4 Bootloader을 포팅하자.

px4fmuv5_bl으로 porting 할 예정

MakeFile으로 찾아본 px4fmuv5_b make 방식
```make
${MAKE} ${MKFLAGS} -f  Makefile.f7 TARGET_HW=PX4_FMU_V5 LINKER_FILE=stm32f7.ld TARGET_FILE_NAME=$@
```

1. for get bootsubmodule (lib 다운로드)
  git submodule sync --recursive
  git submodule update --init --recursive
  make
 
2. Makefile.7 파일
```make
${MAKE} ${MKFLAGS} -f  Makefile.f7 TARGET_HW=PX4_FMU_V5 LINKER_FILE=stm32f7.ld TARGET_FILE_NAME=$@

#
# PX4 bootloader build rules for STM32F7 targets.
# Since the STM32F7 is not supported fully in libopencm3 at this time,
# this build is a bit of a hack to use the similar IP blocks of the F469
# for USB by telling libopencm3 it is an STM32F4
ARCH=stm32
OPENOCD		?= openocd

JTAGCONFIG ?= interface/olimex-jtag-tiny.cfg
#JTAGCONFIG ?= interface/jtagkey-tiny.cfg

# 5 seconds / 5000 ms default delay
PX4_BOOTLOADER_DELAY	?= 5000

SRCS		 = $(COMMON_SRCS) $(addprefix $(ARCH)/,$(ARCH_SRCS)) main_f7.c
LIBS			= opencm3_stm32f7 opencm3_stm32f4

FLAGS			+= -g -mthumb -mcpu=cortex-m7 -mfloat-abi=hard -mfpu=fpv5-sp-d16 \
				-DTARGET_HW_$(TARGET_HW) \
				-T$(LINKER_FILE) \
				-L$(LIBOPENCM3)/lib $(addprefix -l, $(LIBS)) \
				-DSTM32F7

# libopencm3 doesn't have USB driver in stm32f7 library, tell it to use the one from stm32f4
# for the cdcacm
$(BUILD_DIR_ROOT)/$(TARGET_FILE_NAME)/stm32/cdcacm.o : FLAGS+=-DSTM32F4
#
# General rules for making dependency and object files
# This is where the compiler is called
#
include rules.mk

#upload: all flash flash-bootloader
upload: all flash-bootloader

flash-bootloader:
	$(OPENOCD) --search ../px4_bootloader -f $(JTAGCONFIG) -f stm32f4x.cfg -c init -c "reset halt" -c "flash write_image erase $(BINARY) 0x08000000" -c "reset run" -c shutdown

```
libopencm3 이 뭐지 => libopencm3 프로젝트는 다양한 ARM Cortex-M 마이크로 컨트롤러 용 오픈 소스 펌웨어 라이브러리이다.



