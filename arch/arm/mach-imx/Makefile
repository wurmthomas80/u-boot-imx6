# SPDX-License-Identifier: GPL-2.0+
#
# (C) Copyright 2000-2006
# Wolfgang Denk, DENX Software Engineering, wd@denx.de.
#
# (C) Copyright 2011 Freescale Semiconductor, Inc.

ifeq ($(SOC),$(filter $(SOC),mx25 mx35 mx5 mx6 mx7 mx8m vf610))
obj-y	= iomux-v3.o
endif

ifeq ($(SOC),$(filter $(SOC),mx8m))
ifneq ($(CONFIG_SPL_BUILD),y)
obj-$(CONFIG_IMX_BOOTAUX) += imx_bootaux.o
endif
obj-$(CONFIG_ENV_IS_IN_MMC) += mmc_env.o
obj-$(CONFIG_FEC_MAC_FUSE) += mac.o
obj-$(CONFIG_SYS_I2C_MXC) += i2c-mxv7.o
obj-$(CONFIG_IMX_VIDEO_SKIP) += video.o
obj-y += cpu.o
endif

ifeq ($(SOC),$(filter $(SOC),mx5 mx6))
obj-y	+= cpu.o speed.o
obj-$(CONFIG_GPT_TIMER) += timer.o
obj-$(CONFIG_SYS_I2C_MXC) += i2c-mxv7.o
endif
ifeq ($(SOC),$(filter $(SOC),mx7 mx6 mxs mx8m))
obj-y	+= misc.o
obj-$(CONFIG_SPL_BUILD)	+= spl.o
endif
ifeq ($(SOC),$(filter $(SOC),mx7))
obj-y 	+= cpu.o
obj-$(CONFIG_SYS_I2C_MXC) += i2c-mxv7.o
obj-$(CONFIG_ENV_IS_IN_MMC) += mmc_env.o
endif

ifeq ($(SOC),$(filter $(SOC),mx5 mx6 mx7 mx8m))
ifeq ($(CONFIG_SPL_BUILD)$(CONFIG_TPL_BUILD),)
obj-$(CONFIG_CMD_FBPANEL) += fbpanel.o
endif
endif

ifeq ($(SOC),$(filter $(SOC),mx6 mx7))
obj-y 	+= cache.o init.o
obj-$(CONFIG_FEC_MAC_FUSE) += mac.o
obj-$(CONFIG_IMX_VIDEO_SKIP) += video.o
obj-$(CONFIG_IMX_RDC) += rdc-sema.o
ifneq ($(CONFIG_SPL_BUILD),y)
obj-$(CONFIG_IMX_BOOTAUX) += imx_bootaux.o
endif
obj-$(CONFIG_SATA) += sata.o
obj-$(CONFIG_SECURE_BOOT)    += hab.o
obj-$(CONFIG_SYSCOUNTER_TIMER) += syscounter.o
endif
ifeq ($(SOC),$(filter $(SOC),mx7ulp))
obj-y  += cache.o
obj-$(CONFIG_SECURE_BOOT) += hab.o
endif
ifeq ($(SOC),$(filter $(SOC),vf610))
obj-y += ddrmc-vf610.o
endif
ifneq ($(CONFIG_SPL_BUILD),y)
obj-$(CONFIG_CMD_BMODE) += cmd_bmode.o
obj-$(CONFIG_CMD_HDMIDETECT) += cmd_hdmidet.o
obj-$(CONFIG_CMD_DEKBLOB) += cmd_dek.o
endif

PLUGIN = board/$(BOARDDIR)/plugin

ifeq ($(CONFIG_USE_IMXIMG_PLUGIN),y)

$(PLUGIN).o: $(PLUGIN).S FORCE
	$(Q)mkdir -p $(dir $@)
	$(call if_changed_dep,as_o_S)

