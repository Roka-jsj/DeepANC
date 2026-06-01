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
