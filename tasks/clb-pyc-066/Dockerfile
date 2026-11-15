FROM public.ecr.aws/docker/library/python:3.8.20-bullseye@sha256:11da1db2f485e2560be9817f92932a29f68fefc332b0167e8c51ae308ee23337
ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV PYTHONUNBUFFERED=1 LC_ALL=C.UTF-8 LANG=C.UTF-8 PYTHONPATH=/testbed/src MAX_CONCURRENCY=4
RUN sed -i '/bullseye-security/d; s|http://deb.debian.org|https://deb.debian.org|g' /etc/apt/sources.list \
    && apt-get -o Acquire::Retries=5 update \
    && apt-get -o Acquire::Retries=5 install -y --no-install-recommends --no-upgrade \
       gcc=4:10.2.1-1 libjpeg62-turbo-dev=1:2.0.6-4 \
       zlib1g-dev=1:1.2.11.dfsg-2+deb11u2 libfreetype6-dev=2.10.4+dfsg-1+deb11u1 \
       libopenjp2-7-dev=2.4.0-3 libtiff-dev=4.2.0-1+deb11u5 git ca-certificates \
    && rm -rf /var/lib/apt/lists/*
RUN python -m pip install --no-cache-dir pip==24.3.1 setuptools==68.2.2 wheel==0.42.0 \
       pytest==7.4.4 packaging==23.2 pluggy==1.3.0 iniconfig==2.0.0 exceptiongroup==1.2.2 tomli==2.0.1
ARG REPO_URL=https://github.com/python-pillow/Pillow.git
ARG BASE_COMMIT=3fb62360e61949a2405db0771ad89f1c9a2b25e4
ARG INSTANCE_ID=clb-pyc-066
RUN git init /testbed && cd /testbed \
    && git remote add origin "${REPO_URL}" \
    && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" \
    && git checkout --detach FETCH_HEAD \
    && git remote remove origin \
    && git config core.autocrlf false \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"
WORKDIR /testbed
ENV PYTHONPATH=/testbed:/testbed/Tests
RUN python setup.py build_ext -i \
    && python -c 'import PIL; from PIL import Image; print(PIL.__version__,PIL.__file__); assert Image.core.PILLOW_VERSION == PIL.__version__' \
    && git restore --source=HEAD --worktree -- . \
    && test -z "$(git status --porcelain)"
CMD ["/bin/bash"]
