FROM public.ecr.aws/ubuntu/ubuntu:18.04@sha256:975b418c2c6c72bdfb978ac631c3fae2f42b752f5ab29dec706a01f3f6acc14b
ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN apt-get update && apt-get install -y --no-install-recommends build-essential ca-certificates git pkg-config python-minimal python2.7 python2.7-dev libsqlite3-dev && rm -rf /var/lib/apt/lists/*
ENV PYTHONUNBUFFERED=1 LC_ALL=C.UTF-8 LANG=C.UTF-8 PYTHONPATH=/testbed/build/lib.linux-x86_64-2.7
ARG REPO_URL=https://github.com/ghaering/pysqlite.git
ARG BASE_COMMIT=d7c44a6b62ccf27ae2a006d32dc25cd49ca45320
ARG INSTANCE_ID=clb-pyc-156
RUN git init /testbed && cd /testbed && git remote add origin "${REPO_URL}" && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" && git checkout --detach "${BASE_COMMIT}" && git remote remove origin && git config core.autocrlf false && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"
WORKDIR /testbed
RUN python2.7 setup.py build_ext --force build_py --force && python2.7 -c "import pysqlite2.dbapi2 as db,pysqlite2._sqlite as native; print(db.version,db.sqlite_version,native.__file__); assert db.version == '2.6.3'; assert native.__file__.startswith('/testbed/build/lib.linux-x86_64-2.7/pysqlite2/_sqlite')" && git checkout HEAD -- . && printf '%s\n' '/build/' '*.pyc' >> .git/info/exclude && test -z "$(git status --porcelain)"
CMD ["/bin/bash"]
