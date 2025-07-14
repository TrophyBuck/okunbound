FROM alpine:latest

# Install unbound
RUN apk add unbound
#RUN echo "http://dl-4.alpinelinux.org/alpine/edge/community/" >> /etc/apk/repositories && \
#    apk add --update unbound bash bind-tools && \
#    rm -rf /tmp/* /var/tmp/* /var/cache/apk/*

# Expose udp port
EXPOSE 53/udp

WORKDIR /etc/unbound

# the default conf just forwards to google public dns
COPY unbound.conf /etc/unbound/

# entry point takes care of starting up unbound
COPY entrypoint.sh /bin/entrypoint.sh
RUN chmod 755 /bin/entrypoint.sh

ENTRYPOINT ["/bin/entrypoint.sh"]
