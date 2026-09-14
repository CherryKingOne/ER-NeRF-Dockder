仓库中有一个 `Dockerfile`,可以用它构建并启动一个带 CUDA/PyTorch/pytorch3d 等依赖的容器环境,但需要注意这个 Dockerfile 只是准备了运行环境(不包含数据预处理和训练的自动化流程),启动容器后仍需在容器内手动执行 `data_utils/process.py`、`main.py` 等训练/测试命令。

## Dockerfile 内容说明

该 Dockerfile 基于 `nvcr.io/nvidia/cuda:11.7.1-cudnn8-devel-ubuntu20.04` 镜像,通过 Miniconda 安装 Python 3.10、PyTorch 1.13.1、CUDA Toolkit 11.7.1、pytorch3d、tensorflow-gpu 等依赖,并将整个仓库代码 `COPY` 到镜像内的 `/ernerf` 目录,同时声明了 `/ernerf` 为 volume。容器启动后默认进入 `/bin/bash`。

## 构建镜像

在仓库根目录执行:

```bash
docker build -t ernerf .
```

## 启动容器

由于训练/推理需要 GPU,启动时需加 `--gpus all`(需要主机装有 nvidia-container-toolkit)。另外由于 Dockerfile 中把 `/ernerf` 声明为 volume,建议挂载本地代码/数据目录以便持久化数据和权重:

```bash
docker run --gpus all -it \
  -v $(pwd):/ernerf \
  ernerf
```

进入容器后(默认工作目录为 `/ernerf`),即可按照 README 中的说明继续操作,例如:

- 数据预处理:
```bash
python data_utils/process.py data/<ID>/<ID>.mp4
```

- 训练:
```bash
python main.py data/obama/ --workspace trial_obama/ -O --iters 100000
python main.py data/obama/ --workspace trial_obama/ -O --iters 125000 --finetune_lips --patch_size 32
``` 

- 测试/推理:
```bash
python main.py data/obama/ --workspace trial_obama/ -O --test
```

需要注意:Dockerfile 本身不会自动下载 face-parsing 模型、3DMM 模型或数据集,这些仍需按 README 中「Preparation」部分的说明在容器内(或挂载的目录中)手动下载。
