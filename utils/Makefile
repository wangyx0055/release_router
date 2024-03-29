#
# Miscellaneous utilities Makefile
#
# Copyright (C) 2009, Broadcom Corporation
# All Rights Reserved.
# 
# THIS SOFTWARE IS OFFERED "AS IS", AND BROADCOM GRANTS NO WARRANTIES OF ANY
# KIND, EXPRESS OR IMPLIED, BY STATUTE, COMMUNICATION OR OTHERWISE. BROADCOM
# SPECIFICALLY DISCLAIMS ANY IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS
# FOR A SPECIFIC PURPOSE OR NONINFRINGEMENT CONCERNING THIS SOFTWARE.
#
# $Id: Makefile 361287 2012-10-08 00:28:04Z $
#

include ../common.mak

WLU_OBJS := wlu.o wlu_cmd.o wlu_iov.o wlu_linux.o bcmutils.o bcmwifi_channels.o wlu_client_shared.o wlu_pipe.o wlu_pipe_linux.o miniopt.o bcm_app_utils.o
ifeq ($(RTCONFIG_BCMWL6),y)
WLU_OBJS += wlu_rates_matrix.o wlc_ppr.o
endif
ifneq ($(wildcard $(SRCBASE)/wl/exe/wlu_common.c),)
WLU_OBJS += wlu_common.o
endif
ifeq ($(RTCONFIG_BCMWL6),y)
WLU_OBJS += wlc_clm_data.o
endif

ifeq ($(RWL),1)
WL_OBJS := $(WLU_OBJS)
SERVER_OBJS := wlu_server_linux.o wlu_pipe_linux.o wlu_pipe.o wlu_server_shared.o shellproc_linux.o
APUT_OBJS := wfa_apcmdproc.o wfa_apcmdlib.o wfa_apcs.o wfa_apcmdresp.o wfa_apcmdvalid.o wfa_apcmdtbls.o wfa_apcmdcvt.o wfa_val2str.o wfa_apdiy.o wfa_sock.o wfa_aput_all.o

SOCKET_OBJS := $(SERVER_OBJS:%.o=socket/%.o)
WIFI_OBJS := $(SERVER_OBJS:%.o=wifi/%.o)
WL_OBJS := $(WL_OBJS:%.o=wl_obj/%.o) 
WFA_APUT_OBJS := $(APUT_OBJS:%.o=aput/%.o)

SERVER_SOCKET  := wl_server_socket
SERVER_WIFI  := wl_server_wifi
WFA_APUT_ALL := wfa_aput_all
WL_EXE  := wl
endif

CFLAGS += -s -I$(SRCBASE)/include
ifneq ($(RTCONFIG_BCMWL6),y)
CFLAGS += -I$(TOP)/shared -I$(TOP)/nvram
endif
CFLAGS += $(if $(WLAN_ComponentIncPath),$(WLAN_ComponentIncPath),$(addprefix -I,$(wildcard $(SRCBASE)/shared/bcmwifi/src)))
#ifeq ($(CONFIG_RTR_OPTIMIZE_SIZE),y)
ifeq ($(SFP),y)
CFLAGS  += -Os
CFLAGS  += -ffunction-sections -fdata-sections
LDFLAGS += -ffunction-sections -fdata-sections -Wl,--gc-sections
else
CFLAGS  += -O2
endif
CFLAGS += -DBCMINTERNAL -DBCMDBG
ifneq ($(RTCONFIG_BCMWL6),y)
LDFLAGS += -L$(TOP)/nvram -L$(INSTALLDIR)/nvram/usr/lib -lnvram
LDFLAGS += -L$(TOP)/shared -L$(INSTALLDIR)/shared/usr/lib -lshared
endif

#UTILS := mii-tool robocfg ttcp
#ifneq ($(CONFIG_LINUX26),y)
#CFLAGS += -U CONFIG_NVRAM_SIZE
#endif

