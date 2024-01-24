# Builds either a recovery image (Sonoma-recovery.img) or a full installer (Sonoma-full.img) for Sonoma.

# To build the full installer you must run this on macOS.
# The recovery can be built on either macOS or Linux.

# For Ubuntu (or similar Linux distribution) you'll need to run this first to get the required packages:
# sudo apt install qemu-utils make

# For macOS you'll probably need to run xcode-select --install to get the commandline tools

SONOMA_APP=/Applications/Install\ macOS\ Sonoma.app

LINUX_TOOLS = qemu-img

OS :=
UNAME_S := $(shell uname -s)

ifeq ($(UNAME_S),Darwin)
	OS = MACOS
endif

# If this is Linux make sure we have all our build tools available:
ifeq ($(OS),)
	K := $(foreach exec,$(LINUX_TOOLS),\
			$(if $(shell which $(exec)),some string,$(error "Missing required $(exec) tool for build")))
endif

all: Sonoma-recovery.img

%.img : %.dmg
	ln $< $@ || cp $< $@

ifeq ($(OS),MACOS)

Sonoma-full.dmg : $(SONOMA_APP)
	hdiutil create -o "$@" -size 15g -layout GPTSPUD -fs HFS+J
	hdiutil attach -noverify -mountpoint /Volumes/install_build "$@"
	sudo "$</Contents/Resources/createinstallmedia" --volume /Volumes/install_build --nointeraction
	hdiutil detach "/Volumes/Install macOS Sonoma"

# Avoid redownloading Sonoma if the app already exists
ifeq (,$(wildcard $(SONOMA_APP)))
$(SONOMA_APP) : InstallAssistant.pkg
	sudo installer -pkg $< -target /
endif

else

Sonoma-full.dmg :
	$(error "Building a full installer requires this script to be run on macOS, run 'make Sonoma-recovery.img' instead")

endif

Sonoma-recovery.dmg : com.apple.recovery.boot/BaseSystem.dmg
	rm -f $@
ifeq ($(OS),MACOS)
	hdiutil convert $< -format UDRW -o $@
else
	qemu-img convert $< -O raw $@
endif

com.apple.recovery.boot/BaseSystem.dmg :
	../../fetch-macOS-v2.py --action download --board-id Mac-4B682C642B45593E --os latest

InstallAssistant.pkg :
	../../backups/fetch-macOS.py --version latest --title "macOS Sonoma"

clean :
	rm -f SharedSupport.dmg InstallAssistant.pkg Sonoma-recovery.img Sonoma-full.img Sonoma-recovery.dmg Ventura-full.dmg
	rm -rf content com.apple.recovery.boot
