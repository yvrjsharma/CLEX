# CLEX: Continuous Length Extrapolation for Large Language Models
This repo provides the official implementation of our paper "CLEX: Continuous Length Extrapolation for Large Language Models"

<div style='display:flex; gap: 0.25rem; '>
<a href='https://huggingface.co/DAMO-NLP-SG/Video-LLaMA-2-7B-Finetuned'><img src='https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Checkpoint-blue'></a> 
<a href='https://huggingface.co/spaces/DAMO-NLP-SG/Video-LLaMA'><img src='https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Demo-blue'></a>
<a href='https://arxiv.org/abs/2306.02858'><img src='https://img.shields.io/badge/Paper-PDF-red'></a>
</div>

## News
- [10.25] ðŸš€ðŸš€ Release the code of **CLEX** and the long-context base & chat models trained with CLEX. 

## Features and Highlights of CLEX
- **Simple and Clear**: _MINIMAL_ code and architecture changes. Extending context windown size of LLMs _WITHOUT_ either recurrent memory caching or sparse attention.
- **Train Short, Test Long**: _NO_ performance drop on the sequences _4x longer_ than the training ones 
- **Continuous Length Extrapolation**: Explicitly modeling the continuous-time growth of context window size

## Model Zoo
<div align="center">

| Model Name | Model Type | Starting Point | Train Data |Train Length | MAX Test Length | HF Repo |
|:-----|:-----|:-----------|:-----------|:-----------|:-----------|:------:|
| CLEX-7B-4K | base | LLaMA-2-7B | [Redpajama-Book](https://huggingface.co/datasets/togethercomputer/RedPajama-Data-1T) | 4K | 16K | [TBD]() |
| CLEX-7B-Chat-4K | chat | CLEX-7B-4K | [UltraChat](https://github.com/thunlp/UltraChat) | 4K | 16K | [TBD]() |
| CLEX-7B-16K | base | LLaMA-2-7B | [Redpajama-Book](https://huggingface.co/datasets/togethercomputer/RedPajama-Data-1T) | 16K | 64K | [TBD]() |
| CLEX-7B-Chat-16K | chat | CLEX-7B-16K | [UltraChat](https://github.com/thunlp/UltraChat) | 16K | 64K | [TBD]() |
</div>

## Supported LLMs
- [x] LLaMA-2
- [ ] Falcon
- [ ] GPT-NeoX
- [ ] QWen



## Usage

### Environment Setup
```bash
conda create -yn clex python=3.9
conda activate clex

git clone https://github.com/DAMO-NLP-SG/CLEX.git
cd CLEX
pip install -r requirements.txt
# install flash-attn separately
pip install flash-attn==2.0.8 --no-build-isolation
```

### Code Snippet for Minimal Usage

```bash
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

tokenizer = AutoTokenizer.from_pretrained("DAMO-NLP-SG/CLEX", trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained("DAMO-NLP-SG/CLEX", torch_dtype=torch.bfloat16)
inputs = tokenizer("What is CLEX?", return_tensors="pt")
sample = model.generate(**inputs, max_length=128)
print(tokenizer.decode(sample[0]))
```

### Inference with Command Line Interface
We replicate the command line interface of [FastChat](https://github.com/lm-sys/FastChat) here.
You can use the command below to to enable the streaming chatting upon CLEX. The CLEX-7b-chat-4k supports the input sequence lengths up to 16k. 
```bash
python3 serve/cli.py --model-path DAMO-NLP-SG/CLEX --num-gpu 1
```
You can also try our web GUI demo [here]().



## Training
To customize the long-context LLaMA-2 with CLEX on your own data, run the script `scripts/train_lm.sh` as follows:
```bash
./scripts/train_lm.sh
```
For training the chat model, run the script `scripts/train_chat.sh` instead.

## Evaluation
### Language Modelling
Here is the evaluation PPLs of the base models trained with CLEX. We apply training and evaluation on a subset of 2B tokens from the [RedPajama-Book](https://github.com/togethercomputer/RedPajama-Data) corpus, where the training and test sets are splitted by 99:1.

| Models | Train Length | Eval.(4k) | Eval.(8k) | Eval.(16k) | Eval.(32k) | Eval.(64k) |
| --- | --- | --- | --- | --- | --- | --- |
| LLaMA-2-7B | 4k  | 6.04 | 20.54 | >100 | >1000 | >1000 |
| CodeLLaMA-7B | 16k | 7.6 | 7.4 | 7.33 | 15.12 | 52.02 |
| Naive FT | 16k | 5.98 | 5.93 | 5.91 | 18.31 | > 100 |
| PI  | 16k | 5.9 | 5.71 | 5.72 | 6.05 | 8.75 |
| Yarn (s=16) | 16k | 6.5 | 5.71 | 5.73 | 5.99 | 8.51 |
| Yarn (s=32) | 16k | 6.61 | 5.94 | 5.96 | 6.08 | 6.22 |
| CL-Scaling | 16k | 24.99 | 5.86 | 5.87 | 10.56 | 41.09 |
| ALIBI | 4k  | 6.34 | 6.39 | 6.41 | 6.5 | 6.51 |
| RandomPos | 4k  | 5.88 | >100 | >1000 | >1000 | >1000 |
| CLEX-7B-4k | 4k  | **5.86** | 5.7 | 5.87 | 14.53 | 30.51 |
| CLEX-7B-16k | 16k | 5.88 | **5.68** | **5.52** | **5.55** | **5.64** |


### LongBench

We evaluate the chat models trained with CLEX on the [LongBench](https://github.com/THUDM/LongBench), where the average length of most tasks ranging from 5k to 16k. Most results of baselines are taken from the original repo. * denotes that it needs to truncate the input sequence within the train length.

| Model              | Train Length | Avg.  | Single-Document QA | Multi-Document QA | Summarization | Few-shot Learning | Sythetic Task | Code Completion |
| ------------------ | ------------ | ----- | ------------------ | ----------------- | ------------- | ----------------- | ------------- | --------------- |
| GPT-3.5-Turbo      | 16k?         | 44.66 | 45.1               | 36.23             | 23.9          | 57.58             | 51            | 54.15           |
| CodeLLaMA-7B       | 16k          | 33.42 | 32.19              | 21.49             | 20.06         | 57.73             | 8.92          | 60.11           |
| Vicuna-v1.5-7B     | 16k          | 30.54 | 31.75              | 18.8              | 23.25         | 56.83             | 5.33          | 47.25           |
| LongChat-v1.5-7B   | 32k          | 31.59 | 28.78              | 20.33             | 22.45         | 50.8              | 13.03         | 54.15           |
| XGen-7B (*)        | 8k           | 24.96 | 22.15              | 18.02             | 19.05         | 47.23             | 4.7           | 38.6            |
| InternLM-7B (*)    | 8k           | 22.64 | 21.45              | 17.9              | 15.2          | 41.55             | 3.3           | 36.45           |
| Llama2-7B-chat (*) | 4k           | 26.76 | 21.65              | 18.2              | 18.53         | 49.95             | 4.13          | 48.1            |
| Baichuan-13B (ALiBi)       | 4k           | 13.49 | 18.36              | 6.79              | 9.93          | 11.72             | 1.85          | 32.28           |
| ALiBi-7B-4k        | 4k           | 9.93  | 7.23               | 5.98              | 7.4           | 5.69              | 0.67          | 32.61           |
| CLEX-7B-Chat-4k         | 4k           | 32.72 | 29.38              | 20.08             | 23.25         | 56.02             | 9.67          | 57.94           |



## Citation
If you find our project useful, hope you can star our repo and cite our paper as follows:
```
@article{damonlpsg2023clex,
  author = {Chen, Guanzheng and Li, Xin and Meng, Zaiqiao and Liang, Shangsong and Bing, Lidong},
  title = {CLEX: Continuous Length Extrapolation for Large Language Models},
  year = 2023,
  journal = {arXiv preprint arXiv:xxx},
  url = {https://arxiv.org/abs/xxx}
}
```