CFLAGS += -DLINUX -I$(SRCBASE)/wl/exe
ifeq ($(RTCONFIG_BCMWL6), y)
CFLAGS += -I$(SRCBASE)/wl/olpc/include -I$(SRCBASE)/wl/ppr/include
endif

# Use newer 11ac ratespec for wl command line
CFLAGS += -DD11AC_IOTYPES -DPPR_API

#ifdef WLCNT
CFLAGS += -DWLCNT
#endif

ifeq ($(CONFIG_TRAFFIC_MGMT),y)
CFLAGS += -DTRAFFIC_MGMT
endif

vpath %.c $(SRCBASE)/tools/misc
ifeq ($(RTCONFIG_BCMWL6),y)
vpath %.c $(SRCBASE)/shared/bcmwifi/src $(SRCBASE)/wl/ppr/src
endif

ifneq ($(wildcard $(SRCBASE)/et/linux/et.c),)
UTILS += et
vpath %.c $(SRCBASE)/et/linux
endif

ifneq ($(wildcard $(SRCBASE)/wl/exe),)
CFLAGS += -I$(SRCBASE)/wl/exe
UTILS += wl
vpath %.c $(SRCBASE)/wl/exe $(SRCBASE)/wl/clm/src $(SRCBASE)/shared
endif

ifeq ($(RWL),1)
ifeq ($(BUILD_WFA_APUT),1)
ifneq ($(wildcard $(SRCBASE)/router/aput),)
CFLAGS += -I$(SRCBASE)/router/aput
UTILS += $(WFA_APUT_ALL)
vpath %.c $(SRCBASE)/router/aput
endif
endif

ifneq ($(wildcard $(SRCBASE)/wl/exe),)
UTILS += $(SERVER_SOCKET)
vpath %.c $(SRCBASE)/wl/exe $(SRCBASE)/shared
endif

ifneq ($(wildcard $(SRCBASE)/wl/exe),)
UTILS += $(SERVER_WIFI)
vpath %.c $(SRCBASE)/wl/exe $(SRCBASE)/shared
endif
endif

ifeq ($(RTCONFIG_5301X),y)
CFLAGS += -DBCM5301X
endif
UTILS += robocfg

all: $(UTILS)

ifeq ($(wildcard $(SRCBASE)/wl/exe/*.c),)
wl:
	-cp -f $(SRCBASE)/wl/exe/prebuilt/wl wl
else
wl: $(WLU_OBJS)
	$(CC) $^ -o $@ $(LDFLAGS)
endif

ifeq ($(RWL),1)
$(WFA_APUT_ALL): $(WFA_APUT_OBJS)
	$(CC) $(LDFLAGS) -o $@ $^ 

aput/%.o: %.c
	@mkdir -p aput
	$(CC) -c $(CFLAGS) -o $@ $^
$(SERVER_SOCKET): $(SOCKET_OBJS)
	$(CC) $(LDFLAGS) -o $@ $^ 

socket/%.o: %.c
	@mkdir -p socket
	$(CC) -c $(CFLAGS) -DRWL_SOCKET -o $@ $^

$(SERVER_WIFI): $(WIFI_OBJS)
	$(CC) $(LDFLAGS) -o $@ $^

wifi/%.o: %.c
	@mkdir -p wifi
	$(CC) -c $(CFLAGS) -DRWL_WIFI -o $@ $^
endif

mii-tool: mii-tool.c mii.h

ifeq ($(wildcard $(SRCBASE)/router/utils/robocfg.c),)
robocfg:
	-cp -f $(SRCBASE)/wl/exe/prebuilt/robocfg robocfg
else
robocfg: robocfg.c etc53xx.h
endif

ttcp: epi_ttcp.c
	$(CC) $(CFLAGS) -o $@ $^

clean:
	rm -f *.o *~ $(UTILS)
	rm -rf socket wifi epi_ttcp

install: all
	install -d $(INSTALLDIR)/usr/sbin
	install $(UTILS) $(INSTALLDIR)/usr/sbin
	$(STRIP) $(foreach file,$(UTILS),$(INSTALLDIR)/usr/sbin/$(file))
