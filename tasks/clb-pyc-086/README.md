# Rebuild the Base image for clb-pyc-086

Run these commands from the task repository root with Docker configured for Linux containers.

```bash
docker build --platform linux/amd64 --no-cache --pull --progress plain   -t clb-local/pyc-case-086:rebuild-v1 ./tasks/clb-pyc-086
docker image inspect --format '{{.Id}}' clb-local/pyc-case-086:rebuild-v1
```

The [Dockerfile](Dockerfile) uses a pinned Python parent image and fetches the exact pygame Base commit with a shallow Git checkout. It installs Debian and Python dependencies and compiles the native pygame extensions. The [ignore rules](.dockerignore) limit the build context to the Dockerfile. Debian repository updates and image creation metadata can affect future rebuilds.

To evaluate a local rebuild, use a local copy of this task repository. In both `tasks/index.json` and `evaluation/index.json`, find the entry whose `instance_id` is `clb-pyc-086`. Set `image.image_id` to the ID printed above and set `image.pull_reference` to `null`. The companion CLI then uses the local image. The release indexes describe the published image.

With `clb` installed, run the dataset check and Gold evaluation from the root of that local copy.

```bash
clb dataset check .
clb eval run clb-pyc-086 --task-repo . --patch gold/clb-pyc-086.diff --output-root grading --run-id rebuild-086-check
```

The expected Gold result is `resolved` with two target tests and two regression tests passing. With the inference dependencies installed, the Base can also be checked using `clb infer clb-pyc-086 --task-repo . --preflight --output-dir runs/rebuild-086`.
