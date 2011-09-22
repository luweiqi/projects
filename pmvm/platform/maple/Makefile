.DEFAULT_GOAL := all


PM_INC := -I../../vm
CINCS = $(PM_INC) -I$(abspath .) 


TARGET = main


CPPSRCS := main.cpp

# Valid BOARDs: maple, maple_native, ...
BOARD ?= maple_native
MEMORY_TARGET ?= flash

# USB ID for DFU upload
VENDOR_ID  := 1EAF
PRODUCT_ID := 0003

# Guess the MCU based on the BOARD (can be overridden )
# FIXME the error LED config needs to be in wirish/ instead
ifeq ($(BOARD), maple)
   MCU := STM32F103RB
   PRODUCT_ID := 0003
   ERROR_LED_PORT := GPIOA
   ERROR_LED_PIN  := 5
   DENSITY := STM32_MEDIUM_DENSITY
endif
ifeq ($(BOARD), maple_native)
   MCU := STM32F103ZE
   PRODUCT_ID := 0003
   ERROR_LED_PORT := GPIOC
   ERROR_LED_PIN  := 15
   DENSITY := STM32_HIGH_DENSITY
endif
ifeq ($(BOARD), maple_mini)
   MCU := STM32F103CB
   PRODUCT_ID := 0003
   ERROR_LED_PORT := GPIOB
   ERROR_LED_PIN  := 1
   DENSITY := STM32_MEDIUM_DENSITY
endif
ifeq ($(BOARD), maple_RET6)
   MCU := STM32F103RE
   PRODUCT_ID := 0003
   ERROR_LED_PORT := GPIOA
   ERROR_LED_PIN := 5
   DENSITY := STM32_HIGH_DENSITY
endif

# Some target specific things
ifeq ($(MEMORY_TARGET), ram)
   LDSCRIPT := $(BOARD)/ram.ld
   VECT_BASE_ADDR := VECT_TAB_RAM
endif
ifeq ($(MEMORY_TARGET), flash)
   LDSCRIPT := $(BOARD)/flash.ld
   VECT_BASE_ADDR := VECT_TAB_FLASH
endif
ifeq ($(MEMORY_TARGET), jtag)
   LDSCRIPT := $(BOARD)/jtag.ld
   VECT_BASE_ADDR := VECT_TAB_BASE
endif

# Useful paths
ifeq ($(LIB_MAPLE_HOME),)
SRCROOT := /home/cospan/Projects/libmaple
else
SRCROOT := $(LIB_MAPLE_HOME)
endif
BUILD_PATH = build
LIBMAPLE_PATH := $(SRCROOT)/libmaple
SUPPORT_PATH := $(SRCROOT)/support
#OPENOCD_WRAPPER := $(SRCROOT)/scripts/openocd-wrapper.sh

OBJS += $(CSRCS:%.c=$(BUILD_PATH)/%.o) 
OBJS += $(CPPSRCS:%.cpp=$(BUILD_PATH)/%.o)




# Compilation flags.
# FIXME remove the ERROR_LED config
GLOBAL_FLAGS    := -D$(VECT_BASE_ADDR)					     \
		   -DBOARD_$(BOARD) -DMCU_$(MCU)			     \
		   -DERROR_LED_PORT=$(ERROR_LED_PORT)			     \
		   -DERROR_LED_PIN=$(ERROR_LED_PIN)			     \
		   -D$(DENSITY) $(CINCS) 
GLOBAL_CFLAGS   := -Os -g3 -gdwarf-2  -mcpu=cortex-m3 -mthumb -march=armv7-m \
		   -nostdlib -ffunction-sections -fdata-sections	     \
		   -Wl,--gc-sections $(GLOBAL_FLAGS) \
		   -D__DEBUG__=1 
GLOBAL_CXXFLAGS := -fno-rtti -fno-exceptions -Wall $(GLOBAL_FLAGS)
GLOBAL_ASFLAGS  := -mcpu=cortex-m3 -march=armv7-m -mthumb		     \
		   -x assembler-with-cpp $(GLOBAL_FLAGS)

LDDIR    := $(SUPPORT_PATH)/ld
LDFLAGS  = -T$(LDDIR)/$(LDSCRIPT) -L$(LDDIR)    \
            -mcpu=cortex-m3 -mthumb -Xlinker     \
            --gc-sections --print-gc-sections --march=armv7-m -Wall \
			-L../../vm -l$(PM_LIB_ROOT) -L.

# Set up build rules and some useful templates
include $(SUPPORT_PATH)/make/build-rules.mk
include $(SUPPORT_PATH)/make/build-templates.mk

