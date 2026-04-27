# Install

## uv

### Create an Environment

```sh
uv venv  --relocatable --python-preference=only-managed  --python=3.12 --prompt=my_env venv
```

### uv/Apex

[source](https://github.com/astral-sh/uv/issues/7771#issuecomment-2885423024)

```sh
uv pip install -v \
    --disable-pip-version-check \
    --no-cache-dir \
    --no-build-isolation \
    --config-setting '"--build-option=--cpp_ext"' \
    --config-setting '"--build-option=--cuda_ext"' \
    git+https://github.com/NVIDIA/apex.git@master
```

## Unsloth.ai

### On GPSC7

- Set proxies(http and https) and correct `tmp` directories in `.profile`
- `nrc_profile.sh -c` : To update the profile
- Created user directory and set symlinks for `.cache` and `.conda`
- `conda create --name unsloth_env python=3.10`
- `source activate unsloth_env` : I didn't load moniconda module because that was introducing python 3.9 in the system path and I was not able to remove it.
- `conda install pytorch-cuda=12.1 pytorch cudatoolkit xformers -c pytorch -c nvidia -c xformers`
- `pip install "unsloth[colab-new] @ git+https://github.com/unslothai/unsloth.git"`
- `pip install --no-deps trl peft accelerate bitsandbytes`
- I still got torch not found error, even though Step 7 was successful, so I just reinstalled it and then it worked.
- Launch jupyter in a job with `partition=gpu_a100`, `account=nrc_ict__gpu_a100` and `qos=low`
