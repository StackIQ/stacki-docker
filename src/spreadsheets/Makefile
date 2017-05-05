# @SI_Copyright@
# @SI_Copyright@

ROLLROOT	= ../..

include $(STACKBUILD)/etc/CCRules.mk

build::
	echo "Building spreadsheets"
install::
	mkdir -p $(ROOT)/opt/stack/share/examples/spreadsheets/
	( for x in *.csv; do    \
		$(INSTALL) -m444 $$x $(ROOT)/opt/stack/share/examples/spreadsheets/$(ROLL)-$(ROLLVERSION)-$$x ; \
	done)
