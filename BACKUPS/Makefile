# cspell:ignore noreport dirsfirst nanokvm Tailscale tailscale
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
	@ssh $(REMOTE_HOST) 'mkdir -p ~/.ssh'
	@ssh $(REMOTE_HOST) 'chmod -v 700 ~/.ssh'
	@ssh $(REMOTE_HOST) 'tee ~/.ssh/authorized_keys' < ~/.ssh/authorized_keys

.PHONY: lock-ssh-auth
lock-ssh-auth:
	@ssh $(REMOTE_HOST) 'mkdir -p /etc/ssh/sshd_config.d'
	@ssh $(REMOTE_HOST) 'tee -a /etc/ssh/sshd_config' <<< "Include /etc/ssh/sshd_config.d/*.conf"
	@ssh $(REMOTE_HOST) 'tee -a /etc/ssh/sshd_config.d/no-password.conf' <<< "PasswordAuthentication no"
	@ssh $(REMOTE_HOST) reboot

.PHONY: lock-ssh-auth-test
lock-ssh-auth-test:
	@ssh -o PubkeyAuthentication=no root@$(REMOTE_HOST) || echo "SSH password login failed."

# TODO: Ensure Tailscale is installed and up to date.
.PHONY: tailscale-login
tailscale-login:
	@ssh $(REMOTE_HOST) "tailscale login"

# TODO: Does not work yet, returns: "sed: unmatched '/'".
# If jq was available it would be: tailscale status -json | jq -r '.Self.DNSName'
# https://github.com/jqlang/jq/releases/download/jq-1.7.1/jq-linux-riscv64
# https://gist.github.com/steinwaywhw/a4cd19cda655b8249d908261a62687f8
.PHONY: tailscale-cert
tailscale-cert:
	@ssh $(REMOTE_HOST) 'tailscale cert $$(tailscale status -json | grep -e 'DNSName' | head -n1 | sed -e 's/ *"DNSName": "\(.*\)\.",$/\1/')'

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
