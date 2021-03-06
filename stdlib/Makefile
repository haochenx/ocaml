#**************************************************************************
#*                                                                        *
#*                                 OCaml                                  *
#*                                                                        *
#*            Xavier Leroy, projet Cristal, INRIA Rocquencourt            *
#*                                                                        *
#*   Copyright 1999 Institut National de Recherche en Informatique et     *
#*     en Automatique.                                                    *
#*                                                                        *
#*   All rights reserved.  This file is distributed under the terms of    *
#*   the GNU Lesser General Public License version 2.1, with the          *
#*   special exception on linking described in the file LICENSE.          *
#*                                                                        *
#**************************************************************************

ROOTDIR = ..

include $(ROOTDIR)/Makefile.common

TARGET_BINDIR ?= $(BINDIR)

COMPILER=$(ROOTDIR)/ocamlc$(EXE)
CAMLC=$(CAMLRUN) $(COMPILER)
COMPFLAGS=-strict-sequence -absname -w +a-4-9-41-42-44-45-48 \
          -g -warn-error A -bin-annot -nostdlib -principal \
          -safe-string -strict-formats
OPTCOMPILER=$(ROOTDIR)/ocamlopt$(EXE)
CAMLOPT=$(CAMLRUN) $(OPTCOMPILER)
CAMLDEP=$(BOOT_OCAMLC) -depend
DEPFLAGS=-slash

OC_CPPFLAGS += -I$(ROOTDIR)/runtime

include StdlibModules

OBJS=$(addsuffix .cmo,$(STDLIB_MODULES))
NOSTDLIB= camlinternalFormatBasics.cmo camlinternalAtomic.cmo stdlib.cmo
OTHERS=$(filter-out $(NOSTDLIB),$(OBJS))

PREFIXED_OBJS=$(filter stdlib__%.cmo,$(OBJS))
UNPREFIXED_OBJS=$(PREFIXED_OBJS:stdlib__%.cmo=%)

.PHONY: all
all: stdlib.cma std_exit.cmo camlheader target_camlheader camlheader_ur

ifeq "$(RUNTIMED)" "true"
all: camlheaderd target_camlheaderd
endif

ifeq "$(RUNTIMEI)" "true"
all: camlheaderi target_camlheaderi
endif

.PHONY: allopt opt.opt # allopt and opt.opt are synonyms
allopt: stdlib.cmxa std_exit.cmx
opt.opt: allopt

.PHONY: install
install::
	$(INSTALL_DATA) \
	  stdlib.cma std_exit.cmo *.cmi camlheader_ur \
	  "$(INSTALL_LIBDIR)"
ifeq "$(INSTALL_SOURCE_ARTIFACTS)" "true"
	$(INSTALL_DATA) \
	  *.cmt *.cmti *.mli *.ml \
	  "$(INSTALL_LIBDIR)"
endif
	$(INSTALL_DATA) target_camlheader "$(INSTALL_LIBDIR)/camlheader"

ifeq "$(RUNTIMED)" "true"
install::
	$(INSTALL_DATA) target_camlheaderd "$(INSTALL_LIBDIR)/camlheaderd"
endif

ifeq "$(RUNTIMEI)" "true"
install::
	$(INSTALL_DATA) target_camlheaderi "$(INSTALL_LIBDIR)/camlheaderi"
endif

.PHONY: installopt
installopt: installopt-default

.PHONY: installopt-default
installopt-default:
	$(INSTALL_DATA) \
	  stdlib.cmxa stdlib.$(A) std_exit.$(O) *.cmx \
	  "$(INSTALL_LIBDIR)"
	cd "$(INSTALL_LIBDIR)"; $(RANLIB) stdlib.$(A)

ifeq "$(UNIX_OR_WIN32)" "unix"
HEADERPROGRAM = header
HEADER_PATH = $(BINDIR)/
HEADER_TARGET_PATH = $(TARGET_BINDIR)/
else # Windows
HEADERPROGRAM = headernt
HEADER_PATH =
HEADER_TARGET_PATH =
endif

TARGETHEADERPROGRAM = target_$(HEADERPROGRAM)

# The shebang test in configure.ac will need updating if any runtime is
# introduced with a suffix more than one character long (camlheader_ur doesn't
# matter).
CAMLHEADERS =\
  camlheader target_camlheader camlheader_ur \
  camlheaderd target_camlheaderd \
  camlheaderi target_camlheaderi

# The % in pattern rules must always match something, hence the slightly strange
# patterns and $(subst ...) since `camlheader%:` wouldn't match `camlheader`
ifeq "$(SHEBANGSCRIPTS)" "true"
camlhead%: $(ROOTDIR)/Makefile.config Makefile
ifeq "$(LONG_SHEBANG)" "true"
	echo '#!/bin/sh' > $@
	echo 'exec "$(BINDIR)/ocamlrun$(subst er,,$*)" "$$0" "$$@"' >> $@
else
	echo '#!$(BINDIR)/ocamlrun$(subst er,,$*)' > $@
endif

# TODO This does not take long shebangs into account (since TARGET_BINDIR is not
#      yet processed by configure)
target_%: $(ROOTDIR)/Makefile.config Makefile
	echo '#!$(TARGET_BINDIR)/ocamlrun$(subst camlheader,,$*)' > $@

