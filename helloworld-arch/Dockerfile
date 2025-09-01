FROM archlinux:latest

RUN pacman -Sy && pacman -S gcc --noconfirm

WORKDIR /app

COPY main.c main.c

RUN gcc main.c -o /app/hello

ENTRYPOINT ["/app/hello"]
