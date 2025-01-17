-include $(TOPDIR)/.config
-include $(TOPDIR)/Make.defs
include $(APPDIR)/Make.defs

UROS_DIR = $(APPDIR)/$(CONFIG_UROS_DIR)

ifeq ($(CONFIG_WINDOWS_NATIVE),y)
  BIN = ..\libapps$(LIBEXT)
else
ifeq ($(WINTOOL),y)
  BIN = ..\\libapps$(LIBEXT)
else
  BIN = ../libapps$(LIBEXT)
endif
endif

# Build type flag
ifeq ($(CONFIG_DEBUG_SYMBOLS),y)
	BUILD_TYPE = Debug
else
	BUILD_TYPE = RelWithDebInfo
endif

all: colcon_compile $(BIN)
  .PHONY: clean depend distclean preconfig
  .PRECIOUS: ../../libapps$(LIBEXT)


%.stamp_archive: %.a
	#-$(Q) mkdir -p extract/$(*F)
	-$(Q) mkdir -p extract/temp
	cd extract/temp && $(ARCROSSDEV)ar -x $<
	cd extract/temp; for file in *; do mv $$file ../$(*F)_$$file; done
	touch $@ -r $<

%.stamp_update: %.stamp_archive
	$(ARCROSSDEV)ar r $(BIN) extract/$(*F)_*
	touch $@

ARCHIVES = $(shell find $(UROS_DIR)/install -name '*.a')

$(BIN): $(ARCHIVES:.a=.stamp_update)

arm_toolchain.cmake: $(TOPDIR)/.config arm_toolchain.cmake.in
	cat arm_toolchain.cmake.in | \
		sed "s/@CROSSDEV@/$(CROSSDEV)/g" | \
		sed "s/@NUTTX_TOPDIR@/$(subst /,\/,$(TOPDIR))/g" |\
		sed "s/@ARCH_CPU_FLAGS@/\"$(ARCHCPUFLAGS)\"/g" |\
		sed "s/@ARCH_OPT_FLAGS@/\"$(ARCHOPTIMIZATION)\"/g" \
		> arm_toolchain.cmake

colcon_compile: arm_toolchain.cmake
	$(Q) cd $(UROS_DIR); \
		colcon build \
			--packages-ignore-regex=.*_cpp \
			--cmake-args \
			-DBUILD_SHARED_LIBS=OFF \
			-DCMAKE_POSITION_INDEPENDENT_CODE=ON \
			-DBUILD_TESTING=OFF \
			-DCMAKE_BUILD_TYPE=$(BUILD_TYPE) \
			-DCMAKE_TOOLCHAIN_FILE=$(APPDIR)/uros/arm_toolchain.cmake \
			-DCMAKE_VERBOSE_MAKEFILE=ON \

install: 

context: arm_toolchain.cmake

.depend:

depend: .depend

clean:
	cd $(UROS_DIR); colcon build --cmake-target clean --cmake-target-skip-unavailable
	rm -f $(ARCHIVES:.a=.stamp_archive)
	rm -f $(ARCHIVES:.a=.stamp_update)
	rm -rf extract

distclean:
	$(call DELDIR, $(UROS_DIR)/build)
	$(call DELDIR, $(UROS_DIR)/install)
	$(call DELDIR, $(UROS_DIR)/log)
	$(call DELFILE, $(UROS_DIR)/.built)


preconfig:

-include Make.dep
