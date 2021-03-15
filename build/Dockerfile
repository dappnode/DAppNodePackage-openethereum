FROM --platform=amd64 ubuntu:xenial as builder
WORKDIR /build

ARG UPSTREAM_VERSION
ARG TARGETARCH

# install aarch64(armv8) dependencies and tools
RUN dpkg --add-architecture arm64
RUN echo '# source urls for arm64 \n\
	deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ xenial main \n\
	deb-src [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ xenial main \n\
	deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main \n\
	deb-src [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main \n\
	deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ xenial-security main \n\
	deb-src [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ xenial-security main \n # end arm64 section' >> /etc/apt/sources.list &&\
	sed -r 's/deb h/deb \[arch=amd64\] h/g' /etc/apt/sources.list > /tmp/sources-tmp.list && \
	cp /tmp/sources-tmp.list /etc/apt/sources.list&& \
	sed -r 's/deb-src h/deb-src \[arch=amd64\] h/g' /etc/apt/sources.list > /tmp/sources-tmp.list&&cat /etc/apt/sources.list &&\
	cp /tmp/sources-tmp.list /etc/apt/sources.list&& echo "next"&&cat /etc/apt/sources.list

# install tools and dependencies
RUN apt-get -y update && \
	apt-get upgrade -y && \
	apt-get install -y --no-install-recommends \
		curl make cmake file ca-certificates  \
		g++ gcc-aarch64-linux-gnu g++-aarch64-linux-gnu \
		libc6-dev-arm64-cross binutils-aarch64-linux-gnu \
		libudev-dev libudev-dev:arm64 git \
		&& \
	apt-get clean

# install rustup
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y

# rustup directory
ENV PATH /root/.cargo/bin:$PATH

# show backtraces
ENV RUST_BACKTRACE 1

# show tools
RUN rustup toolchain install 1.50.0 && rustup default 1.50.0 && rustc -vV && cargo -V

# build parity
RUN git clone -b ${UPSTREAM_VERSION} https://github.com/openethereum/openethereum /openethereum
RUN cd /openethereum && \
	mkdir -p .cargo && \
	echo '[target.aarch64-unknown-linux-gnu]\n\
	linker = "aarch64-linux-gnu-gcc"\n'\
	>>.cargo/config && \
	cat .cargo/config	
	
RUN	cd /openethereum && \
	if [ "$TARGETARCH" = "arm64" ]; then export ARCH="aarch64"; else export ARCH="x86_64"; fi && \
 	rustup target add ${ARCH}-unknown-linux-gnu && \
	cargo build --target ${ARCH}-unknown-linux-gnu --release --verbose

RUN if [ "$TARGETARCH" = "arm64" ]; then export ARCH="aarch64"; else export ARCH="x86_64"; fi && \	
	/usr/bin/${ARCH}-linux-gnu-strip /openethereum/target/${ARCH}-unknown-linux-gnu/release/openethereum && \
	cp /openethereum/target/${ARCH}-unknown-linux-gnu/release/openethereum /usr/local/bin/openethereum

FROM debian:buster-slim

COPY --from=builder /usr/local/bin/openethereum /usr/local/bin/openethereum

ENTRYPOINT [ "sh", "-c", "exec openethereum --jsonrpc-port 8545 --jsonrpc-interface all --jsonrpc-hosts all --jsonrpc-cors all --ws-interface 0.0.0.0 --ws-port 8546 --ws-origins all --ws-hosts all --ws-max-connections 1000 --metrics --metrics-interface=all ${EXTRA_OPTS}"]
