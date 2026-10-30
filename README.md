# RFID-ExSim 데이터 탐구 프로젝트

고등학생 데이터 기반 자율탐구 프로젝트를 위한 **RFID-ExSim 데이터셋 분석·시각화 노트북**입니다.

| 파일 | 내용 |
|---|---|
| `rfid_analysis.ipynb` | 데이터 불러오기 → 탐색(EDA) → 시나리오별 분석(S1~S5) → 종합 비교 → 결정 트리 모델 |
| `보고서_작성_가이드.md` | 「SIMs 데이터 기반 자율탐구 보고서」 양식의 각 칸에 노트북의 어떤 결과를 넣을지 정리 |

## 1. 데이터 받기
원본: <https://github.com/yasserhmimou9/RFID-ExSim-Dataset/tree/v1.0> (MIT License)

GitHub 저장소에는 설명서만 있고, **실제 데이터는 README의 OneDrive 링크**에서 내려받아야 합니다.
압축을 풀면 다음과 같은 구조가 나옵니다.

```
rfid_dataset/
└─ data/processed/
   ├─ rfid_dataset_S1_all.jsonl   ← 노트북이 읽는 파일
   ├─ ...
   └─ rfid_dataset_S5_all.jsonl
```

## 2. 실행하기

**Google Colab (설치 없이, 추천)**
1. <https://colab.research.google.com> → `파일 > 노트북 업로드` → `rfid_analysis.ipynb`
2. 왼쪽 폴더 아이콘에서 `rfid_dataset` 폴더(또는 zip)를 업로드합니다. zip이면 `!unzip 파일명.zip`으로 압축을 풉니다.
3. `DATA_DIR = Path("/content/rfid_dataset")`처럼 경로를 맞추고 `런타임 > 모두 실행`을 누릅니다.

**내 컴퓨터 (Jupyter)**
```bash
pip install pandas matplotlib scikit-learn koreanize-matplotlib jupyter
jupyter notebook rfid_analysis.ipynb
```

## 3. 노트북에서 다루는 탐구 질문

| 절 | 질문 | 보고서 칸 |
|---|---|---|
| S1 | 태그와의 거리·각도가 초당 읽기 수에 영향을 줄까? | Ⅲ-2, Ⅲ-3 |
| S2 | 두 리더기가 동시에 읽을 때 시간차와 점유율은? | Ⅲ-2, Ⅲ-4 |
| S3 | 같은 UID가 여러 실물 태그에서 나오면 복제를 찾아낼 수 있을까? | Ⅲ-4 |
| S4 | 재전송 공격은 정상과 읽기 간격 분포가 다를까? | Ⅲ-2 |
| S5 | 폭주 공격 중 초당 읽기 수는 얼마나 치솟나? | Ⅲ-3 |
| 종합 | 어떤 지표가 공격을 가장 잘 구별할까? | Ⅲ-2, Ⅳ-1 |
| 심화 | 세션 요약 지표로 시나리오를 맞히는 결정 트리 | Ⅲ-3, Ⅴ-1 |

> 노트북은 데이터셋 README에 적힌 열 구조(`timestamp_iso`, `session_id`, `scenario`, `device_id`, `tag_local_id`, `uid_hash`, `distance_cm`, `orientation_deg`, `event`, `raw_payload`)를 기준으로 작성되었습니다.
> 실제 파일에서 `event` 값의 종류나 빈 값이 어떻게 나오는지는 **2절 출력**에서 꼭 확인하고 보고서에 적으세요.
