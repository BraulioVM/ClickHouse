# docker build -t yandex/clickhouse-pvs-test .

FROM yandex/clickhouse-binary-builder

RUN apt-get update --yes \
    && apt-get install \
        bash \
        wget \
        software-properties-common \
        gpg-agent \
        debsig-verify \
        strace \
        protobuf-compiler \
        protobuf-compiler-grpc \
        libprotoc-dev \
        libgrpc++-dev \
        libc-ares-dev \
        --yes --no-install-recommends

#RUN wget -nv -O - http://files.viva64.com/etc/pubkey.txt | sudo apt-key add -
#RUN sudo wget -nv -O /etc/apt/sources.list.d/viva64.list http://files.viva64.com/etc/viva64.list
#
#RUN apt-get --allow-unauthenticated update -y \
#    && env DEBIAN_FRONTEND=noninteractive \
#        apt-get --allow-unauthenticated install --yes --no-install-recommends \
#            pvs-studio

ENV PKG_VERSION="pvs-studio-latest"

RUN set -x \
    && export PUBKEY_HASHSUM="686e5eb8b3c543a5c54442c39ec876b6c2d912fe8a729099e600017ae53c877dda3368fe38ed7a66024fe26df6b5892a" \
    && wget -nv https://files.viva64.com/etc/pubkey.txt -O /tmp/pubkey.txt \
    && echo "${PUBKEY_HASHSUM} /tmp/pubkey.txt" | sha384sum -c \
    && apt-key add /tmp/pubkey.txt \
    && wget -nv "https://files.viva64.com/${PKG_VERSION}.deb" \
    && { debsig-verify ${PKG_VERSION}.deb \
    || echo "WARNING: Some file was just downloaded from the internet without any validation and we are installing it into the system"; } \
    && dpkg -i "${PKG_VERSION}.deb"

CMD echo "Running PVS version $PKG_VERSION" && cd /repo_folder && pvs-studio-analyzer credentials $LICENCE_NAME $LICENCE_KEY -o ./licence.lic  \
    && cmake . -D"ENABLE_EMBEDDED_COMPILER"=OFF -D"USE_INTERNAL_PROTOBUF_LIBRARY"=OFF -D"USE_INTERNAL_GRPC_LIBRARY"=OFF \
    && ninja re2_st clickhouse_grpc_protos \
    && pvs-studio-analyzer analyze -o pvs-studio.log -e contrib -j 4 -l ./licence.lic; \
    cp /repo_folder/pvs-studio.log /test_output; \
    plog-converter -a GA:1,2 -t fullhtml -o /test_output/pvs-studio-html-report pvs-studio.log; \
    plog-converter -a GA:1,2 -t tasklist -o /test_output/pvs-studio-task-report.txt pvs-studio.log
