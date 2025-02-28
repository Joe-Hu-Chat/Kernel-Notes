## make xxx_defconfig

configure the kernel to before compilation. Target `xxx_defconfig` matches the rule below.

```makefile
 601 %config: outputmakefile scripts_basic FORCE
 602         $(Q)$(MAKE) $(build)=scripts/kconfig $@
```

The target has two prerequisites, and a nonexistent file `FORCE` to depend on. So, the `%config` is a phony target this way.

### outputmakefile

```makefile
 532 # Basic helpers built in scripts/basic/
 533 PHONY += scripts_basic
 534 scripts_basic:
 535         $(Q)$(MAKE) $(build)=scripts/basic
 536         $(Q)rm -f .tmp_quiet_recordmcount
 537 
 538 PHONY += outputmakefile
 539 # Before starting out-of-tree build, make sure the source tree is clean.
 540 # outputmakefile generates a Makefile in the output directory, if using a
 541 # separate output directory. This allows convenient use of make in the
 542 # output directory.
 543 # At the same time when output Makefile generated, generate .gitignore to
 544 # ignore whole output directory
 545 outputmakefile:
 546 ifdef building_out_of_srctree
 547         $(Q)if [ -f $(srctree)/.config -o \
 548                  -d $(srctree)/include/config -o \
 549                  -d $(srctree)/arch/$(SRCARCH)/include/generated ]; then \
 550                 echo >&2 "***"; \
 551                 echo >&2 "*** The source tree is not clean, please run 'make$(if $(findstring command line, $(origi     n ARCH)), ARCH=$(ARCH)) mrproper'"; \
 552                 echo >&2 "*** in $(abs_srctree)";\
 553                 echo >&2 "***"; \
 554                 false; \
 555         fi
 556         $(Q)ln -fsn $(srctree) source
 557         $(Q)$(CONFIG_SHELL) $(srctree)/scripts/mkmakefile $(srctree)
 558         $(Q)test -e .gitignore || \
 559         { echo "# this is build directory, ignore it"; echo "*"; } > .gitignore
 560 endif
```

### scripts_basic

```makefile
 532 # Basic helpers built in scripts/basic/
 533 PHONY += scripts_basic
 534 scripts_basic:
 535         $(Q)$(MAKE) $(build)=scripts/basic
 536         $(Q)rm -f .tmp_quiet_recordmcount
```

In `scripts/Kbuild.include` file:

```makefile
154 ###
155 # Shorthand for $(Q)$(MAKE) -f scripts/Makefile.build obj=
156 # Usage:
157 # $(Q)$(MAKE) $(build)=dir
158 build := -f $(srctree)/scripts/Makefile.build obj
```

So, the rule in top `Makefile` expands to:

```makefile
scripts_basic:
		$(Q)$(MAKE) -f $(srctree)/scripts/Makefile.build obj=scripts/basic
        $(Q)rm -f .tmp_quiet_recordmcount
```

In top `Makefile`: 

```makefile
 148 abs_srctree := $(realpath $(dir $(lastword $(MAKEFILE_LIST))))
 ...
 242 ifneq ($(KBUILD_ABS_SRCTREE),)
 243 srctree := $(abs_srctree)
 244 endif
```



## make