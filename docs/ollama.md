# oLLaMa

## Converting to oLLaMa

I don't recall on which site I found this but it was suggested to first go to f16 then quantize for better results.

- [Ollama Model File](https://github.com/ollama/ollama/blob/main/docs/modelfile.md)

### First in f16

First convert to F16.

#### Setup

```sh
git clone git@github.com:ggerganov/llama.cpp.git
cd llama.cpp
uv venv --relocatable --python=3.12 --prompt=llama.cpp venv
source venv/bin/activate
uv pip install numpy torch sentencepiece pyaml safetensors
export HF_HOME=$models/HuggingFace
```

#### Run

```python
python $HOME/git/llama.cpp/convert_hf_to_gguf.py \
  --outtype f16 \
  --outfile models--Unbabel--TowerInstruct-13B-v0.1-{ftype}.gguf \
  $HF_HOME/hub/models--Unbabel--TowerInstruct-13B-v0.1/snapshots/3965e508b334b28422969bf9a87fddbe6ee95b7f/
```

```python
python $HOME/git/llama.cpp/convert_hf_to_gguf.py \
  --outtype f16 \
  --outfile ModelSpace--GemmaX2-28-9B-v0.1-{ftype}.gguf \
  $HF_HOME/hub/models--ModelSpace--GemmaX2-28-9B-v0.1/snapshots/bd1bc3359faeb3cd01abb04ce470b410c2cc95d0
```

### Quantization

#### Setup

From `llama.cpp/` build `llama-quantize`.

```sh
mkdir build
cd build
$HOME/opt/cmake-3.30.0/bin/cmake ..
make
```

#### Run

```sh
$HOME/git/llama.cpp/build.sam/bin/llama-quantize \
  Unbabel--TowerInstruct-13B-v0.1-f16.gguf \
  Unbabel--TowerInstruct-13B-v0.1-f16.ggml-Q4_K_M.gguf \
  Q4_K_M
```

```sh
$HOME/git/llama.cpp/build.sam/bin/llama-quantize \
  ModelSpace--GemmaX2-28-9B-v0.1-f16.gguf \
  ModelSpace--GemmaX2-28-9B-v0.1-f16.ggml-Q4_K_M.gguf \
  Q4_K_M
```

### Converting to oLLaMa

```sh
ollama create --file Unbabel--TowerInstruct-13B-v0.1.Modelfile Unbabel--TowerInstruct-13B-v0.1
ollama create --file Unbabel--TowerInstruct-13B-v0.1-Q4_K_M.Modelfile Unbabel--TowerInstruct-13B-v0.1-Q4_K_M
```

where `Unbabel--TowerInstruct-13B-v0.1-Q4_K_M.Modelfile`

```
FROM $MODELS/transformers/hub/models--Unbabel--TowerInstruct-13B-v0.1/snapshots/3965e508b334b28422969bf9a87fddbe6ee95b7f/ggml-model-Q4_K_M.gguf
```

Can we make a model that automatically translate French to English by changing the template to incorporate the instruction?
The following commands embed the template in the model.

```sh
ollama create --file Unbabel--TowerInstruct-13B-v0.1-Q4_K_M.en2fr.Modelfile Unbabel--TowerInstruct-13B-v0.1-Q4_K_M.en2fr
ollama create --file Unbabel--TowerInstruct-13B-v0.1-Q4_K_M.fr2en.Modelfile Unbabel--TowerInstruct-13B-v0.1-Q4_K_M.fr2en
ollama create --file ModelSpace--GemmaX2-28-9B-v0.1-Q4_K_M.en2fr.Modelfile  ModelSpace--GemmaX2-28-9B-v0.1-Q4_K_M.en2fr
ollama create --file ModelSpace--GemmaX2-28-9B-v0.1-Q4_K_M.fr2en.Modelfile  ModelSpace--GemmaX2-28-9B-v0.1-Q4_K_M.fr2en
```

Create models we no specific template.

```sh
ollama create --file Unbabel--TowerInstruct-13B-v0.1-Q4_K_M.Modelfile  Unbabel--TowerInstruct-13B-v0.1-Q4_K_M
ollama create --file ModelSpace--GemmaX2-28-9B-v0.1-Q4_K_M.Modelfile ModelSpace--GemmaX2-28-9B-v0.1-Q4_K_M
```

Where `ModelSpace--GemmaX2-28-9B-v0.1-Q4_K_M.fr2en.Modelfile`

```
FROM ModelSpace--GemmaX2-28-9B-v0.1-f16.ggml-Q4_K_M.gguf

TEMPLATE """Translate this from French to English:\nFrench: {{ .Prompt }}\nEnglish:"""
```

Where `Unbabel--TowerInstruct-13B-v0.1-Q4_K_M.fr2en.Modelfile`

```
FROM models--Unbabel--TowerInstruct-13B-v0.1-f16.ggml-Q4_K_M.gguf

TEMPLATE """{{- range .Messages }}<|im_start|>{{ .Role }}
Translate the following text from French into English.\nFrench: {{ .Content }}\nEnglish:<|im_end|>
{{ end }}<|im_start|>assistant
"""
```

## Translate

```sh
ollama run Unbabel--TowerInstruct-13B-v0.1-Q4_K_.fr2enM < corpora/C13-1202-1333.fr
ollama run Unbabel--TowerInstruct-13B-v0.1-Q4_K_.fr2enM < corpora/C14-0508-1335.fr
```
