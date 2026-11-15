FROM public.ecr.aws/ubuntu/ubuntu:18.04@sha256:975b418c2c6c72bdfb978ac631c3fae2f42b752f5ab29dec706a01f3f6acc14b
ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN apt-get update && apt-get install -y --no-install-recommends build-essential ca-certificates git python2.7 python2.7-dev openjdk-8-jdk-headless && rm -rf /var/lib/apt/lists/*
ENV PYTHONUNBUFFERED=1 LC_ALL=C.UTF-8 LANG=C.UTF-8 JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 PYTHONPATH=/testbed:/testbed/src/main/python PYTHONDONTWRITEBYTECODE=1
ARG REPO_URL=https://github.com/bcdev/jpy.git
ARG BASE_COMMIT=20cdbb714300bdb2502d9ac44740c85c64cce691
ARG INSTANCE_ID=clb-pyc-176
RUN git init /testbed && cd /testbed && git remote add origin "${REPO_URL}" && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" && git checkout --detach "${BASE_COMMIT}" && git remote remove origin && git config core.autocrlf false && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"
WORKDIR /testbed
RUN python2.7 setup.py clean build_ext --inplace --force && mkdir -p target/test-classes && javac -d target/test-classes src/main/java/org/jpy/annotations/*.java src/test/java/org/jpy/fixtures/*.java && python2.7 -c "import jpy,jpyutil; print(jpy.__file__); assert jpy.__file__.startswith('/testbed/'); jpyutil.init_jvm(); assert jpy.has_jvm()" && printf '%s\n' '/build/' '/target/' '/src/main/python/*.so' >> .git/info/exclude && test -z "$(git status --porcelain)"
CMD ["/bin/bash"]
