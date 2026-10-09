FROM haproxy:3.0.29-alpine

USER root
RUN apk add --no-cache keepalived iproute2 iputils

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
