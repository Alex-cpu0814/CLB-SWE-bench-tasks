# CLB SWE-bench tasks

Task assets for the 72-task CLB SWE-bench release.

- [tasks/](tasks/index.json) holds the task index, Base commits, and pinned Docker image references. Each `tasks/<instance_id>/task.json` holds the public statement alongside its Dockerfile when a recipe is available.
- [Rebuild the clb-pyc-086 image](tasks/clb-pyc-086/README.md) contains the Docker recipe and local verification steps for the pygame case.
- [evaluation/](evaluation/index.json) holds the evaluation index, per-case scripts, Test Patches, and grading lists.
- [gold/](gold/) holds the reference patch for each instance ID, exported byte for byte from the pinned Hugging Face dataset.
- [data/selection/](data/selection/) holds the selection workbook, Fix parent snapshot, and upstream patch extraction plans.

[task-repo.json](task-repo.json) records the origin code commit, the Hugging Face dataset revision and file checksum, and the Gold Patch checksums.

Install the CLI from the [companion code repository](https://github.com/Alex-cpu0814/CLB-SWE-bench). From this task checkout, validate the assets with `clb dataset check .`. Pass `--task-repo .` to `clb infer` and `clb eval`. The shared Python-C runner is installed with the code package. The generation runner supplies only the selected statement and Base environment to the agent.
