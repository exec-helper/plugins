PROJECT_NAME:=exec-helper-plugins
MAINTAINER?=maintainer
MAINTAINER_EMAIL?=maintainer@exec-helper.xyz

PREFIX?=$(CURDIR)/../../dist
SOURCE_DIR?=$(CURDIR)/../../plugins
GIT_VERSION:=$(shell git -C $(SOURCE_DIR) describe --long "--match=*.*.*" 2>/dev/null || git -C $(SOURCE_DIR) log -n1 --pretty=format:g%h)
SOURCE_VERSION:=$(shell echo $(GIT_VERSION) | sed -E 's@([^-]*)-.*$$@\1@g')
GIT_REVISION:=$(shell echo $(GIT_VERSION) | sed -E 's@[^-]*-([^-])*-.*$$@\1@g')
DEBIAN_REVISION:=$(shell echo $$(($(GIT_REVISION)+1)))
GIT_COMMIT:=$(shell git -C $(SOURCE_DIR) log --format=%h -n1)

YEAR:=$(shell date +'%Y')

VERSION:=$(SOURCE_VERSION)-$(DEBIAN_REVISION)~git$(GIT_COMMIT)
SOURCE_CHANGES_FILE:=$(PROJECT_NAME)_$(VERSION)_source.changes

DSC_FILE:=$(PROJECT_NAME)_$(VERSION).dsc

DISTRIBUTION:=$(shell lsb_release --codename --short)
SYSTEM_DESCRIPTION:=$(shell lsb_release --description --short)

DEBIAN_DIR:=$(CURDIR)/debian
BUILD_DIR:=$(CURDIR)/build

DEBIAN_ARCHIVE=$(BUILD_DIR)/debian.tar.xz
SOURCE_ARCHIVE_NAME:=$(PROJECT_NAME)_$(SOURCE_VERSION).orig.tar.xz
SOURCE_ARCHIVE:=$(BUILD_DIR)/$(SOURCE_ARCHIVE_NAME)
SOURCE_FILES:=bash.lua

PREPARE_MAKEFILE:=prepare.Makefile
SOURCE_MAKEFILE:=source.Makefile
BINARY_MAKEFILE:=binary.Makefile

all: prepare

prepare:
	$(MAKE) --directory dpkg-generator --makefile=$(PREPARE_MAKEFILE) PROJECT_NAME=$(PROJECT_NAME) DEBIAN_DIR=$(DEBIAN_DIR) BUILD_DIR=$(BUILD_DIR)/prepare DEBIAN_ARCHIVE=$(DEBIAN_ARCHIVE) SOURCE_DIR=$(SOURCE_DIR) SOURCE_FILES=$(SOURCE_FILES) VERSION=$(VERSION) DISTRIBUTION=$(DISTRIBUTION) SOURCE_ARCHIVE=$(SOURCE_ARCHIVE) GITCHANGELOG_RC=$(CURDIR)/../../.gitchangelog.rc "SYSTEM_DESCRIPTION=$(SYSTEM_DESCRIPTION)" YEAR=$(YEAR) "MAINTAINER=$(MAINTAINER)" "MAINTAINER_EMAIL=$(MAINTAINER_EMAIL)"

source: prepare
	$(MAKE) --directory dpkg-generator --makefile=$(SOURCE_MAKEFILE) PROJECT_NAME=$(PROJECT_NAME) BUILD_DIR=$(BUILD_DIR)/source PACKAGE_DIR=$(PREFIX)/source DEBIAN_ARCHIVE=$(DEBIAN_ARCHIVE) SOURCE_FILES=$(SOURCE_FILES) CHANGES_FILE=$(SOURCE_CHANGES_FILE) SOURCE_ARCHIVE_NAME=$(SOURCE_ARCHIVE_NAME) SOURCE_ARCHIVE=$(SOURCE_ARCHIVE) DSC_FILE=$(DSC_FILE)

binary: prepare
	$(MAKE) --directory dpkg-generator --makefile=$(BINARY_MAKEFILE) DEBIAN_ARCHIVE=$(DEBIAN_ARCHIVE) SOURCE_ARCHIVE_NAME=$(SOURCE_ARCHIVE_NAME) SOURCE_ARCHIVE=$(SOURCE_ARCHIVE) BUILD_DIR=$(BUILD_DIR)/binary PACKAGE_DIR=$(PREFIX)/binary DSC_FILE=$(DSC_FILE)

clean:
	$(MAKE) --directory dpkg-generator --makefile=$(PREPARE_MAKEFILE) PROJECT_NAME=$(PROJECT_NAME) BUILD_DIR=$(BUILD_DIR)/prepare DEBIAN_ARCHIVE=$(DEBIAN_ARCHIVE) SOURCE_DIR=$(SOURCE_DIR) SOURCE_FILES=$(SOURCE_FILES) "VERSION=$(VERSION)" DISTRIBUTION=$(DISTRIBUTION) SOURCE_ARCHIVE=$(SOURCE_ARCHIVE) clean
	$(MAKE) --directory dpkg-generator --makefile=$(SOURCE_MAKEFILE) PROJECT_NAME=$(PROJECT_NAME) BUILD_DIR=$(BUILD_DIR)/source PACKAGE_DIR=$(PREFIX)/source DEBIAN_ARCHIVE=$(DEBIAN_ARCHIVE) SOURCE_FILES=$(SOURCE_FILES) CHANGES_FILE=$(SOURCE_CHANGES_FILE) SOURCE_ARCHIVE=$(SOURCE_ARCHIVE) clean
	$(MAKE) --directory dpkg-generator --makefile=$(BINARY_MAKEFILE) DEBIAN_ARCHIVE=$(DEBIAN_ARCHIVE) SOURCE_ARCHIVE=$(SOURCE_ARCHIVE) BUILD_DIR=$(BUILD_DIR)/binary PACKAGE_DIR=$(PREFIX)/binary clean
	rm -rf $(BUILD_DIR)
	rm -rf $(PREFIX)

list:
	@$(MAKE) -pRrq -f $(lastword $(MAKEFILE_LIST)) : 2>/dev/null | awk -v RS= -F: '/^# File/,/^# Finished Make data base/ {if ($$1 !~ "^[#.]") {print $$1}}' | sort | egrep -v -e '^[^[:alnum:]]' -e '^$@$$'

.PHONY: prepare source build clean all list
