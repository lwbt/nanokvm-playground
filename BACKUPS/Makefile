# cspell:ignore noreport dirsfirst nanokvm Tailscale
BUILD_DATE := $(shell date -I)
BACKUP_FOLDER := backup-$(shell date -I)
IMAGES_FOLDER := images
IMAGES_LIST := $(BACKUP_FOLDER)/images.txt
# Read target hostname from the file 'HOST'. Default to 'nanokvm'
REMOTE_HOST := $(shell cat HOST 2>/dev/null || echo "nanokvm")
#REMOTE_HOST := $(strip $(shell cat HOST 2>/dev/null))
#REMOTE_HOST ?= nanokvm

SHELL := /bin/bash

.PHONY: all
all: create-backup

.PHONY: test
test:
	@echo todo
#	TODO

.PHONY: clean
clean: clean-all

.PHONY: clean-all
clean-all: clean-today
	@tree "$(IMAGES_FOLDER)"
	rm --interactive=once --verbose --recursive --one-file-system "$(IMAGES_FOLDER)"

.PHONY: clean-today
clean-today:
	@tree "$(BACKUP_FOLDER)"
	rm --interactive=once --verbose --recursive --one-file-system "$(BACKUP_FOLDER)"

.PHONY: ssh-auth
ssh-auth: create-ssh-auth

.PHONY: create-ssh-auth
create-ssh-auth:
	@ssh nanokvm 'mkdir -p ~/.ssh'
	@ssh nanokvm 'chmod -v 700 ~/.ssh'
	@ssh nanokvm 'tee > ~/.ssh/authorized_keys' < ~/.ssh/authorized_keys
	@ssh nanokvm 'cat ~/.ssh/authorized_keys'

.PHONY: backup
backup: backup-images

.PHONY: create-backup
create-backup: backup-images

.PHONY: backup-images
backup-images: $(IMAGES_FOLDER) $(BACKUP_FOLDER) $(IMAGES_LIST)
	@rsync -av --progress --copy-links --ignore-existing "$(REMOTE_HOST):/data/" "$(IMAGES_FOLDER)"
#	@rsync -av --progress "$(REMOTE_HOST):/data/" "$(IMAGES_FOLDER)"
	@for line in $(shell cat "$(IMAGES_LIST)"); do \
	  ln -svf "../$${line/\/data/images}" "$(BACKUP_FOLDER)/"; \
	done
	@tree

$(IMAGES_FOLDER):
	@mkdir -v "$(IMAGES_FOLDER)"

$(BACKUP_FOLDER):
	@mkdir -v "$(BACKUP_FOLDER)"

#	TODO: Ping command needs improvement, it actually pings the Tailscale
#	DNS here where I coincidentally picked the same name.
$(IMAGES_LIST):
	@ping -q -c 1 -w 1 $(REMOTE_HOST) && \
	ssh $(REMOTE_HOST) "find /data/ -type f" > "$(IMAGES_LIST)"
#	TODO: Clean up.
#	That's nice, but does not work on busybox and it also does not entirely do what was intended.
#	@tree -f -L 4 -Q -i --noreport --dirsfirst -o "$(IMAGES_LIST)"

.PHONY: restore-backup
restore-backup:
	echo todo
#	TODO