camlheader_ur: Makefile
	echo '#!' | tr -d '\012' > $@

else # Hashbang scripts not supported

$(CAMLHEADERS): $(HEADERPROGRAM).c $(ROOTDIR)/Makefile.config Makefile

# $@.exe is deleted to ensure no Cygwin .exe mangling takes place
camlhead%: tmphead%.exe
	rm -f $@.exe
	mv $< $@

# Again, pattern weirdness here means that the dot is always present so that
# tmpheader.exe matches.
tmpheader%exe: $(HEADERPROGRAM)%$(O)
	$(call MKEXE_BOOT,$@,$^ $(EXTRALIBS))
# FIXME This is wrong - mingw could invoke strip; MSVC equivalent?
ifneq "$(UNIX_OR_WIN32)" "win32"
	strip $@
endif

$(HEADERPROGRAM)%$(O): \
  OC_CPPFLAGS += -DRUNTIME_NAME='"$(HEADER_PATH)ocamlrun$(subst .,,$*)"'

$(HEADERPROGRAM)%$(O): $(HEADERPROGRAM).c
	$(CC) -c $(OC_CFLAGS) $(CFLAGS) $(OC_CPPFLAGS) $(CPPFLAGS) \
	  $(OUTPUTOBJ)$@ $^

camlheader_ur: camlheader
	cp camlheader $@

ifeq "$(UNIX_OR_WIN32)" "unix"
tmptargetcamlheader%exe: $(TARGETHEADERPROGRAM)%$(O)
	$(call MKEXE_BOOT,$@,$^ $(EXTRALIBS))
	strip $@

$(TARGETHEADERPROGRAM)%$(O): $(HEADERPROGRAM).c
	$(CC) -c $(OC_CFLAGS) $(CFLAGS) $(OC_CPPFLAGS) $(CPPFLAGS) \
	      -DRUNTIME_NAME='"$(HEADER_TARGET_PATH)ocamlrun$(subst .,,$*)"' \
	      $(OUTPUTOBJ)$@ $^

target_%: tmptarget%.exe
	rm -f $@.exe
	mv $< $@
else
target_%: %
	cp $< $@
endif

endif # ifeq "$(SHEBANGSCRIPTS)" "true"

stdlib.cma: $(OBJS)
	$(CAMLC) -a -o $@ $^

stdlib.cmxa: $(OBJS:.cmo=.cmx)
	$(CAMLOPT) -a -o $@ $^

sys.ml: $(ROOTDIR)/VERSION sys.mlp
	sed -e "s|%%VERSION%%|`sed -e 1q $< | tr -d '\r'`|" sys.mlp > $@

.PHONY: clean
clean::
	rm -f sys.ml

clean::
	rm -f $(CAMLHEADERS)

.SUFFIXES: .mli .ml .cmi .cmo .cmx

export AWK

%.cmi: %.mli
	$(CAMLC) $(COMPFLAGS) $(shell ./Compflags $@) -c $<

stdlib__%.cmi: %.mli
	$(CAMLC) $(COMPFLAGS) $(shell ./Compflags $@) -o $@ -c $<

%.cmo: %.ml
	$(CAMLC) $(COMPFLAGS) $(shell ./Compflags $@) -c $<

stdlib__%.cmo: %.ml
	$(CAMLC) $(COMPFLAGS) $(shell ./Compflags $@) -o $@ -c $<

%.cmx: %.ml
	$(CAMLOPT) $(COMPFLAGS) $(OPTCOMPFLAGS) $(shell ./Compflags $@) -c $<

stdlib__%.cmx: %.ml
	$(CAMLOPT) $(COMPFLAGS) $(OPTCOMPFLAGS) $(shell ./Compflags $@) \
	           -o $@ -c $<

# Dependencies on the compiler
COMPILER_DEPS=$(filter-out -use-prims $(CAMLRUN), $(CAMLC))
$(OBJS) std_exit.cmo: $(COMPILER_DEPS)
$(OBJS:.cmo=.cmi) std_exit.cmi: $(COMPILER_DEPS)
$(OBJS:.cmo=.cmx) std_exit.cmx: $(OPTCOMPILER)

# Dependencies on Stdlib (not tracked by ocamlc -depend)

$(OTHERS) std_exit.cmo: stdlib.cmi
$(OTHERS:.cmo=.cmi) std_exit.cmi: stdlib.cmi
$(OBJS:.cmo=.cmx) std_exit.cmx: stdlib.cmi
$(OTHERS:.cmo=.cmx) std_exit.cmx: stdlib.cmx

clean::
	rm -f *.cm* *.o *.obj *.a *.lib *.odoc

include .depend

EMPTY :=
SPACE := $(EMPTY) $(EMPTY)

.PHONY: depend
depend:
	$(CAMLDEP) $(DEPFLAGS) $(filter-out stdlib.%,$(wildcard *.mli *.ml)) \
	  > .depend.tmp
	$(CAMLDEP) $(DEPFLAGS) -pp "$(AWK) -f ./remove_module_aliases.awk" \
	  stdlib.ml stdlib.mli >> .depend.tmp
	sed -Ee \
	  's#(^| )(${subst ${SPACE},|,${UNPREFIXED_OBJS}})[.]#\1stdlib__\2.#g' \
	  .depend.tmp > .depend
	rm -f .depend.tmp
