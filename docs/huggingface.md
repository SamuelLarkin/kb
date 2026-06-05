# HuggingFace

## Generate.py

```python tile="generate.py"
#!/usr/bin/env  python3
"""
uv venv --relocatable --managed-python --prompt=axolotl --python=3.12 venv
source venv/bin/activate ""
uv pip install datasets sacrebleu torch transformers typer
"""

import json
import sys
from pathlib import Path
from typing import Any, Sequence

import typer

app = typer.Typer()
DEFAULT_SYSTEM = "You are a helpful AI assistant named SmolLM, trained by Hugging Face."


def convert2messages(example: dict[str, Any]) -> dict[str, Any]:
    """
    Transform into messages.
    """
    example["messages"] = [
        {
            "role": "system",
            "content": DEFAULT_SYSTEM,
        },
        {
            "role": "user",
            "content": f"""{example["instruction"]}\n{example["input"]}""",
        },
    ]

    return example


def apply_chat_template(example: dict[str, Any], tokenizer) -> dict[str, Any]:
    """
    Tokenizes a single example.
    """
    example["chat"] = tokenizer.apply_chat_template(
        example["messages"],
        tokenize=False,
        add_generation_prompt=True,
    )

    return example


def infer_batch(
    examples: dict[str, list],
    tokenizer,
    model,
    max_new_tokens: int,
    temperature: float,
    top_p: float,
    do_sample: bool,
) -> dict[str, list]:
    """
    Performs inference on a single batch.

    Return: {"prediction": predictions}
    """
    import torch

    encoded_batch = tokenizer(
        examples["chat"],
        return_tensors="pt",
        padding=True,
        truncation=True,
    )

    # CRITICAL: Move inputs to the same device as the model
    encoded_batch = {k: v.to(model.device) for k, v in encoded_batch.items()}

    with torch.inference_mode():
        outputs = model.generate(
            **encoded_batch,
            max_new_tokens=max_new_tokens,
            temperature=temperature,
            top_p=top_p,
            do_sample=do_sample,
        )
        outputs = outputs.cpu()

    response_ids = outputs[:, encoded_batch["input_ids"].shape[1] :]
    predictions = tokenizer.batch_decode(response_ids, skip_special_tokens=True)

    return {"prediction": predictions}


def metrics(
    hyps: Sequence[str],
    refs: Sequence[str],
    debug: bool = False,
):
    from sacrebleu.metrics import BLEU, CHRF, TER

    if debug:
        print(f"""{hyps[:3]=}""", file=sys.stderr)
        print(f"""{refs[:3]=}""", file=sys.stderr)

    for metric in (BLEU(), CHRF(), TER()):
        score = metric.corpus_score(hyps, [refs])
        print(score, file=sys.stderr)
        print(metric.get_signature(), file=sys.stderr)


@app.command()
def cli_generate(
    input_filename: Path,
    checkpoint="HuggingFaceTB/SmolLM2-135M-Instruct",
    batch_size: int = 128,
    max_new_tokens: int = 150,
    temperature: float = 0.2,
    top_p: float = 0.9,
    do_sample: bool = True,
):
    """
    \b
    srun \\
      --job-name=generate \\
      --account=dt-base \\
      --partition=SynthiaGPU-Preempt \\
      --time=00:20:00 \\
      --nodes=1 \\
      --gpus-per-node=1 \\
      --ntasks-per-node=1 \\
      --cpus-per-task=16 \\
      --mem=64G \\
      python \\
        scripts/generate.py \\
            --checkpoint=saves/SmolLM2-1.7B-Instruct/sft/lora/30595 \\
            data/sft/validation-2026-02.jsonl

    \b
    jq -r .prediction translation.jsonl \\
    | sacrebleu --tokenize=13a <(jq -r .output translation.jsonl) --metric bleu ter chrf
    """
    from functools import partial

    import torch
    from datasets import load_dataset
    from transformers import AutoModelForCausalLM, AutoTokenizer

    """
    <|im_start|>system
    You are a helpful AI assistant named SmolLM, trained by Hugging Face.<|im_end|>
    <|im_start|>user
    Translate the following text from English to French
    Honourable senators, I would like to take a moment to acknowledge a cowardly act of terrorism committed in New York yesterday.<|im_end|>
    <|im_start|>assistant
    Honorables sénateurs, je voudrais prendre un moment pour souligner un acte lâche de terrorisme commis hier à New York.<|im_end|>
    """

    device = (
        "cuda" if torch.cuda.is_available() else "cpu"
    )  # for GPU usage or "cpu" for CPU usage
    tokenizer = AutoTokenizer.from_pretrained(checkpoint, padding_side="left")
    tokenizer.padding_side = (
        "left"  # Makes it easier to strip out the prompt from the prediction.
    )
    # for multiple GPUs install accelerate and do `model = AutoModelForCausalLM.from_pretrained(checkpoint, device_map="auto")`
    model = AutoModelForCausalLM.from_pretrained(checkpoint).to(device)
    model.eval()

    # Prepare the dataset.
    ds = load_dataset("json", data_files=str(input_filename), streaming=False)
    ds = ds.map(
        convert2messages,
        batched=False,
        desc="2messages",
    )
    ds = ds.map(
        lambda ex: apply_chat_template(ex, tokenizer),
        batched=False,
        desc="2chat",
    )

    # Translate the dataset.
    ds = ds.map(
        partial(
            infer_batch,
            tokenizer=tokenizer,
            model=model,
            max_new_tokens=max_new_tokens,
            temperature=temperature,
            top_p=top_p,
            do_sample=do_sample,
        ),
        batched=True,
        batch_size=batch_size,
        # remove_columns=["text"],  # Optional: save memory by dropping raw text
        desc="translating",
    )

    # WARNING: Can't write to sys.stdout
    # ds["train"].to_json(sys.stdout)

    ds = ds["train"]
    for prediction in ds:
        print(json.dumps(prediction, ensure_ascii=False))

    from operator import itemgetter

    metrics(
        hyps=ds["prediction"],
        refs=list(map(itemgetter("output"), ds)),
    )


if __name__ == "__main__":
    app()
```

## preprocess_logits_for_metrics

preprocess*logits_for_metrics (`Callable[[torch.Tensor, torch.Tensor], torch.Tensor]`, \_optional*): W \[1/7]
A function that preprocess the logits right before caching them at each evaluation step.
Must take two tensors, the logits and the labels, and return the logits once processed as desired.
The modifications made by this function will be reflected in the predictions received by `compute_metrics`.

Note that the labels (second parameter) will be `None` if the dataset does not have them.
