<img src="./diagram.png" width="450px"></img>

## Self-Rewarding Language Model (wip)

Implementation of the training framework proposed in <a href="https://arxiv.org/abs/2401.10020">Self-Rewarding Language Model</a>, from MetaAI

They really took the <a href="https://arxiv.org/abs/2305.18290">title of the DPO paper</a> to heart.

May generalize the framework so one can add <a href="https://arxiv.org/abs/2401.01335v1">SPIN</a> as well.

## Appreciation

- <a href="https://a16z.com/supporting-the-open-source-ai-community/">A16Z Open Source AI Grant Program</a> and <a href="https://huggingface.co/">🤗 Huggingface</a> for the generous sponsorships, as well as my other sponsors, for affording me the independence to open source current artificial intelligence research

## Install

``` bash
$ pip install self-rewarding-lm-pytorch
```

## Usage

```python
import torch
from torch import Tensor

from self_rewarding_lm_pytorch import (
    SelfRewardingTrainer,
    create_mock_dataset
)

from x_transformers import TransformerWrapper, Decoder

transformer = TransformerWrapper(
    num_tokens = 256,
    max_seq_len = 1024,
    attn_layers = Decoder(
        dim = 512,
        depth = 1,
        heads = 8
    )
)

SFTDataset = create_mock_dataset(100, lambda: (torch.randint(0, 256, (256,)), torch.tensor(1)))
PromptDataset = create_mock_dataset(100, lambda: 'mock prompt')

def decode_tokens(tokens: Tensor) -> str:
    decode_token = lambda token: str(chr(max(32, token)))
    return ''.join(list(map(decode_token, tokens)))

def encode_str(seq_str: str) -> Tensor:
    return Tensor(list(map(ord, seq_str)))

trainer = SelfRewardingTrainer(
    transformer,
    train_sft_dataset = SFTDataset(),
    spin = False,
    num_preference_pairs = [1, 1],
    preference_max_seq_len = 64,
    prompt_dataset = PromptDataset(),
    tokenizer_encode = encode_str,
    tokenizer_decode = decode_tokens,
    accelerate_kwargs = dict(
        cpu = True
    ),
    dpo_trainer_kwargs = dict(
        batch_size = 1
    )
)

trainer(overwrite_checkpoints = True)
```

## Todo

- [x] generalize the sampling so that it can progress at different positions in the batch, fix all sampling to be batched. also allow for left padded sequences, in the case some people have transformers with relative positions that allow for that
- [x] handle eos

- [ ] remove early stopper in favor of just simple few line logic - have a function that accepts List[float] and decide what to do
- [ ] figure out how best to handle different impl of kv cache, for now just do without
- [ ] allow for different strategies for sampling the pairs
- [ ] consider KTO
- [ ] any order of sft, spin, self-rewarding dpo, dpo with external reward model
- [ ] show an example for using your own reward prompt instead of default llm-as-judge

## Citation

```bibtex
@misc{yuan2024selfrewarding,
    title   = {Self-Rewarding Language Models}, 
    author  = {Weizhe Yuan and Richard Yuanzhe Pang and Kyunghyun Cho and Sainbayar Sukhbaatar and Jing Xu and Jason Weston},
    year    = {2024},
    eprint  = {2401.10020},
    archivePrefix = {arXiv},
    primaryClass = {cs.CL}
}
```

```bibtex
@article{Chen2024SelfPlayFC,
    title   = {Self-Play Fine-Tuning Converts Weak Language Models to Strong Language Models},
    author  = {Zixiang Chen and Yihe Deng and Huizhuo Yuan and Kaixuan Ji and Quanquan Gu},
    journal = {ArXiv},
    year    = {2024},
    volume  = {abs/2401.01335},
    url     = {https://api.semanticscholar.org/CorpusID:266725672}
}
```

```bibtex
@article{Rafailov2023DirectPO,
    title   = {Direct Preference Optimization: Your Language Model is Secretly a Reward Model},
    author  = {Rafael Rafailov and Archit Sharma and Eric Mitchell and Stefano Ermon and Christopher D. Manning and Chelsea Finn},
    journal = {ArXiv},
    year    = {2023},
    volume  = {abs/2305.18290},
    url     = {https://api.semanticscholar.org/CorpusID:258959321}
}
```
