# WORKFLOW_GUIDE

> GitHub에는 실행에 필요한 코드·공유 설정·의존성·필수 고정 리소스·최소 문서만 올린다.
> CSV, figures, 데이터, 학습 가중치, 실험 결과, 원고, 로그는 로컬 또는 별도 저장소에서 관리한다.

- 작성·수정: 2026-11-04 (Asia/Seoul)
- 버전: 1.0
- 적용 범위: 개인 GitHub 프로젝트의 저장소 구성, 연구·대회 코드 구조 정리, 업로드와 유지·보수
- 이 문서는 구조를 바꾸는 기준이다. 문서 등록만으로 로컬 폴더 이동이나 연구 코드 업로드가 수행되지는 않는다.

## 1. 저장소와 로컬 분류 폴더

독립적으로 설치·실행·버전·공개 범위를 관리할 프로젝트마다 저장소를 만든다.
연구는 기본적으로 논문·독립 연구별, 대회는 대회·참가 회차별로 나눈다.

```text
로컬 작업공간/                         # 이 상위 폴더는 Git 저장소가 아님
├── research/
│   ├── ida-steganalysis/             # 독립 Git 저장소
│   └── jpeg-aware-steganalysis/      # 독립 Git 저장소
├── competitions/
│   └── <competition-slug>/           # 독립 Git 저장소
├── apps/
│   └── <app-slug>/
├── tools/
│   └── <tool-slug>/
└── forks/
    └── <upstream-name>/
```

```text
GitHub 계정/                          # 아래 항목은 각각 별도 저장소
├── ida-steganalysis
├── jpeg-aware-steganalysis
├── <competition-slug>
├── <app-slug>
├── <tool-slug>
└── <upstream-fork>
```

- 로컬 폴더명과 원격 저장소명은 달라도 된다. 기존 로컬 폴더명을 바꾸는 일은 별도로 판단한다.
- 같은 연구의 모델·baseline·ablation은 같은 저장소의 모듈과 설정으로 관리한다.
- 같은 대회의 라운드는 같은 저장소의 설정으로 관리하고, 독립적인 대회·회차는 분리한다.
- 학회판·저널 확장판은 코드와 실험을 공유하는 정도에 따라 같은 저장소와 태그로 관리할 수 있다.
- 별도 앱·도구·라이브러리는 연구 저장소에 섞지 않는다.
- 저장소명은 소문자·숫자·하이픈을 사용한다. 대회 연도·회차처럼 식별에 필요한 정보만 이름에 포함한다.
- 외부 코드 수정은 Fork 또는 출처·기준 commit·수정 이력을 추적할 수 있는 방식으로 관리한다.
- 기존 저장소·대회 코드는 실제 작업 요청이 있을 때 개별적으로 정리한다. 이 가이드가 일괄 변경을 뜻하지 않는다.

## 2. 연구·ML 코드의 공통 목표 구조

새 연구와 구조를 정리하는 연구는 다음 역할과 경로를 공통 기준으로 사용한다.
사용하지 않는 빈 폴더는 만들지 않는다. 아래 명령·파일명은 목표 양식이며 실제 구현과 README를 함께 맞춘다.

```text
<repository>/
├── README.md                         # 목적, 환경, 설치, 자료 준비, 실행, 검증
├── AGENTS.md                         # 해당 프로젝트의 작업·검증 지침
├── .gitignore
├── pyproject.toml                    # 패키지·의존성·설치 설정
├── <lock-file>                       # 선택한 도구의 잠금 파일, 사용할 때 포함
├── configs/
│   ├── default.yaml                  # 공유 기본값
│   ├── local.example.yaml            # 서버별 경로 설정 양식
│   ├── experiments/                  # 실험 조건·모델 선택·seed
│   └── transforms/                   # 전처리·augmentation 설정, 필요할 때
├── src/
│   └── <package_name>/               # 예: ida_steganalysis
│       ├── __init__.py
│       ├── models/                   # 모델·attention·baseline 구현
│       ├── data/                     # Dataset·loader·전처리·CSV 생성 로직
│       ├── engine/                   # 학습·평가·추론 로직
│       ├── analysis/                 # 필요한 분석·시각화 생성 코드
│       ├── utils/                    # 공통 함수
│       └── resources/                # 실행에 필수인 작은 고정 커널 등
├── scripts/
│   ├── train.py                      # 얇은 실행 진입점
│   ├── evaluate.py
│   ├── prepare_data.py               # 데이터 목록·CSV 등을 로컬에 생성
│   └── analyze.py                    # 필요한 경우에만 사용
├── tests/                            # 코드 동작·작은 합성 입력 검증
└── docs/                             # 데이터 준비·외부 코드 출처 등 최소 문서
```

