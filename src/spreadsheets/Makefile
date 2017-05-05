# @SI_Copyright@
# @SI_Copyright@

ROLLROOT	= ../..

include $(STACKBUILD)/etc/CCRules.mk

build::
	echo "Building spreadsheets"
install::
	mkdir -p $(ROOT)/opt/stack/share/examples/spreadsheets/docker
	( for x in *.csv; do    \
		$(INSTALL) -m444 $$x $(ROOT)/opt/stack/share/examples/spreadsheets/docker/$(ROLL)-$(ROLLVERSION)-$$x ; \
	done)
