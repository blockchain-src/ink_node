FROM golang:1.22 AS build-op-node

WORKDIR /app

ENV REPO=https://github.com/ethereum-optimism/optimism.git
ENV VERSION=v1.10.0
# for verification:
ENV COMMIT=910c9ade39c0bcdff5f2badd94efbe016a428e73

RUN git clone $REPO --branch op-node/$VERSION --single-branch . && \
    git switch -c branch-$VERSION && \
    bash -c '[ "$(git rev-parse HEAD)" = "$COMMIT" ]'

RUN cd op-node && make VERSION=$VERSION op-node

FROM golang:1.22

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    netcat-openbsd \
    jq \
    curl \
    supervisor && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir -p /var/log/supervisor

# Create non-root user
RUN useradd -m -u 1000 -s /bin/bash opuser

WORKDIR /app

COPY --from=build-op-node /app/op-node/bin/op-node ./
COPY op-node/config /config
COPY op-node/entrypoint.sh entrypoint.sh
RUN chmod +x ./entrypoint.sh

# Set proper ownership
RUN chown -R opuser:opuser /app /var/log/supervisor

USER opuser

ENTRYPOINT [ "./entrypoint.sh" ]
