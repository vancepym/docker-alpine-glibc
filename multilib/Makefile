NAME = vancepym/alpine-glibc
VERSION = alpine-3.22-multilib

.PHONY: all build force

all: build

build:
	docker buildx build -t $(NAME):$(VERSION) .

force:
	docker buildx build --force-rm --no-cache -t $(NAME):$(VERSION) .
