# AI

## Ġ Delimiter

Why are llm tokenizer using Ġ as the marker for a new word?

LLM tokenizers use the special symbol Ġ (Unicode U+0120) to represent a space primarily due to historical implementation choices in Byte-Pair Encoding (BPE) rather than a deliberate linguistic design.

The Byte-Level Mapping Origin

In the development of GPT-2, researchers needed to handle text as raw bytes to ensure the tokenizer could process any character, including non-ASCII ones, without inflating the vocabulary size.
To save the tokenizer’s vocabulary and merge rules in a human-readable text file, they had to map the 256 possible byte values to Unicode characters.

- Problem: Standard bytes like space (byte 32) or control characters (like byte 31) are either invisible, unprintable, or problematic for file parsing (e.g., spaces break the format of merge files).
- Solution: They mapped these problematic bytes to the next available unproblematic Unicode characters starting from codepoint 256.
- Result: Byte 32 (the space character) was mapped to codepoint 288 (256 + 32), which corresponds to the character Ġ.
  Similarly, byte 31 became ğ (codepoint 287).

Why Use Ġ Instead of Removing Spaces?

Using a special marker like Ġ preserves word boundaries within the token stream.

- Word Boundaries: If spaces were simply removed or ignored, the tokenizer might merge words incorrectly (e.g., turning "cat dog" into "catdog").
  By keeping the space as a distinct token (Ġ), the model knows where one word ends and another begins.
- Tokenization Efficiency: This approach allows the model to treat prefixes of words as separate tokens (e.g., "Ġrunning" vs. "running") while maintaining structural integrity.

Contrast with SentencePiece (▁)

It is worth noting that not all tokenizers use Ġ.
The SentencePiece library, used by models like T5 and Llama 2/3 (with some variations), uses a low line underscore (▁) for the same purpose.
This is a hardcoded convention in the SentencePiece package, chosen to visually distinguish word boundaries from regular text without relying on byte-mapping quirks.

## Softmax

The softmax function with temperature $T$ modifies the standard softmax by scaling the logits $z_i$ before exponentiation.
The formula is:

$$P_i = \frac{e^{z_i/T}}{\sum_j e^{z_i/T}}$$

Where:

- $P_i$ is the probability of token $i$.
- $z_i$ is the logit (raw score) for token $i$.
- $T$ is the temperature parameter ($T > 0$).

Effect of $T$:

- $T \to 0$: The distribution becomes peaked, approaching a one-hot vector where the token with the highest logit has probability $1$.
- $T = 1$: The standard softmax function; the original probability distribution is preserved.
- $T \to \infty$: The distribution becomes uniform, where all tokens have equal probability ($1/N$).

This scaling allows precise control over the "sharpness" of the probability distribution during AI inference.

## Temperature

LLM temperature is a hyperparameter that controls the randomness and creativity of text generation during inference by modifying the probability distribution of the model's output tokens.
Technically, it works by dividing the model's raw scores (logits) by the temperature value before applying the softmax function; a value of 1.0 leaves the distribution unchanged, values below 1.0 sharpen the distribution to favor high-probability tokens, and values above 1.0 flatten it to allow more diverse, less probable selections.

Common applications and ranges include:

- Low Temperature (0.0–0.3): Produces deterministic, factual, and consistent outputs, ideal for technical documentation, coding, or factual Q&A where accuracy is paramount.
- Medium Temperature (0.4–0.8): Balances structure and creativity, suitable for conversational AI or general summarization tasks requiring natural language flow.
- High Temperature (0.9–1.5+): Maximizes diversity and unpredictability, used for brainstorming, creative writing, or idea generation, though excessively high values can lead incoherent or "chaotic" results.

Temperature is often used in conjunction with top-k or top-p (nucleus) sampling to further constrain the pool of candidate tokens, ensuring that increased randomness does not compromise the coherence of the generated text.
