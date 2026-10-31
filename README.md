# RFID-ExSim 데이터 자료집

2026 SIMs 1학년 데이터 기반 자율탐구를 위한 **RFID-ExSim 데이터셋 설명과 기초 분석 자료**입니다.
이 문서와 자료집에는 해석이나 결론을 넣지 않았습니다. 무엇이 흥미로운지, 무엇을 더 파고들지는 직접 판단하세요.

| 파일 | 내용 |
|---|---|
| `README.md` | 데이터셋 전체 설명(1부)과 분석 방법(2부) |
| `데이터셋_설명.md` | RFID 개념과 용어를 고1 수준으로 풀어 쓴 설명 |
| `RFID_자료집.ipynb` | S1~S5 기초 분석과 그래프 (실행 결과 포함, GitHub에서 바로 볼 수 있음) |
| `자료집_그래프/` | 자료집의 그래프 14장 (PNG) |

---

# 1부. 데이터셋 설명

## 1-1. 개요

| 항목 | 내용 |
|---|---|
| 이름 | RFID-ExSim (RFID Experimental Simulation Dataset) v1.0 |
| 만든 사람 | Yasser Hmimou (ENSET-2IACS / EMSI-LPRI) |
| 출처 | <https://github.com/yasserhmimou9/RFID-ExSim-Dataset/tree/v1.0> (데이터 파일은 그 README의 OneDrive 링크) |
| 논문 | *RFID-ExSim: A multi-scenario experimental dataset for collision timing, tag cloning, replay injection, and flooding stress in passive RFID systems* ([PMC12936520](https://pmc.ncbi.nlm.nih.gov/articles/PMC12936520/)) |
| 라이선스 | MIT (자유롭게 사용 가능, 출처 표시) |
| 목적 | 정상적인 RFID 읽기와 여러 공격·이상 상황을 기록해, RFID 보안과 이상 탐지 연구에 쓰도록 함 |
| 규모 | 기록 **408,422줄**, 시나리오 5개, 세션 396개 |
| 측정 기간 | 2025년 11월 4일 ~ 11월 11일 (UTC) |

## 1-2. 실험 장비

| 장비 | 설명 |
|---|---|
| 리더기 2대 | ESP32 보드 + MFRC522 RFID 모듈(13.56MHz). 이름은 `ESP32_A`, `ESP32_B` |
| 태그 12개 | 배터리 없는 수동형 RFID 카드. 이름은 `TAG01` ~ `TAG12` (`tags.csv`) |
| 기록 방식 | 리더기가 컴퓨터에 USB(시리얼 포트 `COM3`, `COM4`, `COM8`, `COM9`)로 연결되어, 읽을 때마다 한 줄씩 저장 |
| 배선 | `firmware/esp32_reader/notes_wiring.md`에 리더기별 핀 연결표 |

## 1-3. 다섯 가지 시나리오

공식 문서(`docs/scenarios.md`)의 설명과, 실제 파일에서 확인한 값입니다.

| 시나리오 | 공식 설명 | 기록 수 | 세션 수 | 거리(cm) | 각도(°) | 기록 종류(event) | 측정일 |
|---|---|---|---|---|---|---|---|
| **S1** 정상 읽기 | 태그를 여러 거리·각도에서 평범하게 읽음 (조건마다 100~150회) | 13,004 | 108 | 1, 3, 4 | 0, 45, 90 | read_success 12,472 / raw 532 | 11/04~11/09 |
| **S2** 동시 읽기·충돌 | 리더기 A·B를 동시에 켜고 같은 태그를 읽게 함 (태그마다 150회) | 31,776 | 108 | 1, 3, 4 | 0, 45, 90 | read_success 31,776 | 11/10 |
| **S3** 태그 복제 | 같은 UID를 가진 태그 쌍이 함께 있을 때를 관찰 (쌍마다 3회) | 32,400 | 108 | 1, 2, 3 | 0, 45, 90 | read_success 32,400 | 11/11 |
| **S4** 재전송·주입 | 리더 A가 정상 태그를 읽는 동안 같은 UID를 복제 태그·휴대폰 에뮬레이션·소프트웨어 주입으로 다시 넣음 | 41,719 | 36 | 1, 2, 3 | 0, 45, 90 | read_success 31,050 / read_injected 10,258 / read_fail 411 | 11/10 |
| **S5** 폭주(DoS 유사) | 짧은 시간에 대량의 읽기 시도(버스트 1,000~5,000회)를 반복 | 289,523 | 36 | 기록 파일에 없음 (계획표에 1, 2, 3) | 기록 파일에 없음 (계획표에 0, 45, 90) | read_success 289,523 | 11/11 |

## 1-4. 한 줄(기록 1건)의 구조

기록 파일은 **JSONL** 형식이고, 한 줄이 리더기 기록 1건입니다. S1 파일의 첫 줄:

```json
{"timestamp_iso": "2025-11-04T08:12:55.469573Z", "session_id": "S20251104_S1_TAG01_d1_o0_run1", "scenario": "S1",
 "device_id": "ESP32_B", "port": "COM4", "tag_local_id": "TAG01", "uid_hash": "9720a7f2733d", "uid_plain_local": "83C11D13",
 "distance_cm": 1, "orientation_deg": 0, "event": "read_success", "raw_payload": "UID: 83 C1 1D 13"}
```

| 열 | 의미 | 유형 | 단위·값 | 있는 시나리오 |
|---|---|---|---|---|
| `timestamp_iso` | 기록 시각 (세계 표준시) | 시간 | 100만분의 1초까지 | 전체 |
| `session_id` | 실험 회차 이름 (예: 날짜_시나리오_태그_거리_각도_회차) | 범주 | – | 전체 |
| `scenario` | 시나리오 | 범주 | S1~S5 | 전체 |
| `device_id` | 기록한 리더기 | 범주 | ESP32_A, ESP32_B | 전체 |
| `port` | 리더기가 연결된 포트 | 범주 | COM3, COM4, COM8, COM9 | 전체 |
| `tag_local_id` | 실물 태그 이름 | 범주 | TAG01~TAG12 | 전체 (S1은 raw 줄에서 비어 있음) |
| `uid_hash` | 태그 UID를 해시로 바꾼 값 | 범주 | 문자열 | S1~S4 |
| `uid_plain_local` | 태그 UID 원본 | 범주 | 16진수 문자열 | 전체 |
| `distance_cm` | 태그–리더기 거리 | 수치 | cm | S1~S4 |
| `orientation_deg` | 태그 각도 | 수치 | ° | S1~S4 |
| `event` | 기록 종류 | 범주 | read_success, read_fail, read_injected, raw | 전체 |
| `raw_payload` | 리더 모듈이 보낸 원본 메시지 | 문자 | – | 전체 |
| `reader`, `source_file` | 리더 표시, 원래 파일 이름 | 범주 | – | S4 |

## 1-5. 폴더 구조

```
rfid_dataset/
├─ data/
│  ├─ processed/   시나리오별 기록(rfid_dataset_S1~S5_all.jsonl), 요약 파일, 연구자 그래프
│  ├─ plans/       실험 계획표 s1_plan.csv ~ s5_plan.csv
│  ├─ raw/         가공 전 원본 기록 (세션·리더기별 파일 937개)
│  └─ calibration/ 장비 보정 기록 (S1, TAG01)
├─ docs/           methodology.md (수집 방법), scenarios.md (시나리오 정의)
├─ figures/        연구자가 만든 S3~S5 그래프
├─ firmware/       리더기 배선 메모
└─ tags.csv        태그 12개 목록
```

| 파일 | 내용 | 한 행의 의미 |
|---|---|---|
| `meta_sessions_S1.csv` | S1 세션별 기록 수·성공 수·성공률·조건 | 세션 1개 (108행) |
| `meta_sessions_S2.csv` | S2 리더기·거리·각도별 기록 수·신호 세기·성공률 | 조건 1개 (18행) |
| `s2_collisions_summary.csv` | 두 리더 기록 시각 쌍과 그 차이(`dt`, 초) | 기록 쌍 1개 |
| `s3_summary_counts.csv` | 태그·리더기별 기록 수 | 태그 × 리더기 |
| `s3_clone_collisions_summary.csv` | 같은 UID의 두 리더 기록 시각과 차이(`delta_ms`) | 기록 쌍 1개 |
| `s4_replay_summary.csv` | 재전송 지연(`delta_ms`) | 기록 1개 |
| `s4_merge_summary.json` | S4 원본 파일별 기록 수 | – |
| `s5_success_rates_per_session.csv` | S5 세션별 A 기록 수, B 기록 수, 성공률 | 세션 1개 (36행) |
| `plans/s?_plan.csv` | 세션마다 정한 실험 조건 (거리, 각도, 반복 수, 재전송 방식, 요청 속도 등) | 세션 또는 태그 쌍 1개 |

---

# 2부. 분석 방법

`RFID_자료집.ipynb`에서 한 작업을 순서대로 적었습니다.

## 2-1. 도구
- Google Colab, Python
- pandas (표 계산), NumPy (수치 계산), matplotlib (그래프), koreanize-matplotlib (그래프 한글 글꼴)

## 2-2. 데이터 불러오기와 전처리
1. **불러오기**: `data/processed/rfid_dataset_S1_all.jsonl` ~ `S5_all.jsonl` 다섯 파일을 각각 표로 읽었습니다.
2. **시간 변환**: `timestamp_iso` 문자열을 세계 표준시(UTC) 날짜·시간 형식으로 바꾸고, 시간 순으로 정렬했습니다.
3. **파생 변수, 기록 간격**: 같은 세션(`session_id`)·같은 리더기(`device_id`) 안에서 **직전 기록과의 시간 차이(초)**를 계산해 `간격_초` 열을 만들었습니다. 모든 기록 종류(event)를 포함해 계산했습니다.
4. **계획표 읽기**: `s5_plan.csv`는 마지막 `notes` 칸 안에 쉼표가 있어 일반 방식으로 읽으면 칸이 밀립니다. 그래서 앞쪽 칸은 그대로 두고 남는 부분을 마지막 칸에 합쳐서 읽었습니다. (다른 계획표·요약 파일도 같은 방식으로 읽음)
5. **결측치 확인**: 열마다 값이 채워진 비율(%)을 시나리오별로 계산했습니다. 값을 지우거나 채워 넣는 처리는 하지 않았습니다.

## 2-3. 성공률 계산
시나리오마다 성공률을 구할 수 있는 자료가 달라서 아래처럼 각각 계산했습니다.

| 시나리오 | 계산 방법 | 사용한 파일 |
|---|---|---|
| S1 | 세션별 `success_rate`를 같은 거리·각도끼리 평균 | `meta_sessions_S1.csv` |
| S2 | 리더기별 `success_rate`를 같은 거리·각도끼리 평균 | `meta_sessions_S2.csv` |
| S3 | 전체 기록 중 `event = read_success` 인 비율 | 기록 파일 |
| S4 | 전체 기록 중 `event ≠ read_fail` 인 비율 | 기록 파일 |
| S5 | 세션별 `success_rate_pct`를 같은 거리·각도끼리 평균 (거리·각도는 `session_id`로 계획표와 연결해 가져옴) | `s5_success_rates_per_session.csv` + `s5_plan.csv` |

## 2-4. 그래프

모든 시나리오에 같은 방법을 적용했고, 시나리오마다 색을 고정했습니다 (S1 파랑, S2 주황, S3 초록, S4 노랑, S5 분홍).

| 그래프 | 만든 방법 | 파일 |
|---|---|---|
| 거리·각도별 성공률 비교 | 2-3의 성공률을 시나리오별 작은 그래프 5개로 나란히. x축 각도, 선 색 거리 | `비교_거리각도별_성공률.png` |
| 시나리오별 기록 간격 비교 | `간격_초`의 상자그림. 세로축 로그 눈금, 극단값은 표시하지 않음 | `비교_기록간격.png` |
| 기록 간격 분포 (S1~S5 각각) | `간격_초` 히스토그램. 가로축 로그 눈금(0.0001~1,000초), 0초 간격 제외 | `S?_기록간격_분포.png` |
| S1 태그별·리더기별 기록 수 | 기록 파일에서 태그 × 리더기 개수를 세어 쌓은 막대 | `S1_태그별_리더기별_기록수.png` |
| S2 두 리더 시각 차이 | `s2_collisions_summary.csv`의 `dt`를 ms로 바꿔 히스토그램. ±400ms 밖 값은 양 끝 칸에 모음 | `S2_AB_시간차.png` |
| S3 두 리더 시각 차이 | `s3_clone_collisions_summary.csv`의 `delta_ms` 히스토그램 (0.25ms 간격) | `S3_두리더_시간차.png` |
| S4 방식별 기록 종류 | 기록 파일과 `s4_plan.csv`를 `session_id`로 연결해, 재전송 방식 × 리더기별 event 비율을 쌓은 막대 | `S4_방식별_기록종류.png` |
| S4 재전송 지연 | `s4_replay_summary.csv`의 `delta_ms`를 재전송 방식별 히스토그램으로 (상위 1% 제외 범위) | `S4_재전송지연.png` |
| S5 세션별 성공률 | `s5_success_rates_per_session.csv`의 `success_rate_pct` 막대 | `S5_세션별_성공률.png` |
| S5 시계열 | 한 세션(`S5_TAG01_run1`)의 기록을 0.1초 단위로 세어 초당 값으로 환산, 리더기별 선 | `S5_시계열_S5_TAG01_run1.png` |

각 그래프 아래에는 그래프에 쓰인 숫자를 표(평균, 분위수 등)로 함께 출력했습니다.

## 2-5. 직접 실행하기
1. 출처의 README에 있는 OneDrive 링크에서 데이터 zip(약 6MB)을 받습니다.
2. <https://colab.research.google.com> → `파일 > 노트북 업로드` → `RFID_자료집.ipynb`
3. `런타임 > 모두 실행` → 파일 선택 창에서 zip 선택 (압축은 자동으로 풀림)
4. 코드의 시나리오 이름, 세션 이름(`SID`), 열 이름을 바꾸면 다른 조건도 같은 방식으로 볼 수 있습니다.
