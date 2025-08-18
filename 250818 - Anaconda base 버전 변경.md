기본 base의 python 버전이 3.11이었는데, claude mcp 중 하나를 사용하기 위해서는 3.12가 필요했다.
anaconda 재설치가 필요할줄 알았는데, 아니었다.
아래 단계로 진행하기

1. 아나콘다 환경 업데이트
```bash
conda update -n base conda
conda update --all
python -m pip install --upgrade pip
```

2. 파이썬 버전 바꾸기
```bash
conda search python | grep {원하는 버전}
conda install python={원하는 버전}
conda update --all
```

3. 이미 깔려있던 pip 모듈 재설치 필요!