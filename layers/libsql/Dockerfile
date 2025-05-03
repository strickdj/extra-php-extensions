ARG PHP_VERSION
ARG BREF_VERSION
FROM bref/build-php-$PHP_VERSION:$BREF_VERSION AS ext

ENV LD_LIBRARY_PATH=/usr/lib:/usr/lib64:$LD_LIBRARY_PATH

RUN yum install -y clang

# need cmake version 18 or later, as of now aws lambda has 17.5
RUN curl https://cmake.org/files/v3.18/cmake-3.18.0.tar.gz --output cmake-3.18.0.tar.gz \
    && tar -xvzf cmake-3.18.0.tar.gz \
    && cd cmake-3.18.0 \
    && ./bootstrap && make && make install;

# install rust toolchain
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y;

# clone turso-client-php
RUN git clone https://github.com/tursodatabase/turso-client-php.git

WORKDIR ${PHP_BUILD_DIR}/turso-client-php/

RUN . "$HOME/.cargo/env" && PATH=/usr/local/bin:$PATH \
    && PHP=/opt/bin/php PHP_CONFIG=/opt/bin/php-config cargo

RUN . "$HOME/.cargo/env" && PATH=/usr/local/bin:$PATH \
    && PHP=/opt/bin/php PHP_CONFIG=/opt/bin/php-config cargo build --release
RUN cp ./target/release/liblibsql_php.so /tmp/liblibsql_php.so

RUN echo 'extension=liblibsql_php.so' > /tmp/ext.ini
RUN php /bref/lib-copy/copy-dependencies.php /tmp/liblibsql_php.so /tmp/extension-libs


FROM scratch

COPY --from=ext /tmp/liblibsql_php.so /opt/bref/extensions/liblibsql_php.so
COPY --from=ext /tmp/ext.ini /opt/bref/etc/php/conf.d/ext-liblibsql_php.ini
COPY --from=ext /tmp/extension-libs /opt/lib

#docker run -it --entrypoint /bin/bash bref/build-php-83:2

