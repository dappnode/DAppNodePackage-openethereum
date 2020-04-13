# Build Parity in a stock alpine builder container
FROM alpine:edge as builder

ARG OPENETHEREUM_VERSION=v2.7.2

# show backtraces
ENV RUST_BACKTRACE 1

RUN apk --no-cache add \
    build-base \
    cargo \
    cmake \
    eudev-dev \
    linux-headers \
    perl \
    rust \ 
    git \ 
    clang-dev \
    llvm-dev

RUN git clone -b ${OPENETHEREUM_VERSION} https://github.com/OpenEthereum/open-ethereum /open-ethereum
WORKDIR /open-ethereum
ENV C=clang CXX=clang++ RUSTFLAGS="-C link-arg=-s"

RUN cargo build --release --features final --target x86_64-alpine-linux-musl
RUN strip target/x86_64-alpine-linux-musl/release/parity

RUN cp /open-ethereum/target/x86_64-alpine-linux-musl/release/parity  /open-ethereum/target/x86_64-alpine-linux-musl/release/openethereum

# Pull open-ethereum into a second stage deploy alpine container
FROM alpine:edge

RUN apk --no-cache add \
    libstdc++ \ 
    eudev-libs \
    libgcc \
    bash


COPY --from=builder /open-ethereum/target/x86_64-alpine-linux-musl/release/openethereum /usr/local/bin
    
ENTRYPOINT [ "sh", "-c", "exec openethereum --jsonrpc-port 8545 --jsonrpc-interface all --jsonrpc-hosts all --jsonrpc-cors all --ws-interface 0.0.0.0 --ws-port 8546 --ws-origins all --ws-hosts all --ws-max-connections 1000 ${EXTRA_OPTS}"]
