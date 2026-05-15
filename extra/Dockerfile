# This Dockerfile creates an image suitable to run LibAFL Fuzzer for
# Xen CI tests. It pulls Ubuntu plus all the required packages.

FROM ubuntu:24.04 AS fuzzer-builder
LABEL "maintainer"="Volodymyr Babchuk <volodymyr_babchuk@epam.com>"


ENV DEBIAN_FRONTEND=noninteractive
RUN apt update \
 && apt upgrade -y \
 && apt install -y \
  gcc-aarch64-linux-gnu \
  binutils \
  bison \
  build-essential \
  clang \
  flex \
  gawk \
  gcc \
  git \
  help2man \
  libglib2.0-dev \
  libncurses5-dev \
  libtool \
  libtool-bin \
  llvm \
  meson \
  ninja-build \
  python3-dev \
  python3-setuptools \
  python3-venv \
  texinfo \
  unzip \
  wget


RUN useradd -ms /bin/bash nonroot
USER nonroot
WORKDIR /home/nonroot

ENV PATH=$PATH:/home/nonroot/.cargo/bin:/home/nonroot/.local/bin
RUN wget -qO- https://sh.rustup.rs | sh -s -- -y -q --no-modify-path && rustup default stable

RUN git clone --branch wip-structured-fuzzing --depth=1 https://github.com/xen-troops/xen-fuzzer-rs.git && \
  git clone --branch RELEASE-4.21.1 --depth=1 https://gitlab.com/xen-project/xen.git && \
  cd xen-fuzzer-rs && cargo install cargo-make && \
  XTF_URL=https://github.com/lorc/xtf.git XTF_REV=structured-fuzzing cargo make build_xtf && \
  XEN_PUBLIC=../xen/xen/include/public/ LIBAFL_QEMU_GIT_URL=https://github.com/xen-troops/qemu.git LIBAFL_QEMU_GIT_REV=libafl-qemu-xencov cargo build --release

FROM ubuntu
LABEL "maintainer"="Volodymyr Babchuk <volodymyr_babchuk@epam.com>"

WORKDIR /root

ENV DEBIAN_FRONTEND=noninteractive
RUN apt update \
 && apt upgrade -y \
 && apt install -y \
  binutils \
  checkpolicy \
  gcc-aarch64-linux-gnu \
  libglib2.0-0 \
  make \
  flex \
  bison

COPY --from=fuzzer-builder /home/nonroot/xen-fuzzer-rs/target/xtf/tests/arm-hypercall-fuzzer/test-mmu64le-arm-hypercall-fuzzer /root

COPY --from=fuzzer-builder /home/nonroot/xen-fuzzer-rs/target/xtf/tests/arm-structured-fuzzer/test-mmu64le-arm-structured-fuzzer /root

COPY --from=fuzzer-builder /home/nonroot/xen-fuzzer-rs/target/xtf/tests/arm-vgic-fuzzer/test-mmu64le-arm-vgic-fuzzer /root

COPY --from=fuzzer-builder /home/nonroot/xen-fuzzer-rs/target/release/xen_fuzzer /root

COPY --from=fuzzer-builder /home/nonroot/xen-fuzzer-rs/corpus /root/corpus

COPY --from=fuzzer-builder /home/nonroot/xen-fuzzer-rs/target/release/qemu-libafl-bridge/pc-bios/efi-virtio.rom /root
