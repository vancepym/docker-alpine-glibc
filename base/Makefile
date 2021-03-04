NAME = vancepym/alpine-glibc
VERSION = alpine-3.13

.PHONY: all build force

all: build

build:
	docker build -t $(NAME):$(VERSION) .

force:
	docker build --force-rm --no-cache -t $(NAME):$(VERSION) .