### 파일 배치 기준

| 기존 파일의 역할 | 목표 위치 | 정리 기준 |
| --- | --- | --- |
| arch/ 아래 모델 구현 | src/<package_name>/models/ | 모듈 import와 리소스 경로를 함께 수정 |
| utils/datasets.py, 전처리·loader | src/<package_name>/data/ | 데이터 파일 자체와 분리 |
| utils/csv_generator/ | src/<package_name>/data/ + scripts/prepare_data.py | 생성 코드는 포함, 생성된 CSV는 제외 |
| train*.py의 학습 로직 | src/<package_name>/engine/ | 실행 인자 처리는 scripts로 분리 |
| test*.py의 모델 평가 로직 | src/<package_name>/engine/ + scripts/evaluate.py | 단위 테스트인 tests/와 구분 |
| 시각화·특징 분석 로직 | src/<package_name>/analysis/ | 생성 코드만 포함, 그림·분석 결과는 제외 |
| 반복 실행용 shell | scripts/ | 프로젝트 루트를 계산하고 인자를 전달 |
| 고정 SRM 커널 등 필수 리소스 | src/<package_name>/resources/ | 패키지 설치 시 포함하고 위치 독립적으로 로딩 |
| 원본 데이터·CSV·가중치·로그·결과 | Git 제외 경로 또는 외부 경로 | 기존 로컬 자료 보존 |
| 원고·리뷰·답변서·그림 | Git 제외 경로 | 코드 저장소에 포함하지 않음 |

이동할 때 알고리즘, 기본 학습 조건, 데이터 분할, 전처리 순서와 checkpoint 호환성을 임의로 바꾸지 않는다.
필요하면 기존 train.py/test.py를 새 진입점으로 연결하는 얇은 호환 파일로 잠시 남긴다.
기존 실험 버전은 번호만 보고 최종 모델로 선택하거나 삭제하지 않는다.
README에는 지원하는 모델과 설정의 대응 관계를 기록한다.

웹·앱은 프레임워크의 app/, public/ 등 표준 구조를 따른다.
연구용 engine/ 등을 모든 프로젝트에 강제하지 않으며, 실행 코드·공유 설정·개인 값·산출물의 구분 원칙을 공통 적용한다.

## 3. GitHub에 올리는 범위

| 포함 | 제외 |
| --- | --- |
| 학습·평가·추론·전처리·분석 소스 | 원본·가공 데이터 |
| 실행 스크립트와 데이터 목록 생성 코드 | CSV 파일 및 csv/ 폴더 |
| 공유 YAML/JSON 설정과 개인 설정 예제 | figures/ 및 생성된 그림·그래프 |
| 의존성·패키지·필요한 잠금 파일 | pretrained·학습 checkpoint·ensemble 가중치 |
| 실행에 필수인 작은 고정 커널·리소스 | results/·outputs/·reports/·실험 기록 산출물 |
| 합성 입력 중심의 테스트 | W&B·TensorBoard·실행 로그·캐시 |
| README와 필요한 최소 운영 문서 | 논문 원고·리뷰·답변서·발표자료 |
| 외부 코드에 필요한 라이선스·저작권·인용 정보 | 개인 설정·토큰·설치된 도구·환경 바이너리 |

