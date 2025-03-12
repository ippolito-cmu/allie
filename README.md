# Allie the Chess Bot

Allie is a GPT2-like chess model that learns chess from human gameplay.
It is deployed on [Lichess](https://lichess.org/@/AllieTheChessBot).

Contains data and code for the paper
> **[Human-Aligned Chess With a Bit of Search](https://arxiv.org/abs/2410.03893)**  
> Yiming Zhang, Athul Paul Jacob, Vivian Lai, Daniel Fried, Daphne Ippolito  
> ICLR 2025

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
  - [Training Allie from Scratch](#training-allie-from-scratch)
  - [Evaluation](#evaluation)
- [Model Weights](#model-weights)
- [License](#license)

## Requirements

- Python 3.11
- 8 GPUs for training (adjustable in configuration)
- 1 GPU for evaluation
- Approximately 60GB of storage space for datasets and model weights

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/y0mingzhang/allie.git
   cd allie
   ```

2. Install the package and its dependencies:
   ```bash
   pip install -e .
   ```

## Usage

### Training Allie from Scratch

1. Download the training and evaluation datasets from [Hugging Face](https://huggingface.co/datasets/yimingzhang/allie-data).\
Note: Try `HUB_ENABLE_HF_TRANSFER=1 huggingface-cli download --repo-type dataset --local-dir data  "yimingzhang/allie-data"`.

2. Place the downloaded content in a local directory named `data/`.

3. Launch training (approximately 2 weeks on 8x A6000 GPUs):
   ```bash
   python src/modeling/main.py pretrain_config/medium.yaml
   ```

   Note: You can adjust gradient accumulation in the configuration file if training on fewer than 8 GPUs.

### Evaluation

1. Download the Allie model weights from [Hugging Face](https://huggingface.co/datasets/yimingzhang/allie-models).

2. Move the content to the `models/` directory in your local repository.\
Note: Try "HUB_ENABLE_HF_TRANSFER=1 huggingface-cli download --repo-type dataset --local-dir models  "yimingzhang/allie-models".

3. Run evaluations:

   - For *Allie-Policy*:
     ```bash
     python src/evaluation/evaluate.py \
       --config pretrain_config/medium.yaml \
       --dataset data/lichess-2022-blitz-test/2022-test-annotated.jsonl \
       --decode policy \
       --output_file "allie-eval/allie-policy.json" \
       --quick
     ```

   - For *Allie-Adaptive-Search*, the time-adaptive MCTS variant:
     ```bash
     python src/evaluation/evaluate.py \
       --config pretrain_config/medium.yaml \
       --dataset data/lichess-2022-blitz-test/2022-test-annotated.jsonl \
       --decode adaptive-mcts \
       --output_file "allie-eval/allie-adaptive-search.json" \
       --quick
     ```
     Note: Right now I don't have kv caching implemented in MCTS, and it runs extremely slowly.

   - For other supported inference algorithms (e.g., standard MCTS) and models, refer to `DECODE_STRATEGIES` in `src/evaluation/decode.py`.

## Model Weights

- The `medium` folder contains a GPT2-medium-like Allie model, which we use for evaluations in the paper.
- The `ablations` folder contains models trained for the ablation study in our paper, including:
  - Double parameters (`large`)
  - Half parameters (`small`)
  - Half compute (`short`)
  - Half training data (`half`)

## License

MIT - see [LICENSE](LICENSE).