# Set all submodules here
LIBMAPLE_MODULES := $(SRCROOT)/libmaple
LIBMAPLE_MODULES += $(SRCROOT)/wirish
# Official libraries:
#LIBMAPLE_MODULES += $(SRCROOT)/libraries/Servo
#LIBMAPLE_MODULES += $(SRCROOT)/libraries/LiquidCrystal
#LIBMAPLE_MODULES += $(SRCROOT)/libraries/Wire

# Experimental libraries:
#LIBMAPLE_MODULES += $(SRCROOT)/libraries/FreeRTOS
LIBMAPLE_MODULES += /home/cospan/Projects/python-on-a-chip/src/platform/maple

# call each module rules.mk
$(foreach m,$(LIBMAPLE_MODULES),$(eval $(call LIBMAPLE_MODULE_template,$(m))))

# Main target
include build-targets.mk

.PHONY: install all clean help debug cscope tags ctags ram flash jtag doxygen mrproper

# Target upload commands
UPLOAD_ram   := $(SUPPORT_PATH)/scripts/reset.py && \
                sleep 1                  && \
                $(DFU) -a0 -d $(VENDOR_ID):$(PRODUCT_ID) -D $(BUILD_PATH)/$(BOARD).bin -R
UPLOAD_flash := $(SUPPORT_PATH)/scripts/reset.py && \
                sleep 1                  && \
                $(DFU) -a1 -d $(VENDOR_ID):$(PRODUCT_ID) -D $(BUILD_PATH)/$(BOARD).bin -R
UPLOAD_jtag  := support/scripts/openocd-wrapper.sh flash

# conditionally upload to whatever the last build was
install: INSTALL_TARGET = $(shell cat $(BUILD_PATH)/build-type 2>/dev/null)
install: $(BUILD_PATH)/$(BOARD).bin
	@echo Install target: $(INSTALL_TARGET)
	$(UPLOAD_$(INSTALL_TARGET))

# Force a rebuild if the maple target changed
PREV_BUILD_TYPE = $(shell cat $(BUILD_PATH)/build-type 2>/dev/null)
build-check:
ifneq ($(PREV_BUILD_TYPE), $(MEMORY_TARGET))
	$(shell rm -rf $(BUILD_PATH))
endif




#pmvm makefile
all: pmfeatures.h $(PM_LIB_PATH) $(TARGET)_nat.c $(TARGET)_img.c build-check MSG_INFO $(BUILD_PATH)/$(BOARD).bin ../../vm/pmstdlib_img.c ../../vm/pmstdlib_nat.c



clean:
	rm -rf build
	make -C ../../vm clean
	rm $(TARGET)_img.* $(TARGET)_nat.* pmfeatures.h 

mrproper: clean
	rm -rf doxygen

help:
	@echo ""
	@echo "  libmaple Makefile help"
	@echo "  ----------------------"
	@echo "  "
	@echo "  Be sure to set BOARD if not compiling for Maple (e.g."
	@echo "  use BOARD=maple_mini for mini, etc.)"
	@echo "  "
	@echo "  Compile targets (default MEMORY_TARGET=flash):"
	@echo "      ram:    Compile sketch code to ram"
	@echo "      flash:  Compile sketch code to flash"
	@echo "      jtag:   Compile sketch code to jtag"
	@echo "      sketch: Compile sketch code to target MEMORY_TARGET"
	@echo "  "
	@echo "  Programming targets:"
	@echo "      install:  Upload code to target"
	@echo "  "
	@echo "  Other targets:"
	@echo "      debug:  Start an openocd gdb server, port 3333"
	@echo "      clean: Remove all build and object files"
	@echo "      help: Show this message"
	@echo "      doxygen: Build Doxygen HTML and XML documentation"
	@echo "      mrproper: Remove all generated files"
	@echo "  "

debug:
	@echo "hi"
#	$(OPENOCD_WRAPPER) debug
	support/scripts/openocd-wrapper.sh debug

cscope:
	rm -rf *.cscope
	find . -name '*.[hcS]' -o -name '*.cpp' | xargs cscope -b

tags:
	etags `find . -name "*.c" -o -name "*.cpp" -o -name "*.h"`
	@echo "Made TAGS file for EMACS code browsing"

ctags:
	ctags-exuberant -R .
	@echo "Made tags file for VIM code browsing"

ram:
	@$(MAKE) MEMORY_TARGET=ram --no-print-directory sketch

flash:
	@$(MAKE) MEMORY_TARGET=flash --no-print-directory sketch

jtag:
	@$(MAKE) MEMORY_TARGET=jtag --no-print-directory sketch

doxygen:
	doxygen $(SUPPORT_PATH)/doxygen/Doxyfile

ALL_CFLAGS := $(GLOBAL_CFLAGS)

export PM_LIB_FN CC OBJCOPY NM ALL_CFLAGS AR LDFLAGS PM_FLAGS