- 작은 CSV와 최종 그림도 이번 업로드 정책에서는 제외한다.
- 실행에 CSV가 필요하면 스키마·데이터 준비 방법·생성 명령·분할 기준·seed를 문서화한다.
- 기존 CSV가 유일한 split 원본이면 먼저 로컬에 보존한다. 동일 split을 복원할 수 없는 상태를 재현 가능하다고 쓰지 않는다.
- 공개되지 않은 데이터나 checkpoint는 실제 준비 방법과 접근 조건을 설명한다. 코드만 clone해 모든 자료가 생긴다고 가정하지 않는다.
- .npy, .json, .yaml 등은 확장자만으로 일괄 제외하지 않는다. 고정 커널과 공유 설정은 실행에 필요할 수 있다.
- 그림을 생성하는 Python 코드는 포함할 수 있지만, 생성된 이미지·특징 배열·지표 파일은 산출물 경로로 보낸다.
- 노트북은 실행에 필요한 로직을 모듈·스크립트로 옮긴다. 탐색 노트와 출력이 들어 있는 노트북은 기본 업로드 대상에서 제외한다.
- 제외는 Git 관리 범위를 뜻한다. 로컬 원본을 지우거나 실험 결과를 덮어쓰지 않는다.

## 4. 공통 .gitignore 양식

아래는 연구·ML 저장소에 적용할 기본 양식이다. 경로는 .gitignore 위치를 기준으로 해석된다.
루트에 두고, 프로젝트별로 추가 생성 경로를 확인한다.

```gitignore
# Machine-specific configuration / credentials
.env
.env.*
!.env.example
/configs/local.*
!/configs/local.example.*
/configs/private/

# Python environment / generated files
.venv/
venv/
__pycache__/
*.py[cod]
.pytest_cache/
.mypy_cache/
.ruff_cache/
.ipynb_checkpoints/
build/
dist/
*.egg-info/

# Dataset / generated manifests
/data
/datasets
csv
*.csv

# Figures / research artifacts
figures
/results
/outputs
/reports
/analysis_results
/visualizations
/paper
/paper_archive*/
/manuscript
/notebooks
/backups
/.tools
/.cache

# Checkpoint directories, including nested legacy model folders
ckpt
ckpts
checkpoint
checkpoints
checkpoint_384

# Generated weights, also when written outside the directories above
*.pt
*.pth
*.pth.tar
*.ckpt
*.safetensors
*.onnx

# Experiment logs
wandb
runs
lightning_logs
logs
*.log
```

- csv, figures, ckpt처럼 끝 /가 없는 단일 이름 규칙은 같은 이름의 디렉터리·파일·심볼릭 링크를 대상으로 한다.
- /results는 저장소 루트만 대상으로 한다. 다른 위치에도 결과가 생성되면 해당 경로를 추가한다.
- src/<package_name>/data/의 데이터 처리 코드는 루트에 한정한 /data 규칙으로 제외되지 않는다.
- scripts/와 모델 구현이 있는 models/ 전체를 제외하지 않는다.
- 확장자가 없는 checkpoint도 있으므로 저장 디렉터리 규칙과 실제 stage 내용을 확인한다.
- .gitignore는 이미 추적 중인 파일의 추적을 해제하지 않는다. 필요하면 대상 경로에만 git rm --cached를 적용하고 로컬 파일과 diff를 확인한다.
- Git LFS로 이번 산출물 제외 정책을 우회하지 않는다. 자료 배포가 필요해지면 별도로 방식을 정한다.

## 5. 서버 경로와 실행 설정

공유 코드에 특정 서버의 절대경로·GPU 번호·개인 W&B entity를 고정하지 않는다.
공유 기본 설정에는 개인 경로를 넣지 않고 local.example.yaml로 필요한 키를 안내한다.

```yaml
# configs/local.example.yaml
paths:
  data_root: /replace/with/dataset-root
  manifest_root: /replace/with/local-csv-directory
  checkpoint_root: /replace/with/local-checkpoint-directory
  output_root: /replace/with/local-output-directory
tracking:
  enabled: false
  project: null
  entity: null
```

