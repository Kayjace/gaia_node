# Gaia GPU 노드 설치 가이드
dharma 도메인 기준. dharma에 참여하려면 https://discord.com/channels/1215232680942374912/1350160366637813773 에 있는 스레드주 andrew.smith3416 (현재 시점 기준 닉네임 Andrew | Toda🌱) 이친구한테 DM 보내서 승인 받아야합니다.

## 지원되는 GPU 모델
- RTX 5090/5080/5070/5070Ti
- RTX 4090/4080/4070/4070Ti/4060/4060Ti
- RTX 3090/3090Ti/3080/3080Ti/3070/3070Ti/3060/3060Ti

**참고:** CUDA 코어가 많고 클럭이 높을수록 Gaia의 성능이 향상됩니다.

## 사용 모델
Qwen2.5-Coder-3B-Instruct-Q5_K_M

## 설치 과정

### 시스템 업데이트, 도구, CUDA 툴킷은 설치되어 있다면 건너뛰셔도 좋습니다.

### 1. 시스템 업데이트
```bash
apt update && apt upgrade -y
apt-get update && apt-get upgrade -y
```

### 2. 필요 도구 설치
```bash
apt install pciutils lsof curl nvtop btop -y
apt-get install jq -y
```

### 3. CUDA 툴킷 설치
```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb
apt-get update
apt-get -y install cuda-toolkit-12-8
```

### 4. Gaia GPU 노드 설치 (Qwen2.5-Coder-3B-Instruct-Q5_K_M)
```bash
lsof -t -i:8101 | xargs kill -9
rm -rf $HOME/gaianet
curl -sSfL 'https://github.com/GaiaNet-AI/gaianet-node/releases/latest/download/uninstall.sh' | bash
mkdir $HOME/gaianet
curl -sSfL 'https://github.com/GaiaNet-AI/gaianet-node/releases/latest/download/install.sh' | bash -s -- --ggmlcuda 12
source $HOME/.bashrc
wget -O "$HOME/gaianet/config.json" https://raw.githubusercontent.com/GaiaNet-AI/node-configs/main/qwen-2.5-coder-7b-instruct_rustlang/config.json
CONFIG_FILE="$HOME/gaianet/config.json"
jq '.chat = "https://huggingface.co/gaianet/Qwen2.5-Coder-3B-Instruct-GGUF/resolve/main/Qwen2.5-Coder-3B-Instruct-Q5_K_M.gguf"' "$CONFIG_FILE" > tmp.$$.json && mv tmp.$$.json "$CONFIG_FILE"
jq '.chat_name = "Qwen2.5-Coder-3B-Instruct"' "$CONFIG_FILE" > tmp.$$.json && mv tmp.$$.json "$CONFIG_FILE"
grep '"chat":' $HOME/gaianet/config.json
grep '"chat_name":' $HOME/gaianet/config.json
gaianet config --port 8101
gaianet init
gaianet start
gaianet info
```

## GPU 활동 모니터링 스크립트 (선택사항)

Gaia GPU 노드를 항상 온라인 상태로 유지하기 위한 스크립트입니다. (Dharma 도메인에서만 사용하세요)

다음 명령어로 스크립트 파일 생성
```bash
nano monitor_gpu_activity.sh
```

스크립트 파일에 복붙할 내용:

```bash
#!/bin/bash

# GPU 사용률 확인 함수
check_gpu_utilization() {
    utilization=$(nvidia-smi --query-gpu=utilization.gpu --format=csv,noheader,nounits)
    echo $utilization
}

# 변수 설정
check_interval=5  # 확인 간격 (초)
inactivity_threshold=30  # 비활성 임계값 (초)
inactivity_duration=0

while true; do
    gpu_util=$(check_gpu_utilization)
    if [ "$gpu_util" -eq 0 ]; then
        inactivity_duration=$((inactivity_duration + check_interval))
        echo "$inactivity_duration"
        if [ "$inactivity_duration" -ge "$inactivity_threshold" ]; then
            # GPU가 임계값 이상 비활성 상태일 때 실행할 명령
            echo "GPU가 1분 이상 비활성 상태입니다."
            echo "Gaia 노드를 재시작합니다."
            gaianet stop
            gaianet start
            gaianet info
            echo "Gaia 노드가 재시작되었습니다."
            inactivity_duration=0  # 비활성 지속 시간 초기화
        fi
    else
        inactivity_duration=0  # GPU가 활성 상태면 초기화
    fi
    sleep $check_interval
done
```

저장 완료되면 스크립트를 실행:
```bash
bash monitor_gpu_activity.sh
```

**참고:** Dharma 에 참여하려면 andrew.smith3416 (현시점 닉네임 Andrew | Toda🌱)에게 DM 보내서 승인 받아야합니다.

---
