FROM public.ecr.aws/docker/library/ubuntu:22.04@sha256:5ec03bb3441e8b0bf3b4f9cd4629a1ae763010dc3035bb8da3ae6cf026486401
ENV DEBIAN_FRONTEND=noninteractive JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64 LANG=C.UTF-8 LC_ALL=C.UTF-8
RUN apt-get -o Acquire::Retries=4 update \
    && apt-get -o Acquire::Retries=4 install -y --no-install-recommends ca-certificates \
    && sed -i 's|http://archive.ubuntu.com/ubuntu|https://archive.ubuntu.com/ubuntu|g; s|http://security.ubuntu.com/ubuntu|https://security.ubuntu.com/ubuntu|g' /etc/apt/sources.list
RUN apt-get -o Acquire::Retries=4 update && apt-get install -y --no-install-recommends ant ca-certificates git openjdk-17-jdk-headless gcc g++ libx11-dev make unzip python3 && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN apt-get update && apt-get install -y --no-install-recommends libffi-dev=3.4.2-4 && apt-get clean && rm -rf /var/lib/apt/lists/* && ln -s /usr/bin/python3 /usr/local/bin/python

ARG REPO_URL=https://github.com/LWJGL/lwjgl3.git
ARG BASE_COMMIT=307ebed6ef6ba420b2165ddb2ec21dd9a37479df
ARG INSTANCE_ID=clb-javac-056
LABEL benchmark.instance_id="${INSTANCE_ID}" benchmark.base_commit="${BASE_COMMIT}" org.opencontainers.image.source="https://github.com/LWJGL/lwjgl3"
WORKDIR /testbed
# Only a depth-one Base object store and Base-declared dependency cache enter the image.
RUN git init . && git fetch --depth=1 --no-tags "${REPO_URL}" "${BASE_COMMIT}" && git checkout --detach FETCH_HEAD && git config core.autocrlf false && git checkout --detach "${BASE_COMMIT}" && test -z "$(git status --porcelain)"

RUN python3 - <<'PY'
import hashlib
import shutil
import subprocess
import xml.etree.ElementTree as ET
from pathlib import Path

names = sorted({e.attrib.get('name', e.attrib.get('property', '')) for e in ET.parse('config/build-bindings.xml').iter() if e.attrib.get('name', e.attrib.get('property', '')).startswith('binding.')} - {'binding.DISABLE_CHECKS'})
flags = ['-D' + name + '=' + ('true' if name == 'binding.vulkan' else 'false') for name in names]
archive = Path('/usr/lib/x86_64-linux-gnu/libffi_pic.a')
assert hashlib.sha256(archive.read_bytes()).hexdigest() == 'fe417849a4af3c711b6b94f45219fbeb788896aaab478dd1806ecc23a8cfb85e'
destination = Path('bin/libs/native/linux/x64/org/lwjgl/libffi.a')
destination.parent.mkdir(parents=True, exist_ok=True)
shutil.copyfile(archive, destination)
subprocess.run(['ant', '-Dbuild.type=release/3.3.0', *flags, 'compile-templates'], check=True)
subprocess.run(['ant', '-Dbuild.type=release/3.3.0', '-Dbuild.offline=false', *flags, 'compile-native'], check=True)
PY

# Keep dependencies; evaluations must compile project classes and JNI from current sources.
RUN python3 - <<'PY'
import shutil
from pathlib import Path
root = Path('/testbed').resolve()
for relative in ('bin/classes', 'bin/linux'):
    target = (root / relative).resolve()
    assert target.is_relative_to(root / 'bin')
    if target.exists():
        shutil.rmtree(target)
for library in (root / 'bin/libs/native').rglob('*.so'):
    library.unlink()
PY
RUN git restore --source="${BASE_COMMIT}" --worktree -- . \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}" \
    && test -z "$(git status --porcelain)" \
    && mkdir -p /benchmark /inputs /results
CMD ["/bin/bash"]
