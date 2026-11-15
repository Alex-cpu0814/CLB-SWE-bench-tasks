FROM public.ecr.aws/docker/library/eclipse-temurin:8-jdk-jammy@sha256:d57e5d0e3e5dd4cabb74feccfdb36249a58c76a3354a33ee8737813df93b7e0d

RUN if ! command -v ant >/dev/null || ! dpkg-query -W pocl-opencl-icd >/dev/null 2>&1; then \
      sed -i 's|http://archive.ubuntu.com/ubuntu|https://archive.ubuntu.com/ubuntu|g; s|http://security.ubuntu.com/ubuntu|https://security.ubuntu.com/ubuntu|g' /etc/apt/sources.list \
    && apt-get -o Acquire::Retries=4 update \
      && apt-get install -y --no-install-recommends git python3 gcc libc6-dev ant \
        libffi-dev libx11-dev libxrandr-dev libxxf86vm-dev libxcursor-dev libxinerama-dev \
        libgl-dev libegl-dev pocl-opencl-icd ocl-icd-opencl-dev \
      && rm -rf /var/lib/apt/lists/*; \
    fi
ARG REPO_URL=https://github.com/LWJGL/lwjgl3.git
ARG BASE_COMMIT=f40cd981a1ff81e3da603059a062728ee33ddf02
ARG INSTANCE_ID=clb-javac-075
LABEL benchmark.instance_id="${INSTANCE_ID}" benchmark.base_commit="${BASE_COMMIT}" \
      org.opencontainers.image.source="https://github.com/LWJGL/lwjgl3"
WORKDIR /testbed
RUN python3 - <<'PY'
import hashlib
from pathlib import Path
import shutil
import tarfile
import urllib.request
import zipfile
assets = [('/opt/clb-deps/jcommander.jar', 'https://repo.maven.apache.org/maven2/com/beust/jcommander/1.48/jcommander-1.48.jar', 'a7313fcfde070930e40ec79edf3c5948cf34e4f0d25cb3a09f9963d8bdd84113'), ('/opt/clb-deps/testng.jar', 'https://repo.maven.apache.org/maven2/org/testng/testng/6.8.21/testng-6.8.21.jar', '0811dcf08fe329ef5d4d37d51a6a00032697e7f54ba84c6687bbf7cd39d29fec'), ('/opt/clb-deps/kotlin-compiler-0.13.1513.zip', 'https://github.com/JetBrains/kotlin/releases/download/build-0.13.1513/kotlin-compiler-0.13.1513.zip', 'ac93ad3360f38ba2692a7f8bc7b7c74e8ca4cdb03e6e71377c51cd44bf842698')]
for target, url, digest in assets:
    path = Path(target)
    path.parent.mkdir(parents=True, exist_ok=True)
    with urllib.request.urlopen(url, timeout=180) as response, path.open('wb') as destination:
        shutil.copyfileobj(response, destination)
    assert hashlib.sha256(path.read_bytes()).hexdigest() == digest, str(path)
for archive, parent in [('/opt/clb-deps/kotlin-compiler-0.13.1513.zip', '/opt/clb-deps')]:
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
RUN git init . && git fetch --depth=1 --no-tags "${REPO_URL}" "${BASE_COMMIT}" && git checkout --detach FETCH_HEAD && git config core.autocrlf false && test -z "$(git status --porcelain)" && mkdir -p /benchmark /inputs /results
CMD ["/bin/bash"]
