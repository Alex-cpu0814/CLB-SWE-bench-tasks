FROM public.ecr.aws/docker/library/eclipse-temurin:8-jdk-jammy@sha256:d57e5d0e3e5dd4cabb74feccfdb36249a58c76a3354a33ee8737813df93b7e0d

RUN apt-get update \
    && apt-get install -y --no-install-recommends git python3 gcc libc6-dev \
    && rm -rf /var/lib/apt/lists/*

ARG REPO_URL=https://github.com/LWJGL/lwjgl3.git
ARG BASE_COMMIT=1fa5a394949cad836a104d8e5093f734d26ff8e1
ARG INSTANCE_ID=clb-javac-074
LABEL benchmark.instance_id="${INSTANCE_ID}" benchmark.base_commit="${BASE_COMMIT}" \
      org.opencontainers.image.source="https://github.com/LWJGL/lwjgl3"
WORKDIR /testbed
RUN git init . && git fetch --depth=1 --no-tags "${REPO_URL}" "${BASE_COMMIT}" \
    && git config core.autocrlf false \
    && git checkout --detach "${BASE_COMMIT}" \
    && test -z "$(git status --porcelain)" \
    && mkdir -p /benchmark /inputs /results

CMD ["/bin/bash"]
