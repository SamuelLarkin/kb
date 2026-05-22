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
