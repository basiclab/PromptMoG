<div id="top" align="center">

# PromptMoG: Prompt Embedding Mixture-of-Gaussian Sampling

**A plug-and-play framework for enhancing diversity in long-prompt image generation**

Bo-Kai Ruan, Teng-Fang Hsiao, Ling Lo, Yi-Lun Wu, Hong-Han Shuai

<a href="http://arxiv.org/abs/2511.20251"><img src="https://img.shields.io/badge/arXiv-2511.20251-<color>"></a>
<a href="#license">
  <img alt="License: Apache2.0" src="https://img.shields.io/badge/license-Apache%202.0-blue.svg"/>
</a>

</div>

| Qwen-Image |
|:--------:|
| ![Qwen-Image](assets/long_prompt.jpg) |
| **+PromptMoG (Ours)** |
| ![PromptMoG](assets/pmog.jpg) |
| **+CADS** |
| ![CADS](assets/cads.jpg) |
| **+DiverseFlow** |
| ![DiverseFlow](assets/df.jpg) |

## Installation

```bash
uv sync
# load environment
source .venv/bin/activate
```

## Data Preparation

We have provided the filtered dataset in `data/lpd_bench`. Users can also follow the following steps to generate the dataset themselves.

```bash
# Step 1: Generate a pool of long prompts
python misc/dataset_gen/generate_long_prompt.py --num-prompts-for-topic 60

# Step 2: Filter the prompts
python misc/dataset_gen/post_process_data.py --data-root data/lpd_bench --num_prompts_per_topic 60 --num_remain_per_topic 40
```

The outputs will be saved to `data/lpd_bench/filtered`.

## Usage

### Additional Experiments

<details>
<summary>Diversity Test</summary>

```bash
# Generate images
./scripts/gen_image.sh \
    --dataset_type long \
    --prompt_root_dir data/lpd_bench \
    --output_root_dir outputs/long_prompt
./scripts/gen_image.sh \
    --dataset_type short \
    --prompt_root_dir data/lpd_bench \
    --output_root_dir outputs/short_prompt_1 \
    --first_top 1
./scripts/gen_image.sh \
    --dataset_type short \
    --prompt_root_dir data/lpd_bench \
    --output_root_dir outputs/short_prompt_3 \
    --first_top 3

# Score the diversity
./scripts/scoring_diversity.sh --output_root_dir outputs/long_prompt
./scripts/scoring_diversity.sh --output_root_dir outputs/short_prompt_1
./scripts/scoring_diversity.sh --output_root_dir outputs/short_prompt_3
```

</details>

### PromptMoG

```bash
./scripts/gen_image.sh \
    --dataset_type long \
    --model_type pmog \
    --prompt_root_dir data/lpd_bench \
    --output_root_dir outputs/pmog

./scripts/scoring_diversity.sh --output_root_dir outputs/pmog
./scripts/scoring_lpd.sh --output_root_dir outputs/pmog
```

### Ablation Study

| Model | Gamma | Num Mode | Sigma |
| ----- | ----- | -------- | ----- |
| Flux  | 0.6   | 50       | 0.25  |
| Qwen  | 0.85  | 50       | 0.25  |

<details>
<summary>Exploring the gamma</summary>

```bash
GAMMA_LIST=(0.1 0.35 0.6 0.85 1.1)
for gamma in ${GAMMA_LIST[@]}; do
    ./scripts/ablation/gen_and_scoring_ablation.sh \
        --prompt_root_dir data/lpd_bench \
        --output_root_dir outputs/ablation_gamma_flux_${gamma}_qwen_${gamma} \
        --dataset_type long \
        --model_type pmog \
        --partial_num 8 \
        --flux_gamma ${gamma} \
        --qwen_gamma ${gamma}
    ./scripts/scoring_lpd.sh --output_root_dir outputs/ablation_gamma_flux_${gamma}_qwen_${gamma}
done
```

</details>

<details>
<summary>Exploring the sigma</summary>

```bash
# [0, 0.25, 0.5, 0.75, 1.0]
SIGMA_LIST=(0 0.25 0.5 0.75 1.0)
for sigma in ${SIGMA_LIST[@]}; do
    ./scripts/ablation/gen_and_scoring_ablation.sh \
        --prompt_root_dir data/lpd_bench \
        --output_root_dir outputs/ablation_sigma_flux_${sigma}_qwen_${sigma} \
        --dataset_type long \
        --model_type pmog \
        --partial_num 8 \
        --flux_sigma ${sigma} \
        --qwen_sigma ${sigma}
    ./scripts/scoring_lpd.sh --output_root_dir outputs/ablation_sigma_flux_${sigma}_qwen_${sigma}
done
```

</details>

<details>
<summary>Exploring the mode</summary>

```bash
NUM_MODE_LIST=(1 25 50 75 100)
for num_mode in ${NUM_MODE_LIST[@]}; do
    ./scripts/ablation/gen_and_scoring_ablation.sh \
        --prompt_root_dir data/lpd_bench \
        --output_root_dir outputs/ablation_mode_flux_${num_mode}_qwen_${num_mode} \
        --dataset_type long \
        --model_type pmog \
        --partial_num 8 \
        --flux_num_mode ${num_mode} \
        --qwen_num_mode ${num_mode}
    ./scripts/scoring_lpd.sh --output_root_dir outputs/ablation_mode_flux_${num_mode}_qwen_${num_mode}
done
```

</details>

## Citation

```bibtex
@inproceedings{promptmog,
  title={Diversifying Long Prompt Image Generation through Structured Prompt Embedding Space Sampling},
  author={Ruan, Bo-Kai and Hsiao, Teng-Fang and Lo, Ling and Wu, Yi-Lun and Shuai, Hong-Han},
  booktitle={BMVC},
  year={2026}
}
```


