FROM alpine

RUN apk --no-cache add \
    libstdc++ \ 
    eudev-libs \
    libgcc \
    llvm \
    bash \
    clang


COPY --from=openethereum/openethereum:v3.0.0 /home/openethereum/openethereum /usr/local/bin/openethereum
RUN chmod +x /usr/local/bin/openethereum
ENTRYPOINT [ "sh", "-c", "exec openethereum --jsonrpc-port 8545 --jsonrpc-interface all --jsonrpc-hosts all --jsonrpc-cors all --ws-interface 0.0.0.0 --ws-port 8546 --ws-origins all --ws-hosts all --ws-max-connections 1000 ${EXTRA_OPTS}"]
