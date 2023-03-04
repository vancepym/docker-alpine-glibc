NAME = vancepym/alpine-glibc
VERSION = alpine-3.17-multilib

.PHONY: all build force

all: build

build:
	docker build -t $(NAME):$(VERSION) .

force:
	docker build --force-rm --no-cache -t $(NAME):$(VERSION) .