- 실제 값은 Git에서 제외되는 configs/local.yaml에 작성한다.
- 예제 YAML을 복사하는 것만으로 자동 로딩되지 않는다. 진입점이 읽도록 구현한다.
- 설정 우선순위는 명령행 인자 → 개인 설정 → 공유 기본값으로 통일한다.
- 명령행으로 지정한 실험 설정 파일을 먼저 선택하고, 그 공유 설정 위에 개인 값과 명시적 인자를 적용한다.
- 저장소 내부의 상대경로는 프로젝트 루트를 기준으로 처리한다.
- CSV에는 가능한 한 데이터 루트 기준 상대경로를 사용하고, 로더가 data_root와 결합한다.
- 모델 커널 등 패키지 리소스는 패키지 기준으로 찾는다.
- ../../data/ 또는 현재 작업 디렉터리가 우연히 맞아야만 실행되는 구조를 없앤다.
- W&B 등 외부 기록은 선택 사용으로 하고, 비활성 상태에서도 학습·평가가 가능하게 한다.
- 원시 결과는 제외 경로에 저장한다. 실행 기록에는 공개 가능한 설정·seed·데이터 버전·commit·환경을 로컬에 남긴다.

## 6. 기존 연구 구조를 바꾸는 순서

1. 프로젝트의 AGENTS.md, README, Git 루트·브랜치·원격 주소·기존 변경을 확인한다.
2. 코드·설정·커널·외부 코드·데이터·CSV·가중치·결과를 구분한다.
3. 현재 경로 → 목표 경로 → import·실행 명령·리소스 경로의 대응표를 만든다.
4. 데이터와 실험 자료를 보존한 상태에서 코드와 설정부터 정리한다.
5. 패키지 설치와 scripts 진입점을 구성하고 내부 import·리소스 참조를 갱신한다.
6. 개인 경로 설정과 CSV 생성·외부 자료 준비 절차를 연결한다.
7. .gitignore를 적용하고, 제외 경로 안에 실행 코드가 숨어 있지 않은지 확인한다.
8. 기존 검증과 작은 입력 실행으로 이동 전후 동작을 비교한다.
9. 필요한 코드·설정·문서만 명시적으로 stage하고 diff를 검토한다.
10. 프로젝트별 commit·push 후 원격 파일 목록과 최소 실행을 확인한다.

### 로컬 상위 폴더 이름 변경

paper/에서 research/처럼 상위 경로를 바꾸는 작업은 개별 저장소 코드 정리와 구분한다.
다른 연구, 열린 셸, Docker bind mount, 실행 중인 작업, IDE·도구 설정의 경로 의존성을 먼저 확인한다.
필요하면 기존 경로에 한시적인 호환 심볼릭 링크를 유지하고, 의존성이 해소된 뒤 제거한다.
실행 중인 실험을 임의로 중단하거나 컨테이너를 재시작하지 않는다.
다른 연구의 내부 코드를 함께 일괄 수정하지 않는다.

### 우선 적용할 두 연구

| 연구 | 저장소 이름 제안 | 정리할 핵심 |
| --- | --- | --- |
| IDA | ida-steganalysis | IDA·attention·baseline 모델, 학습·평가·시각화·감사 코드, 기존 수치 검증 |
| JPEG-aware Steganalysis | jpeg-aware-steganalysis | 모델별 학습·평가, JPEG·DCT·ROI 처리, 변환 설정, 데이터 목록 생성 |

IDA는 기존 수치 검증과 외부 이미지 의존 검증을 구분한다.
JPEG-aware는 일반·SF3Net·Two-stream 실행 경로와 모델·변환 설정 대응을 기록한다.
여러 실험 버전은 출처와 용도를 보존하고 논문에 대응하는 기준 버전을 확인한다.
SRM 커널 같은 필수 고정 리소스는 포함하고, 모델 폴더 안의 checkpoint와 도구 바이너리는 제외한다.
논문 원고·리뷰·답변서와 CSV·figures·기존 전체 결과는 로컬에 보존한다.

## 7. 검증과 GitHub 업로드

검증한 환경에서 다음을 확인한다. 데이터·GPU·가중치가 필요한 검증은 조건과 수행 범위를 구분한다.

