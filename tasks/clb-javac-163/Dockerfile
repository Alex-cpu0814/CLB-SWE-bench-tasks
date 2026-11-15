FROM public.ecr.aws/docker/library/ubuntu:16.04@sha256:1f1a2d56de1d604801a9671f301190704c25d604a416f59e03c04f5c6ffee0d6 AS builder
RUN apt-get -o Acquire::Retries=4 -o Acquire::Check-Valid-Until=false update \
    && apt-get -o Acquire::Retries=4 install -y --no-install-recommends apt-transport-https ca-certificates \
    && sed -i 's|http://archive.ubuntu.com|https://mirrors.aliyun.com|g; s|http://security.ubuntu.com|https://mirrors.aliyun.com|g' /etc/apt/sources.list
RUN apt-get -o Acquire::Retries=4 -o Acquire::Check-Valid-Until=false update \
    && apt-get -o Acquire::Retries=4 install -y --no-install-recommends clang gcc libc6-dev binutils \
       ca-certificates xz-utils \
    && rm -rf /var/lib/apt/lists/*
WORKDIR /work

FROM public.ecr.aws/docker/library/centos@sha256:a93df2e96e07f56ea48f215425c6f1673ab922927894595bb5c0ee4c5a955133 AS legacy
FROM builder
ARG REPO_URL=https://github.com/kohlschutter/junixsocket.git
ARG BASE_COMMIT=90716d6008c5c9cfa8a061f3d23531acaea29777
ARG INSTANCE_ID=clb-javac-163
RUN apt-get -o Acquire::Retries=4 -o Acquire::Check-Valid-Until=false update \
    && apt-get -o Acquire::Retries=4 install -y --no-install-recommends git python3 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
COPY --from=legacy / /opt/legacy/
RUN python3 - <<'PY'
import hashlib
from pathlib import Path
import shutil
import tarfile
import urllib.request
import zipfile
assets = [('/opt/java/OpenJDK8U-jdk_x64_linux_hotspot_8u292b10.tar.gz', 'https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u292-b10/OpenJDK8U-jdk_x64_linux_hotspot_8u292b10.tar.gz', '0949505fcf42a1765558048451bb2a22e84b3635b1a31dd6191780eeccaa4ada'), ('/opt/tools/qemu-x86_64-static', 'https://github.com/multiarch/qemu-user-static/releases/download/v7.2.0-1/qemu-x86_64-static', '7132ffd39aef71c26d3344cc0c7dffc530e10e3e720c58c8279a97ef6fdd7784'), ('/opt/deps/junit-console.jar', 'https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-console-standalone/1.6.0-M1/junit-platform-console-standalone-1.6.0-M1.jar', 'e86b2abb61724235c281cc4325329a68fe8731f84bb5628b8fbb2a09d52b117e'), ('/opt/deps/annotations.jar', 'https://repo.maven.apache.org/maven2/com/google/code/findbugs/annotations/3.0.1/annotations-3.0.1.jar', '6b47ff0a6de0ce17cbedc3abb0828ca5bce3009d53ea47b3723ff023c4742f79')]
for target, url, digest in assets:
    path = Path(target)
    path.parent.mkdir(parents=True, exist_ok=True)
    with urllib.request.urlopen(url, timeout=180) as response, path.open('wb') as destination:
        shutil.copyfileobj(response, destination)
    assert hashlib.sha256(path.read_bytes()).hexdigest() == digest, str(path)
for archive, parent in [('/opt/java/OpenJDK8U-jdk_x64_linux_hotspot_8u292b10.tar.gz', '/opt/java')]:
    source = Path(archive)
    if source.suffix == '.zip':
        with zipfile.ZipFile(str(source)) as packed:
            assert all(not name.startswith('/') and '..' not in Path(name).parts for name in packed.namelist())
            packed.extractall(parent)
    else:
        with tarfile.open(str(source)) as packed:
            packed.extractall(parent)
    source.unlink()
PY
RUN git init /testbed && git -C /testbed fetch --depth=1 --no-tags "${REPO_URL}" "${BASE_COMMIT}" && git -C /testbed checkout --detach FETCH_HEAD
ENV JAVA_HOME=/opt/java/jdk8u292-b10
ENV PATH=${JAVA_HOME}/bin:${PATH}
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
RUN chmod 0755 /opt/tools/qemu-x86_64-static \
    && ln -s /usr/bin/python3 /usr/local/bin/python \
    && mkdir -p /benchmark /inputs /results \
    && git -C /testbed config core.autocrlf false \
    && test "$(git -C /testbed rev-parse HEAD)" = "$BASE_COMMIT" \
    && test -z "$(git -C /testbed status --porcelain)" \
    && /opt/tools/qemu-x86_64-static -L /opt/legacy /opt/legacy/lib64/libc.so.6 \
    && /opt/tools/qemu-x86_64-static -L /opt/legacy ${JAVA_HOME}/bin/java \
       -Xint -Xmx64m -XX:+UseSerialGC -XX:ActiveProcessorCount=2 -version
LABEL benchmark.instance_id="${INSTANCE_ID}" benchmark.base_commit="${BASE_COMMIT}"
WORKDIR /testbed
CMD ["/bin/bash"]