$(PLUGIN).bin: $(PLUGIN).o FORCE
	$(Q)mkdir -p $(dir $@)
	$(OBJCOPY) -O binary --gap-fill 0xff $< $@
else

$(PLUGIN).bin:

endif

quiet_cmd_cpp_cfg = CFGS    $@
      cmd_cpp_cfg = $(CPP) $(cpp_flags) -x c -o $@ $<

# mkimage source config file
IMX_CONFIG = $(CONFIG_IMX_CONFIG:"%"=%)

# How to create a cpp processed config file, they all use the same source
%.cfgout: $(IMX_CONFIG) FORCE
	$(Q)mkdir -p $(dir $@)
	$(call if_changed_dep,cpp_cfg)

MKIMAGEFLAGS_u-boot.imx = -n $(filter-out $(PLUGIN).bin $< $(PHONY),$^) -T imximage \
	-e $(CONFIG_SYS_TEXT_BASE)
u-boot.imx: MKIMAGEOUTPUT = u-boot.imx.log

u-boot.imx: u-boot.bin u-boot.cfgout $(PLUGIN).bin FORCE
	$(call if_changed,mkimage)

ifeq ($(CONFIG_OF_SEPARATE),y)
MKIMAGEFLAGS_u-boot-dtb.imx = -n $(filter-out $(PLUGIN).bin $< $(PHONY),$^) -T imximage \
	-e $(CONFIG_SYS_TEXT_BASE)
u-boot-dtb.imx: MKIMAGEOUTPUT = u-boot-dtb.imx.log

u-boot-dtb.imx: u-boot-dtb.bin u-boot-dtb.cfgout $(PLUGIN).bin FORCE
	$(call if_changed,mkimage)
endif

MKIMAGEFLAGS_SPL = -n $(filter-out $(PLUGIN).bin $< $(PHONY),$^) -T imximage \
	-e $(CONFIG_SPL_TEXT_BASE)
SPL: MKIMAGEOUTPUT = SPL.log

SPL: spl/u-boot-spl.bin spl/u-boot-spl.cfgout $(PLUGIN).bin FORCE
	$(call if_changed,mkimage)

MKIMAGEFLAGS_u-boot.uim = -A arm -O U-Boot -a $(CONFIG_SYS_TEXT_BASE) \
		-e $(CONFIG_SYS_TEXT_BASE) -C none -T firmware

u-boot.uim: u-boot.bin FORCE
	$(call if_changed,mkimage)

OBJCOPYFLAGS += -I binary -O binary --pad-to=$(CONFIG_SPL_PAD_TO)
append = cat $(filter-out $< $(PHONY), $^) >> $@

quiet_cmd_pad_cat = CAT     $@
cmd_pad_cat = $(cmd_objcopy) && $(append) || rm -f $@

u-boot-with-spl.imx: SPL u-boot.uim FORCE
	$(call if_changed,pad_cat)

u-boot-with-nand-spl.imx: spl/u-boot-nand-spl.imx u-boot.uim FORCE
	$(call if_changed,pad_cat)

quiet_cmd_u-boot-nand-spl_imx = GEN     $@
cmd_u-boot-nand-spl_imx = (printf '\000\000\000\000\106\103\102\040\001' && \
	dd bs=1015 count=1 if=/dev/zero 2>/dev/null) | cat - $< > $@

spl/u-boot-nand-spl.imx: SPL FORCE
	$(call if_changed,u-boot-nand-spl_imx)

targets += $(addprefix ../../../,SPL spl/u-boot-spl.cfgout u-boot-dtb.cfgout u-boot.cfgout u-boot.uim spl/u-boot-nand-spl.imx)

obj-$(CONFIG_ARM64) += sip.o

obj-$(CONFIG_MX5) += mx5/
obj-$(CONFIG_MX6) += mx6/
obj-$(CONFIG_MX7) += mx7/
obj-$(CONFIG_ARCH_MX7ULP) += mx7ulp/
obj-$(CONFIG_MX8M) += mx8m/
