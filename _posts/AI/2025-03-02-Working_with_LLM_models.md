---
layout: post
title: Working with LLM model files
tags:
    - AI
author: Radosław Śmigielski
---

Why?
====
1. My own confusion about the LLM model types.
2. Lack of documentation.

Intro
=====

GGUF file format
----------------
GGUF model file format is a binary file 
[Full GGUF format documentation](https://github.com/ggml-org/ggml/blob/master/docs/gguf.md)
[GGUF file format]()


Hugging Face Safetensors files
------------------------------
[Safetensors file format documentation](https://huggingface.co/docs/safetensors/)
[Safetensors file format](https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/safetensors/safetensors-format.svg)

Errors' collection
==================
```
ERROR 03-02 10:01:12 engine.py:400] ValueError: Bfloat16 is only supported on GPUs with compute capability
                                    of at least 8.0. Your NVIDIA GeForce GT 1030 GPU has compute capability 6.1.
                                    You can use float16 instead by explicitly setting the`dtype` flag in CLI,
                                    for example: --dtype=half.
```

```
ValueError: Failed to determine backend: Cannot determine which backend to use:
  The model file /home/ai/.cache/instructlab/models is not a GGUF format
  nor a directory containing huggingface safetensors files.
  Cannot determine which backend to use.
Please use a GGUF file for llama-cpp or a directory containing huggingface safetensors files for vllm.
Note that vLLM is only supported on Linux.
```
