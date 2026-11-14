# 镜像审核与发布记录 · 2026-11-10

本记录根据当天保存的镜像审核报告、匿名拉取记录、Base / Gold 对照结果和任务仓库提交整理。
范围为 pygame 的七个 Python–C 案例。`clb-pyc-075` 对应 Python–C 任务，Java 075 属于另一条发布记录。

## 086 试点

旧版 `clb-pyc-086` 的容器内未发现 Fix 提交对象，但 `docker history` 的 `CreatedBy` 字段仍含 Fix SHA 前缀。因此审核同时覆盖容器内容和镜像构建元数据。重建版保留相同的 Base 提交和源码树，记录的系统包与 Python 包清单无差异。新版镜像的配置和历史均通过 Fix 前缀扫描。构建配方和本地复核步骤见 [086 镜像说明](../tasks/clb-pyc-086/README.md)。

## 审核和评分

审核确认 HEAD 与任务指定 Base 一致，Git 浅仓库只可达一个提交，工作区干净。远程、标签、Fix 对象、Fix reflog 命中、补丁文件及评测目录扫描均无发现。随后检查镜像配置和构建历史，并分别运行 Base 和 Gold 对照。目标测试在 Base 失败、Gold 通过，两边的回归检查均通过。发布 `v2` 后，按固定摘要匿名拉取并审核镜像。

F2P 是目标测试，P2P 是回归测试。表中依次为 F2P 和 P2P 的通过数 / 预期数。

| 任务 | 已审核的 Docker Hub 标签 | Base（F2P，P2P） | Gold（F2P，P2P） |
|---|---|---|---|
| `clb-pyc-086` | [`v2`](https://hub.docker.com/r/yutu0814/clb.eval.x86_64.clb-python-c-086/tags) | 0/2，2/2 | 2/2，2/2 |
| `clb-pyc-077` | [`v2`](https://hub.docker.com/r/yutu0814/clb.eval.x86_64.clb-python-c-077/tags) | 0/1，3/3 | 1/1，3/3 |
| `clb-pyc-081` | [`v2`](https://hub.docker.com/r/yutu0814/clb.eval.x86_64.clb-python-c-081/tags) | 0/1，3/3 | 1/1，3/3 |
| `clb-pyc-082` | [`v2`](https://hub.docker.com/r/yutu0814/clb.eval.x86_64.clb-python-c-082/tags) | 0/2，2/2 | 2/2，2/2 |
| `clb-pyc-075` | [`v2`](https://hub.docker.com/r/yutu0814/clb.eval.x86_64.clb-python-c-075/tags) | 0/1，3/3 | 1/1，3/3 |
| `clb-pyc-090` | [`v2`](https://hub.docker.com/r/yutu0814/clb.eval.x86_64.clb-python-c-090/tags) | 0/1，3/3 | 1/1，3/3 |
| `clb-pyc-091` | [`v2`](https://hub.docker.com/r/yutu0814/clb.eval.x86_64.clb-python-c-091/tags) | 0/1，3/3 | 1/1，3/3 |

## 固定镜像引用

以下引用已与任务仓库的 [镜像索引](../tasks/index.json) 核对。

~~~text
yutu0814/clb.eval.x86_64.clb-python-c-086@sha256:2f4eebff2144f1993172f297882f97c43bfde95fe9aba6361c99455e7fe6a724
yutu0814/clb.eval.x86_64.clb-python-c-077@sha256:7707b84c946a4597488097a8db50e6fa4732fd7612035c723cbd95d2e7a1103f
yutu0814/clb.eval.x86_64.clb-python-c-081@sha256:05fb0b27f5330e9a842d9e3263020c214623933f2eb2cb590f2e95737f647457
yutu0814/clb.eval.x86_64.clb-python-c-082@sha256:dcf5deee19c7b652635dc85f52b205afeed4862a8318955cca32dc25cd0033d5
yutu0814/clb.eval.x86_64.clb-python-c-075@sha256:7b465322ad5678d225697bb7e633a00a47d91702620bf5547e386c627d04d3fe
yutu0814/clb.eval.x86_64.clb-python-c-090@sha256:0885aad791f682a1089d444fe3922271eaeeeae285708c62c2a46505c77b5d08
yutu0814/clb.eval.x86_64.clb-python-c-091@sha256:5ee498265139a4da765c6383ccaf11bed389557c29b1094c6dc8cc8c6141a931
~~~

## 任务仓库更新

- 086 试点。 [任务仓库提交](https://github.com/Alex-cpu0814/CLB-SWE-bench-tasks/commit/2b4c51d46be92b1cdfdd6bd40535ed51d88122cd)
- 077、081、082。 [任务仓库提交](https://github.com/Alex-cpu0814/CLB-SWE-bench-tasks/commit/cbdfc61bfd07ddf408d012b889f5f2a5010bf7db)
- 075、090、091。 [任务仓库提交](https://github.com/Alex-cpu0814/CLB-SWE-bench-tasks/commit/a0329633ec253049c1c4c424d7218c932d32449f)
