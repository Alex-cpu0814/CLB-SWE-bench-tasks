FROM public.ecr.aws/docker/library/python:3.8.20-bullseye@sha256:11da1db2f485e2560be9817f92932a29f68fefc332b0167e8c51ae308ee23337
ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV PYTHONUNBUFFERED=1 LC_ALL=C.UTF-8 LANG=C.UTF-8 PYTHONPATH=/testbed
RUN sed -i '/bullseye-security/d; s|http://deb.debian.org|https://deb.debian.org|g' /etc/apt/sources.list \
    && apt-get -o Acquire::Retries=5 update \
    && apt-get -o Acquire::Retries=5 install -y --no-install-recommends --no-upgrade gcc=4:10.2.1-1 g++=4:10.2.1-1 git ca-certificates \
    && rm -rf /var/lib/apt/lists/*
RUN python -m pip install --no-cache-dir pip==24.3.1 setuptools==68.2.2 wheel==0.42.0 setuptools_scm==6.4.2 \
       pytest==7.4.4 packaging==23.2 pluggy==1.3.0 iniconfig==2.0.0 exceptiongroup==1.2.2 tomli==2.0.1
ARG REPO_URL=https://github.com/ultrajson/ultrajson.git
ARG BASE_COMMIT=f26f9679f1b9d7fa2d0488e1e7b1256d9391a85f
ARG INSTANCE_ID=clb-pyc-071
RUN git init /testbed && cd /testbed \
    && git remote add origin "${REPO_URL}" \
    && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" \
    && git checkout --detach "${BASE_COMMIT}" \
    && git remote remove origin \
    && git config core.autocrlf false \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"
ENV SETUPTOOLS_SCM_PRETEND_VERSION=4.0.2.dev3
WORKDIR /testbed
RUN python setup.py build_ext -i --force \
    && python -c 'import ujson; print(ujson.__version__,ujson.__file__); assert ujson.__version__ == "4.0.2.dev3"; assert ujson.__file__.startswith("/testbed/ujson.")' \
    && git restore --source=HEAD --worktree -- . \
    && test -z "$(git status --porcelain)"
CMD ["/bin/bash"]