- 패키지 설치, 소스 문법·import, scripts의 인자·설정 로딩
- 기존 테스트와 합성 입력을 이용한 최소 실행
- 필요한 고정 리소스가 설치 후에도 로딩되는지
- 준비된 데이터의 작은 batch로 데이터 로더·모델 forward가 동작하는지
- 기존 checkpoint가 있는 경우 로딩 호환성과 필요한 평가 경로
- CSV·figures·데이터·가중치·산출물·개인 설정이 stage에 포함되지 않았는지
- 새 clone에서 문서대로 설치·자료 준비·최소 실행할 수 있는지

아래 명령은 점검용이다. 실제 stage 경로와 테스트 명령은 프로젝트에 맞춰 명시한다.

```bash
git rev-parse --show-toplevel
git branch --show-current
git remote -v
git status --short
git check-ignore -v --no-index csv/example.csv figures/example.png
git diff --cached --name-status
git diff --cached --stat
git diff --cached --check
```

- 처음 업로드하는 연구는 각 연구 폴더에 별도 저장소를 초기화한다. research/ 전체를 git init하지 않는다.
- 기존 Git 이력과 외부 저장소 이력이 있다면 확인해 보존한다.
- 공개 범위는 업로드 요청과 프로젝트 조건에 맞춘다. 새 연구 저장소는 별도 공개 지시가 없으면 private을 기본안으로 한다.
- 대용량 폴더를 포함한 채 git add .로 시작하지 않는다.
- .gitignore와 함께 실제 stage 목록을 확인한다. 민감한 개인 값이 없는지도 diff에서 확인한다.
- commit은 목적별로 나누고, push 후 원격 branch와 commit을 조회한다.
- 문서 작성·코드 이동·검증·commit·push의 완료 여부를 구분해 보고한다.
- 수행하지 못한 GPU·전체 학습·최종 성능 재현 검증을 완료했다고 쓰지 않는다.

## 8. 대회 코드와 장기 유지

대회·ML 프로젝트도 같은 소스 구조를 사용하고 필요한 경우 scripts/submit.py를 추가한다.
제출 파일은 outputs/ 또는 submissions/에 생성하고 해당 경로를 .gitignore에 추가한다.
데이터 split·모델·설정·commit·제출 점수의 연결은 로컬 기록에 남긴다.
제출 CSV·전체 리더보드 기록·최종 그림을 코드 저장소에 함께 올리지 않는다.
최종 제출·논문 제출·기준 릴리스는 확인한 commit에 태그로 고정한다.

- 새 코드는 기존 역할의 하위 폴더에 추가한다.
- 새 산출물 경로가 생기면 .gitignore와 실행 설정을 함께 갱신한다.
- 새로운 필수 리소스는 필요성·크기·출처를 확인하고 개별 포함한다.
- 공통 양식이 바뀌면 이 가이드 버전과 변경 이유를 기록한다.
- 템플릿 변경은 기존 저장소에 자동 전파되지 않는다. 필요한 변경만 검토해 적용한다.
- 프로젝트 AGENTS.md에는 이 가이드 링크·적용 버전과 프로젝트별 검증 명령을 기록한다.
- 프로젝트 README에는 지원 상태·실행 명령·외부 자료 준비 방법을 유지한다.
- 실행·패키지 구조를 바꾸면 관련 검증을 수행하고, 프로젝트의 추가 완료 지침도 따른다.

## 9. 문서 관리

이 공개 가이드의 원본은 프로필 저장소의 WORKFLOW_GUIDE.md다.
프로필 README에는 이 파일로 연결되는 링크를 둔다.
개별 연구에는 공통 가이드 사본을 계속 복제하지 않고, 적용 버전과 프로젝트별 추가 사항을 기록한다.
개인 Notion의 다른 관리 문서는 별도 목적의 문서로 유지하며 이 업로드 작업에서 자동 변경하지 않는다.

## 공식 참고 자료

- [Git ignore 규칙과 추적 중인 파일](https://git-scm.com/docs/gitignore)
- [GitHub 프로필 README](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/managing-your-profile-readme)
- [Python 패키지의 src 구조](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/)

## 변경 기록

- 2026-11-04 · v1.0: 프로젝트별 저장소, 연구·대회 공통 소스 구조, 코드 중심 업로드, CSV·figures 제외, 기존 연구 이관·검증 절차 정의.
