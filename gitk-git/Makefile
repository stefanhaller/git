# The default target of this Makefile is...
all::

uname_S := $(shell sh -c 'uname -s 2>/dev/null || echo not')

prefix ?= $(HOME)
bindir ?= $(prefix)/bin
sharedir ?= $(prefix)/share
gitk_libdir   ?= $(sharedir)/gitk/lib
msgsdir    ?= $(gitk_libdir)/msgs
msgsdir_SQ  = $(subst ','\'',$(msgsdir))

SHELL_PATH ?= /bin/sh
TCL_PATH ?= tclsh
TCLTK_PATH ?= wish
INSTALL ?= install
RM ?= rm -f

DESTDIR_SQ = $(subst ','\'',$(DESTDIR))
bindir_SQ = $(subst ','\'',$(bindir))
libdir_SQ = $(subst ','\'',$(gitk_libdir))
TCLTK_PATH_SQ = $(subst ','\'',$(TCLTK_PATH))

### Detect Tck/Tk interpreter path changes
TRACK_TCLTK = $(subst ','\'',-DTCLTK_PATH='$(TCLTK_PATH_SQ)')

GIT-TCLTK-VARS: FORCE
	@VARS='$(TRACK_TCLTK)'; \
		if test x"$$VARS" != x"`cat $@ 2>/dev/null`" ; then \
			echo 1>&2 "    * new Tcl/Tk interpreter location"; \
			echo "$$VARS" >$@; \
		fi

## po-file creation rules
XGETTEXT   ?= xgettext
ifdef NO_MSGFMT
	MSGFMT ?= $(TCL_PATH) po/po2msg.sh
else
	MSGFMT ?= msgfmt
        ifneq ($(shell $(MSGFMT) --tcl -l C -d . /dev/null 2>/dev/null; echo $$?),0)
		MSGFMT := $(TCL_PATH) po/po2msg.sh
        endif
endif

PO_TEMPLATE = po/gitk.pot
ALL_POFILES = $(wildcard po/*.po)
ALL_MSGFILES = $(subst .po,.msg,$(ALL_POFILES))

ifndef V
	QUIET          = @
	QUIET_GEN      = $(QUIET)echo '   ' GEN $@ &&
endif

all:: gitk-wish $(ALL_MSGFILES)

GITK_MACOSXAPP :=

ifeq ($(uname_S),Darwin)
	GITK_MACOSXAPP := YesPlease
endif

ifdef GITK_MACOSXAPP
all:: Gitk.app

TKFRAMEWORK = /System/Library/Frameworks/Tk.framework/Resources/Wish Shell.app
TKEXECUTABLE = Wish Shell
GIT_VERSION = 1.7.3.2

Gitk.app: macosx/Info.plist macosx/gitk.icns macosx/AppMain.tcl
	$(QUIET_GEN)rm -rf '$@' '$@'+ && \
	mkdir -p '$@'+/Contents/MacOS && \
	mkdir -p '$@'+/Contents/Resources/Scripts && \
	cp '$(subst ','\'',$(subst \,,$(TCLTK_PATH)))' \
		'$@'+/Contents/MacOS/Wish && \
	cp macosx/gitk.icns '$@'+/Contents/Resources && \
	sed -e 's/@@GIT_VERSION@@/$(GIT_VERSION)/g' \
		macosx/Info.plist \
		>'$@'+/Contents/Info.plist && \
	sed -e 's|@@bindir@@|$(bindir_SQ)|' \
		macosx/AppMain.tcl \
		>'$@'+/Contents/Resources/Scripts/AppMain.tcl && \
	mv '$@'+ '$@'
endif

install:: all
	$(INSTALL) -d -m 755 '$(DESTDIR_SQ)$(bindir_SQ)'
	$(INSTALL) -m 755 gitk-wish '$(DESTDIR_SQ)$(bindir_SQ)'/gitk
ifdef GITK_MACOSXAPP
	find 'Gitk.app' | cpio -pud '$(DESTDIR_SQ)$(libdir_SQ)'
endif
	$(INSTALL) -d -m 755 '$(DESTDIR_SQ)$(msgsdir_SQ)'
	$(foreach p,$(ALL_MSGFILES), $(INSTALL) -m 644 $p '$(DESTDIR_SQ)$(msgsdir_SQ)' &&) true

uninstall::
	$(foreach p,$(ALL_MSGFILES), $(RM) '$(DESTDIR_SQ)$(msgsdir_SQ)'/$(notdir $p) &&) true
	$(RM) '$(DESTDIR_SQ)$(bindir_SQ)'/gitk

clean::
	$(RM) gitk-wish po/*.msg GIT-TCLTK-VARS

gitk-wish: gitk GIT-TCLTK-VARS
	$(QUIET_GEN)$(RM) $@ $@+ && \
	$(SHELL_PATH) ./generate-tcl.sh "$(gitk_libdir)/Gitk.app/Contents/MacOS/Wish" "$<" "$@"

$(PO_TEMPLATE): gitk
	$(XGETTEXT) -kmc -LTcl --package-name=Gitk -o $@ gitk
update-po:: $(PO_TEMPLATE)
	$(foreach p, $(ALL_POFILES), echo Updating $p ; msgmerge -U --add-location $p $(PO_TEMPLATE) ; )
	@echo "Before committing changes, ensure that a clean-filter is installed:"; \
	echo; \
	echo "	git config filter.gettext-no-location.clean \"msgcat --no-location -\""
$(ALL_MSGFILES): %.msg : %.po
	@echo Generating catalog $@
	$(MSGFMT) --statistics --tcl -l $(basename $(notdir $<)) -d $(dir $@) $<

.PHONY: all install uninstall clean update-po
.PHONY: FORCE
