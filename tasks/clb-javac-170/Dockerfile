FROM public.ecr.aws/docker/library/eclipse-temurin:8-jdk-jammy@sha256:185d4ef919b98cd195690c33f3fd93bd49c66c70f0b662e54566c0dbaebea6ce
USER root
RUN sed -i 's|http://archive.ubuntu.com/ubuntu|https://archive.ubuntu.com/ubuntu|g; s|http://security.ubuntu.com/ubuntu|https://security.ubuntu.com/ubuntu|g' /etc/apt/sources.list \
    && apt-get -o Acquire::Retries=4 update && apt-get install -y --no-install-recommends git python3 ca-certificates && rm -rf /var/lib/apt/lists/*
RUN python3 - <<'PY'
import hashlib
from pathlib import Path
import shutil
import tarfile
import urllib.request
import zipfile
assets = [('/opt/deps/junixsocket-native-common-2.8.0.jar', 'https://repo.maven.apache.org/maven2/com/kohlschutter/junixsocket/junixsocket-native-common/2.8.0/junixsocket-native-common-2.8.0.jar', '88213e301b8c0b1d743f1e1926b5f5f36257b28ecf352bf9578d8c126039b557'), ('/opt/deps/compiler-annotations-1.6.3.jar', 'https://repo.maven.apache.org/maven2/com/kohlschutter/compiler-annotations/1.6.3/compiler-annotations-1.6.3.jar', '5f7e6949e47c10675c6522c134d55839a2223296ce8f184a32c19403170ef653'), ('/opt/deps/org.eclipse.jdt.annotation-2.2.700.jar', 'https://repo.maven.apache.org/maven2/org/eclipse/jdt/org.eclipse.jdt.annotation/2.2.700/org.eclipse.jdt.annotation-2.2.700.jar', '525b777435de57aa8f7d140d552ddc40ad9c6ad753f9d2bab7dd2101de60e431'), ('/opt/deps/error_prone_annotations-2.22.0.jar', 'https://repo.maven.apache.org/maven2/com/google/errorprone/error_prone_annotations/2.22.0/error_prone_annotations-2.22.0.jar', '82a027b86541f58d1f9ee020cdf6bebe82acc7a267d3c53a2ea5cd6335932bbd'), ('/opt/deps/animal-sniffer-annotations-1.23.jar', 'https://repo.maven.apache.org/maven2/org/codehaus/mojo/animal-sniffer-annotations/1.23/animal-sniffer-annotations-1.23.jar', '9ffe526bf43a6348e9d8b33b9cd6f580a7f5eed0cf055913007eda263de974d0'), ('/opt/deps/junit-platform-console-standalone-1.10.0.jar', 'https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-console-standalone/1.10.0/junit-platform-console-standalone-1.10.0.jar', '8192d377bdb53969273ca3805af0eef3b864690f7818300c0d39ac7758da39a0'), ('/opt/deps/kohlschutter-test-util-1.6.3.jar', 'https://repo.maven.apache.org/maven2/com/kohlschutter/kohlschutter-test-util/1.6.3/kohlschutter-test-util-1.6.3.jar', 'd7a87b17c05b7558efebb1ff36c739bb52c21433a1647e69d0a6116edf6e72d9'), ('/opt/deps/kohlschutter-util-1.6.3.jar', 'https://repo.maven.apache.org/maven2/com/kohlschutter/kohlschutter-util/1.6.3/kohlschutter-util-1.6.3.jar', '2775cde3790a9a26cb0aeff7a79b2311e56a2ff9fb0fa7f53f003ec8f4e4a024')]
for target, url, digest in assets:
    path = Path(target)
    path.parent.mkdir(parents=True, exist_ok=True)
    with urllib.request.urlopen(url, timeout=180) as response, path.open('wb') as destination:
        shutil.copyfileobj(response, destination)
    assert hashlib.sha256(path.read_bytes()).hexdigest() == digest, str(path)
for archive, parent in []:
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
ARG REPO_URL=https://github.com/kohlschutter/junixsocket.git
ARG BASE_COMMIT=71a134bce58a22fb17a7a14f00799196c0c71a60
ARG INSTANCE_ID=clb-javac-170
WORKDIR /testbed
RUN git init . && git fetch --depth=1 --no-tags "${REPO_URL}" "${BASE_COMMIT}" && git checkout --detach FETCH_HEAD
RUN git config --global --add safe.directory /testbed && git status --porcelain && java -version && python3 --version
ENV LC_ALL=C.UTF-8
CMD ["/bin/bash"]
