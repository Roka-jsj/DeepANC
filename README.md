# DeepANC

GCRN-complex 기반 Speech Enhancement / Deep ANC 사전 실험 저장소입니다.

## Docker Image

```bash
docker pull jeongsj/deepanc:speech
Clone
git clone https://github.com/Roka-jsj/DeepANC.git
cd DeepANC
Run Docker
docker run --gpus all -it \
  --name deepanc \
  -v $(pwd):/workspace/DeepANC \
  -v ~/DeepANC/datasets:/workspace/datasets \
  jeongsj/deepanc:speech bash
GPU Check
nvidia-smi
python -c "import torch; print(torch.cuda.is_available())"
Dataset Download
LibriSpeech train-clean-100
mkdir -p ~/DeepANC/datasets/LibriSpeech
cd ~/DeepANC/datasets/LibriSpeech

wget https://www.openslr.org/resources/12/train-clean-100.tar.gz
tar -xzf train-clean-100.tar.gz
DNS-Challenge Noise
mkdir -p ~/DeepANC/datasets
cd ~/DeepANC/datasets

git clone https://github.com/microsoft/DNS-Challenge.git
cd DNS-Challenge

curl -L "https://dnschallengepublic.blob.core.windows.net/dns5archive/V5_training_dataset/noise_fullband/datasets_fullband.noise_fullband.audioset_000.tar.bz2" \
| tar -C "./" -f - -x -j
Data Format

GCRN 학습 데이터는 HDF5 .ex 형식입니다.

*.ex
├── mix  # noisy speech
└── sph  # clean speech
Training
cd /workspace/DeepANC/scripts

python -B ./train.py \
  --gpu_ids=0 \
  --tr_list=../filelists/tr_list_librispeech.txt \
  --cv_file=../data/datasets/cv_librispeech/cv_librispeech.ex \
  --ckpt_dir=exp_librispeech_dns \
  --logging_period=100 \
  --clip_norm=5.0 \
  --lr=0.0005 \
  --time_log=./time.log \
  --unit=utt \
  --batch_size=4 \
  --buffer_size=8 \
  --max_n_epochs=30
Evaluation
python -B ./test.py \
  --gpu_ids=0 \
  --tt_list=../filelists/tt_list_librispeech.txt \
  --ckpt_dir=exp_librispeech_dns \
  --model_file=./exp_librispeech_dns/models/latest.pt
Metrics
python -B ./measure.py --metric=snr  --tt_list=../filelists/tt_list_librispeech.txt --ckpt_dir=exp_librispeech_dns
python -B ./measure.py --metric=stoi --tt_list=../filelists/tt_list_librispeech.txt --ckpt_dir=exp_librispeech_dns
python -B ./measure.py --metric=pesq --tt_list=../filelists/tt_list_librispeech.txt --ckpt_dir=exp_librispeech_dns
Result Example
mix SNR: 6.0834 dB
estimated SNR: 8.9397 dB
SNR improvement: +2.8562 dB
Output Files
*_mix.wav      noisy input
*_sph.wav      clean target
*_sph_est.wav  enhanced output
Notes

대용량 데이터셋과 .pt 모델 파일은 GitHub에 포함하지 않습니다.

DockerHub: 실행 환경
GitHub: 코드 및 사용법
Local checkpoints: 학습된 모델 파일
Local datasets: LibriSpeech, DNS-Challenge


# DeepANC

## Overview

본 프로젝트는 GCRN(Complex-valued Gated Convolutional Recurrent Network)을 기반으로 Speech Enhancement 및 향후 Deep ANC(Active Noise Cancellation) 연구를 수행하기 위한 실험 저장소이다.

학습 환경은 Docker 기반으로 구성하였으며, LibriSpeech와 DNS-Challenge 데이터셋을 활용하여 noisy-clean speech pair를 생성한 후 GCRN을 학습하였다.

---

## Docker Image

DockerHub:

```bash
docker pull jeongsj/deepanc:speech
```

DockerHub Repository:

```text
jeongsj/deepanc:speech
```

---

## Repository Clone

```bash
git clone https://github.com/Roka-jsj/DeepANC.git
cd DeepANC
```

---

## Experimental Environment

### Hardware

* CPU: AMD Ryzen
* GPU: NVIDIA RTX 3080 Ti
* RAM: 64 GB

### Software

* Ubuntu 22.04
* Docker
* PyTorch 1.9.0
* CUDA 11.1

---

## Dataset

### Clean Speech

#### LibriSpeech

Dataset:

https://www.openslr.org/12

Downloaded Dataset:

```text
train-clean-100
```

Number of files:

```text
28,539 FLAC files
```

Total duration:

```text
100 hours
```

---

### Noise Dataset

#### DNS-Challenge

Repository:

https://github.com/microsoft/DNS-Challenge

Used Dataset:

```text
noise_fullband
```

---

## Data Generation

### Preprocessing

* Convert to 16 kHz
* Convert to mono channel
* Normalize RMS

### Noisy-Clean Pair Generation

```text
clean speech
+
noise
↓
mix
```

Random SNR:

```text
-5 dB ~ 15 dB
```

---

## HDF5 Format

GCRN input format:

```text
*.ex
├── mix
└── sph
```

### Description

mix:

```text
Noisy Speech
```

sph:

```text
Clean Speech
```

---

## Training Configuration

### Dataset

Generated Pairs:

```text
10000 pairs
```

Train:

```text
9000 pairs
```

Validation:

```text
1000 pairs
```

---

### Hyperparameters

```text
batch_size = 4
buffer_size = 8
learning_rate = 0.0005
max_n_epochs = 30
clip_norm = 5.0
segment_size = 4 sec
segment_shift = 1 sec
```

---

## Training Command

```bash
python -B ./train.py \
  --gpu_ids=0 \
  --tr_list=../filelists/tr_list_librispeech.txt \
  --cv_file=../data/datasets/cv_librispeech/cv_librispeech.ex \
  --ckpt_dir=exp_librispeech_dns \
  --logging_period=100 \
  --clip_norm=5.0 \
  --lr=0.0005 \
  --time_log=./time.log \
  --unit=utt \
  --batch_size=4 \
  --buffer_size=8 \
  --max_n_epochs=30
```

---

## Evaluation

### Test

```bash
python -B ./test.py \
  --gpu_ids=0 \
  --tt_list=../filelists/tt_list_librispeech.txt \
  --ckpt_dir=exp_librispeech_dns \
  --model_file=./exp_librispeech_dns/models/latest.pt
```

---

### Metrics

#### SNR

```bash
python -B ./measure.py --metric=snr
```

#### STOI

```bash
python -B ./measure.py --metric=stoi
```

#### PESQ

```bash
python -B ./measure.py --metric=pesq
```

---

## Experimental Results

### SNR

Input:

```text
6.0834 dB
```

Output:

```text
8.9397 dB
```

Improvement:

```text
+2.8562 dB
```

---

## Output Files

```text
*_mix.wav
```

Noisy input

```text
*_sph.wav
```

Clean target

```text
*_sph_est.wav
```

Model output

---

## Project Structure

```text
DeepANC
├── scripts
├── docs
├── README.md
└── .gitignore
```

---

## Future Work

* LibriSpeech + DNS Full Dataset
* Real-time Streaming Inference
* Jetson AGX Orin Deployment
* Deep ANC
* End-to-End Noise Cancellation

```
```

