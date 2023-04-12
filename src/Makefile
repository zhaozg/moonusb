
ifdef MINGW_PREFIX
MINGW=1
endif

#LINUX=1
MACOS=1

# Lua version
LUAVER?=$(shell lua -e 'print(string.match(_VERSION, "%d+%.%d+") or "5.3")')
ifeq ($(LUAVER),)
# lua-interpreter not found
LUAVER=5.3
endif

# Base install directory
ifdef LINUX
PREFIX?=/usr/local
endif
ifdef MINGW
PREFIX?=$(MINGW_PREFIX)
endif

# Directory where to install Lua modules
L_DIR=$(PREFIX)/share/lua/$(LUAVER)
# Directory where to install Lua C modules
C_DIR=$(PREFIX)/lib/lua/$(LUAVER)
# Directory where to install C headers
H_DIR=$(PREFIX)/include
# Directory where to install C libraries
S_DIR=$(PREFIX)/lib

ifeq ($(D),1)
DEBUG=1
endif

LIBS = -L/usr/local/lib -lluajit-5.1 -lobjc \
       -Wl,-framework,IOKit -Wl,-framework,CoreFoundation -Wl,-framework,Security


LIBUSB_DIR   = libusb/libusb

LIBUSB_SRCS  = $(wildcard $(LIBUSB_DIR)/*.c)
LIBUSB_SRCS += $(LIBUSB_DIR)/os/darwin_usb.c $(LIBUSB_DIR)/os/events_posix.c \
	       $(LIBUSB_DIR)/os/threads_posix.c


Tgt	:= moonusb
Src := $(wildcard *.c)
Objs := $(Src:.c=.o)
LIBUSB_OBJS := $(LIBUSB_SRCS:.c=.o)

INCDIR = -Ilibusb/libusb -I. -I/usr/local/include/lua$(LUAVER)

#COPT	+= -m32
COPT	+= -Wfatal-errors
COPT	+= -Wall -Wextra -Wpedantic
COPT	+= -DCOMPAT53_PREFIX=moonusb_compat_
COPT    += -std=gnu11
COPT 	+= -DLUAVER=$(LUAVER)
ifdef LINUX
COPT    += -fpic
COPT	+= -DLINUX
endif
ifdef MACOS
COPT    += -fpic
COPT	+= -DMACOS
endif
ifdef MINGW
COPT	+= -DMINGW
endif
ifdef DEBUG
COPT	+= -DDEBUG -Og -g
COPT	+= -Wshadow -Wsign-compare -Wundef -Wwrite-strings
COPT	+= -Wdisabled-optimization -Wdeclaration-after-statement
COPT    += -Wmissing-prototypes -Wstrict-prototypes -Wnested-externs
COPT    += -Wold-style-definition
#COPT    += -Wc++-compat
else
COPT	+= -O2
endif

override CFLAGS = $(COPT) $(INCDIR)

default: build

where:
	@echo "PREFIX="$(PREFIX)
	@echo "LUAVER="$(LUAVER)
	@echo $(L_DIR)
	@echo $(C_DIR)
	@echo $(H_DIR)
	@echo $(S_DIR)

clean:
	@-rm -f *.so *.dll *.o *.err *.map *.S *~ *.log
	@-rm -f $(Tgt).symbols

install:
	@-mkdir -pv $(H_DIR)
	@-mkdir -pv $(C_DIR)
	@-mkdir -pv $(S_DIR)
	@-mkdir -pv $(L_DIR)
	@-cp -fpv $(Tgt).h $(H_DIR)
	@-cp -fpvr ../$(Tgt) $(L_DIR)
ifdef LINUX
	@-cp -fpv $(Tgt).so $(C_DIR)
	@-ln -fsv $(C_DIR)/$(Tgt).so $(S_DIR)/lib$(Tgt).so
endif
ifdef MACOS
	@-cp -fpv $(Tgt).so $(C_DIR)
	@-ln -fsv $(C_DIR)/$(Tgt).so $(S_DIR)/lib$(Tgt).so
endif
ifdef MINGW
	@-cp -fpv $(Tgt).dll $(C_DIR)
endif

uninstall:
	@-rm -f $(H_DIR)/$(Tgt).h
	@-rm -f $(C_DIR)/$(Tgt).so
	@-rm -f $(S_DIR)/lib$(Tgt).so
	@-rm -fr $(L_DIR)/$(Tgt)
	@-rm -f $(C_DIR)/$(Tgt).dll

build:	clean $(Tgt)

symbols: build
	@objdump -T $(Tgt).so > $(Tgt).symbols

$(Tgt):	$(Objs) $(LIBUSB_OBJS)
ifdef LINUX
	@$(CXX) -shared -o $(Tgt).so $(Objs) $(LIBUSB_OBJS) $(LIBDIR) $(LIBS)
endif
ifdef MACOS
	@$(CXX) -shared -o $(Tgt).so $(Objs) $(LIBUSB_OBJS) $(LIBDIR) $(LIBS)
endif
ifdef MINGW
	@-$(CXX) -shared -o $(Tgt).dll $(Objs) $(LIBUSB_OBJS) $(LIBDIR) $(LIBS)
endif
	@-rm -f $(Objs) $(LIBUSB_OBJS)
	@echo

