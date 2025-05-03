ARG PHP_VERSION
ARG BREF_VERSION
FROM bref/build-php-$PHP_VERSION:$BREF_VERSION AS ext

ENV LD_LIBRARY_PATH=/usr/lib:/usr/lib64:$LD_LIBRARY_PATH

# This extension requires the libffi library to be installed.
RUN yum install -y libffi-devel

WORKDIR ${PHP_BUILD_DIR}/ext/ffi

# To enable the FFI extension, PHP has to be configured with --with-ffi.
RUN echo $(pwd) && phpize && \
    ./configure \
    --with-ffi;

RUN make -j $(nproc)
RUN make install

# The behaviour of these functions is affected by settings in php.ini.
RUN cp `php-config --extension-dir`/ffi.so /tmp/ffi.so
RUN echo 'extension=ffi.so' > /tmp/ext.ini

RUN php /bref/lib-copy/copy-dependencies.php /tmp/ffi.so /tmp/extension-libs


FROM scratch

COPY --from=ext /tmp/ffi.so /opt/bref/extensions/ffi.so
COPY --from=ext /tmp/ext.ini /opt/bref/etc/php/conf.d/ext-ffi.ini
COPY --from=ext /tmp/extension-libs /opt/lib

# ex: ffi.enable true
