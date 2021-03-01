FROM alpine

# Install build deps + permanent dep: libaio
RUN apk --no-cache add \
    	make \
	alpine-sdk \
	zlib-dev \
	libaio-dev \
	linux-headers \
	coreutils \
	libaio && \
    git clone https://github.com/axboe/fio && \
    cd fio && \
    ./configure && \
    make -j`nproc` && \
    make install && \
    cd .. && \
    rm -rf fio && \
    apk --no-cache del \
    	make \
	alpine-sdk \
	zlib-dev \
	libaio-dev \
	linux-headers \
	coreutils

RUN addgroup -S app \
    && adduser -S -g app app

VOLUME /tmp

WORKDIR /tmp

COPY ./docker-entrypoint.sh /

RUN chmod +x /docker-entrypoint.sh && \
    chown -R app:app /docker-entrypoint.sh

USER app

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["fio"]