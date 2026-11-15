FROM public.ecr.aws/docker/library/ubuntu:22.04@sha256:5ec03bb3441e8b0bf3b4f9cd4629a1ae763010dc3035bb8da3ae6cf026486401
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get -o Acquire::Retries=4 update \
    && apt-get -o Acquire::Retries=4 install -y --no-install-recommends ca-certificates \
    && sed -i 's|http://archive.ubuntu.com/ubuntu|https://archive.ubuntu.com/ubuntu|g; s|http://security.ubuntu.com/ubuntu|https://security.ubuntu.com/ubuntu|g' /etc/apt/sources.list
RUN apt-get -o Acquire::Retries=4 update && apt-get -o Acquire::Retries=4 install -y --no-install-recommends ca-certificates \
    && sed -i 's|https://archive.ubuntu.com|https://mirrors.aliyun.com|g; s|https://security.ubuntu.com|https://mirrors.aliyun.com|g' /etc/apt/sources.list \
    && apt-get update && apt-get -o Acquire::Retries=4 install -y --no-install-recommends \
       ant git python3 openjdk-11-jdk-headless gcc g++ make libclang-14-dev libffi-dev \
    && ln -s /usr/bin/python3 /usr/local/bin/python \
    && test -r /usr/lib/llvm-14/lib/libclang.so \
    && rm -rf /var/lib/apt/lists/*
RUN apt-get update && apt-get -o Acquire::Retries=4 install -y --no-install-recommends libx11-dev \
    && rm -rf /var/lib/apt/lists/*
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
ENV PATH="${JAVA_HOME}/bin:${PATH}"
ARG REPO_URL=https://github.com/LWJGL/lwjgl3.git
ARG BASE_COMMIT=2c8811383487cd827f375abdc534a5d40a174edc
ARG INSTANCE_ID=clb-javac-062
LABEL benchmark.instance_id="${INSTANCE_ID}" benchmark.base_commit="${BASE_COMMIT}" org.opencontainers.image.source="https://github.com/LWJGL/lwjgl3"
WORKDIR /testbed
RUN git init . && git fetch --depth=1 --no-tags "${REPO_URL}" "${BASE_COMMIT}" && git checkout --detach FETCH_HEAD && git config core.autocrlf false \
    && git checkout --detach "${BASE_COMMIT}" && test -z "$(git status --porcelain)"
RUN python3 - <<'PY'
import hashlib
from pathlib import Path
import shutil
import tarfile
import urllib.request
import zipfile
assets = [('/testbed/bin/libs/java/jsr305.jar', 'https://repo.maven.apache.org/maven2/com/google/code/findbugs/jsr305/3.0.2/jsr305-3.0.2.jar', '766ad2a0783f2687962c8ad74ceecc38a28b9f72a2d085ee438b7813e928d0c7'), ('/testbed/bin/libs/java/jcommander.jar', 'https://repo.maven.apache.org/maven2/com/beust/jcommander/1.72/jcommander-1.72.jar', 'e0de160b129b2414087e01fe845609cd55caec6820cfd4d0c90fabcc7bdb8c1e'), ('/testbed/bin/libs/java/testng.jar', 'https://repo.maven.apache.org/maven2/org/testng/testng/6.14.3/testng-6.14.3.jar', '7266b68b9668ea0c4e86e384fd72ccaa29a20a2538b20c586fd3902f008b114e'), ('/testbed/bin/libs/java/joml.jar', 'https://repo.maven.apache.org/maven2/org/joml/joml/1.9.16/joml-1.9.16.jar', '324005d2d924de9003c75627eea2123cc5a1e6905e01510cc5948b0978bf4709'), ('/testbed/bin/libs/java/jmh-core.jar', 'https://repo.maven.apache.org/maven2/org/openjdk/jmh/jmh-core/1.21/jmh-core-1.21.jar', '79aecd73ffb5d95d88b1ac36b505fa30ae3e83788e936838e2be9a51074fd2dd'), ('/testbed/bin/libs/java/jmh-generator-annprocess.jar', 'https://repo.maven.apache.org/maven2/org/openjdk/jmh/jmh-generator-annprocess/1.21/jmh-generator-annprocess-1.21.jar', 'c5636ecbc617732f5acf41f94521cf6ae4f5bc6ad3512e82416fbbaabe805fe5'), ('/testbed/bin/libs/java/jopt-simple.jar', 'https://repo.maven.apache.org/maven2/net/sf/jopt-simple/jopt-simple/5.0.4/jopt-simple-5.0.4.jar', 'df26cc58f235f477db07f753ba5a3ab243ebe5789d9f89ecf68dd62ea9a66c28'), ('/testbed/bin/libs/java/commons-math3.jar', 'https://repo.maven.apache.org/maven2/org/apache/commons/commons-math3/3.6.1/commons-math3-3.6.1.jar', '1e56d7b058d28b65abd256b8458e3885b674c1d588fa43cd7d1cbb9c7ef2b308'), ('/opt/kotlin-compiler.zip', 'https://github.com/JetBrains/kotlin/releases/download/v1.3.50/kotlin-compiler-1.3.50.zip', '69424091a6b7f52d93eed8bba2ace921b02b113dbb71388d704f8180a6bdc6ec')]
for target, url, digest in assets:
    path = Path(target)
    path.parent.mkdir(parents=True, exist_ok=True)
    with urllib.request.urlopen(url, timeout=180) as response, path.open('wb') as destination:
        shutil.copyfileobj(response, destination)
    assert hashlib.sha256(path.read_bytes()).hexdigest() == digest, str(path)
for archive, parent in [('/opt/kotlin-compiler.zip', '/testbed/bin/libs')]:
    source = Path(archive)
    if source.suffix == '.zip':
        with zipfile.ZipFile(source) as packed:
            assert all(not name.startswith('/') and '..' not in Path(name).parts for name in packed.namelist())
            packed.extractall(parent)
    else:
        with tarfile.open(source) as packed:
            packed.extractall(parent)
    source.unlink()
PY
RUN git init -q /opt/dyncall && git -C /opt/dyncall fetch --depth=1 --no-tags https://github.com/LWJGL-CI/dyncall.git 42de4c57115e34e8734c5b0c0e2cbe03244ab7b6 && git -C /opt/dyncall checkout --detach FETCH_HEAD && test "$(git -C /opt/dyncall rev-parse HEAD)" = "42de4c57115e34e8734c5b0c0e2cbe03244ab7b6"
RUN python3 - <<'PY'
"""Restore Unix line endings in the pinned LWJGL-CI Git mirror snapshot."""
from pathlib import Path

root = Path("/opt/dyncall")
changed = []
for path in root.rglob("*"):
    if not path.is_file():
        continue
    data = path.read_bytes()
    if b"\x00" in data or b"\r\n" not in data:
        continue
    try:
        data.decode("utf-8")
    except UnicodeDecodeError:
        continue
    path.write_bytes(data.replace(b"\r\n", b"\n"))
    changed.append(str(path.relative_to(root)))
print("Normalized CRLF in", len(changed), "dyncall text files")
PY
RUN git -C /opt/dyncall config core.autocrlf false && cd /opt/dyncall && ./configure && make -j2 \
    && mkdir -p /testbed/bin/libs/native/linux/x64/org/lwjgl \
    && cp dyncall/libdyncall_s.a dyncallback/libdyncallback_s.a dynload/libdynload_s.a /testbed/bin/libs/native/linux/x64/org/lwjgl/
RUN mkdir -p bin/libs/native/linux/x64/org/lwjgl \
    && cp /usr/lib/x86_64-linux-gnu/libffi_pic.a bin/libs/native/linux/x64/org/lwjgl/libffi.a \
    && git restore --source="${BASE_COMMIT}" --worktree -- . \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}" \
    && test -z "$(git status --porcelain)" \
    && mkdir -p /benchmark /inputs /results
CMD ["/bin/bash"]
