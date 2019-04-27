# @SI_Copyright@
# @SI_Copyright@

PKGROOT		= /usr/local/bin
ROLLROOT	= ../..
RURL		= https://github.com/docker/compose/releases/latest
URL		= https://github.com/docker/compose/releases/download
MAKE.rpmflag    = -bb --define "__os_install_post /usr/lib/rpm/brp-compress"

include $(STACKBUILD)/etc/CCRules.mk

VERS=$(shell curl -s -k $(RURL) | awk -F'[/"]' '{print $$9}')

refresh::
	(							\
		echo "NAME=docker-compose" > version.mk;	\
		echo "VERSION=$(VERS)"	>> version.mk;		\
	)

build::
	curl -s -k -L $(URL)/$(VERSION)/docker-compose-$$(uname -s)-$$(uname -m) -o $(NAME)

install::
	mkdir -p $(ROOT)$(PKGROOT)
	$(INSTALL) -m755 $(NAME) $(ROOT)$(PKGROOT)
