# WORKFLOW_GUIDE

> **GitHub에는 실행에 필요한 코드·공통 설정·의존성·최소 문서만 올린다.**
> 데이터·가중치·실험 결과·그림·로그·인증정보는 로컬 또는 별도 저장소에 보관한다.

| 항목 | 기준 |
| --- | --- |
| 작성 / 수정 | 2026-11-04 / 2026-11-10 · Asia/Seoul |
| 버전 | **1.7** · 가독성 개선, 기존 운영 규칙 유지 |
| 적용 | 개인 GitHub 프로젝트의 구조·commit/push·유지보수 |
| 원본 | 이 저장소의 `WORKFLOW_GUIDE.md`; 프로젝트에는 링크·적용 버전·추가 규칙만 기록 |
| 범위 | 문서 수정 자체는 폴더 이동·코드 업로드·실험 재시작을 뜻하지 않음 |

**바로 찾기:** [구조](#2-common-target-structure-for-research-and-ml-code) · [업로드](#3-what-to-upload-to-github) · [.gitignore](#4-common-gitignore-template) · [경로](#5-machine-paths-and-execution-configuration) · [W&B](#51-wb-experiment-tracking) · [작업 순서](#6-migrating-existing-research-projects) · [push 전 확인](#7-verification-and-github-upload)

## 1. Repositories and Local Grouping Directories

**설치·실행·버전·공개 범위를 독립 관리하는 단위 = 저장소 하나.**

```text
local-workspace/                  # 상위 분류 폴더: Git 저장소가 아님
├── research/<project-slug>/      # 각 프로젝트가 독립 Git 저장소
├── competitions/<project-slug>/
├── apps/<project-slug>/
├── tools/<project-slug>/
└── forks/<upstream-name>/

github-account/
├── <project-slug>                # 로컬 research/ 등의 상위 경로는 올라가지 않음
└── <another-project-slug>
```

| 판단 대상 | 기준 |
| --- | --- |
| 논문 / 독립 연구 | 기본적으로 연구별 저장소; 동일 연구의 모델·baseline·ablation은 모듈·설정으로 구분 |
| 학회 논문 → 저널 확장 | 코드·실험 공유 정도에 따라 같은 저장소 + tag 사용 가능 |
| 대회 | 독립 대회·회차는 분리; 같은 대회 내부 라운드는 설정으로 관리 |
| 앱·도구·라이브러리 | 연구 저장소와 독립 관리 |
| 외부 코드 수정 | fork 등으로 출처·기준 commit·수정 이력 보존 |
| 이름 | 소문자·숫자·하이픈; 연도·회차는 구분에 필요할 때만 포함 |
| 로컬 경로 ↔ GitHub 이름 | 달라도 됨; 로컬 폴더명 변경은 별도 판단 |
| 기존 프로젝트 정리 | 요청된 프로젝트 단위로 수행; 일괄 이전 권한으로 해석하지 않음 |

## 2. Common Target Structure for Research and ML Code

**역할에 맞춰 배치하고, 사용하지 않는 빈 폴더는 만들지 않는다.**

```text
<repository>/
├── README.md                    # 문제·논문·방법·간단한 사용법
├── AGENTS.md                    # 이 프로젝트의 추가 규칙·검증 방법
├── .gitignore
├── pyproject.toml               # 패키지·의존성·설치
├── <lock-file>                  # 사용하는 도구의 lock 파일
├── configs/
│   ├── default.yaml             # 공통 기본값
│   ├── local.example.yaml       # 개인 설정 양식
│   ├── experiments/             # 조건·모델·seed
│   └── transforms/              # 필요할 때 전처리·증강
├── src/
│   ├── data/                    # 실제 데이터·manifest: Git 제외
│   └── <package_name>/          # import 가능한 코드: snake_case
│       ├── __init__.py
│       ├── models/              # 모델·알고리즘
│       ├── utils/               # 공통 함수·로더·data_processing/
│       ├── engine/              # 필요할 때 공유 실행 로직
│       ├── analysis/            # 필요할 때 분석 코드
│       └── resources/           # 실행에 필요한 작은 정적 리소스
├── scripts/                     # train/evaluate/prepare_data 등 작업별 CLI
├── tests/                       # 작은 합성 입력을 쓰는 동작 검증
└── docs/                        # 실행·데이터 준비 등 최소 문서
```

| 내용 | 위치 / 수정 기준 |
| --- | --- |
| `arch/` 등 모델 구현 | `src/<package_name>/models/`; import·리소스 경로 함께 수정 |
| 로더·전처리·split·manifest/CSV 생성 함수 | `utils/` 또는 `utils/data_processing/` |
| 준비·학습·평가 workflow와 인자 처리 | `scripts/`; 반드시 thin wrapper일 필요 없음 |
| 여러 작업이 공유하는 실행 로직 | 필요할 때 `engine/`; 불필요한 중간 위임 계층 금지 |
| 작업별 CLI 이름 | train/evaluate 외에 embed_message/extract_message/generate_cover 등 실제 작업명 허용 |
| 같은 작업의 여러 알고리즘 | 가능하면 `--method`로 선택; 생성 모델·설정은 별도 인자, 기존 method별 기본값 유지 |
| 모델 평가 vs 단위 테스트 | `scripts/evaluate.py` vs `tests/`로 구분 |
| 분석·시각화 | 코드는 `analysis/`, 생성 그림·특징·지표는 제외 경로 |
| 반복 실행 shell script | `scripts/`; 프로젝트 root 확인 후 인자 전달 |
| SRM kernel 등 필수 정적 리소스 | `resources/`; 패키지 설치 후에도 cwd와 무관하게 로드 |
| 실제 데이터·생성 manifest | Git 제외 `src/data/` 또는 명시한 외부 저장소 |
| 가중치·결과·논문 자료 | 기존 로컬 원본 보존, Git 제외 |
| 웹·앱 | 해당 framework의 `app/`, `public/` 등 표준 구조 사용; 연구 구조 강제 금지 |

**구조 변경 시 보존할 것**

- 알고리즘·기본 학습 조건·데이터 split·전처리 순서·checkpoint 호환성.
- 서로 다른 알고리즘·실험 버전의 목적과 출처; 버전 번호만 보고 병합·삭제하지 않음.
- 필요하면 기존 train/test 명령의 임시 호환 wrapper 유지; 지원 모델↔설정은 README에 안내.
- `src/data/`는 Python 패키지가 아님: `__init__.py`·처리 코드·모델을 두지 않음.
- 패키지 탐색은 `<package_name>` 하위로 제한; 실제 데이터는 Git뿐 아니라 source/wheel 배포에서도 제외.
- 기존 데이터·코드는 해당 프로젝트 이전 요청 시에만 이동; 구현·명령·README를 함께 맞춤.

## 3. What to Upload to GitHub

| 올림 | 올리지 않음 |
| --- | --- |
| 학습·평가·추론·전처리·분석 소스 | 원본/가공 데이터·생성 manifest·CSV |
| 실행 script·manifest 생성 코드 | 그림·생성 이미지·특징 배열·지표 파일 |
| 공통 YAML/JSON·개인 설정의 예시 | 개인 경로·계정 설정·토큰·인증정보 |
| 의존성·패키지 설정·필요한 lock 파일 | 가중치·checkpoint·ensemble weight |
| 실행 필수 정적 kernel·리소스 | results/outputs/reports·실험 기록·로그·cache |
| 작은 합성 입력 중심의 테스트·최소 문서 | 논문·리뷰·답변서·발표 자료 |
| 외부 코드 license·저작권·인용 정보 | 설치된 도구·가상환경·binary·기본적으로 탐색/출력 notebook |

| 헷갈리기 쉬운 항목 | 원칙 |
| --- | --- |
| 작은 CSV·최종 그림 | 크기와 무관하게 제외 |
| `src/data/` 안의 데이터 | 소스가 아님; 로더·준비 코드는 외부의 실행 모듈에 배치 |
| `.npy/.json/.yaml` | 확장자만으로 제외하지 않음; 필수 정적 리소스·공통 설정은 포함 가능 |
| 실행에 필요한 CSV | schema·생성 명령·split 규칙·seed 문서화; 유일한 split 원본이면 먼저 로컬 보존 |
| 비공개 데이터·checkpoint | 준비 방법·접근 조건 명시; clone만으로 자원이 제공된다고 주장하지 않음 |
| notebook | 실행 로직을 모듈/script로 옮김; 탐색 notebook·출력 포함 notebook은 기본 제외 |
| W&B | 연동 소스는 허용; run/account 폴더·metadata/summary/history·binary `*.wandb`·인증정보는 제외 |
| 제외의 의미 | Git 추적 제외이며 로컬 원본·기존 결과 삭제가 아님; 재구성 불가능한 split의 재현성을 주장하지 않음 |

## 4. Common .gitignore Template

저장소 root에 적용하고, 프로젝트별 추가 출력 경로를 확인한다.

<details>
<summary>공통 .gitignore 양식 펼치기</summary>

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

# Actual local datasets / generated manifests
/src/data
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

# Experiment logs and W&B local account/run artifacts
wandb
wandb/
.wandb/
*.wandb
wandb-metadata.json
wandb-summary.json
wandb-history.jsonl
wandb-events.jsonl
.netrc
runs
lightning_logs
logs
*.log
```

</details>

| 확인 항목 | 주의점 |
| --- | --- |
| `csv`, `figures`, `ckpt` | 단일 이름 규칙은 같은 이름의 파일·폴더·symlink에 적용 |
| `/results` | 저장소 root만 적용; 다른 출력 위치는 별도 추가 |
| `/src/data` | 실제 데이터만 제외; `src/<package_name>/utils/`의 로더는 포함 |
| 기존 `src/<package_name>/data/` 코드 | 요청된 이전에서 import와 함께 이동; 무조건 삭제·ignore하지 않음 |
| `scripts/`, 모델 구현이 든 `models/` | 폴더 전체를 제외하지 않음 |
| 확장자 없는 checkpoint | 폴더 규칙과 실제 staged 파일을 모두 확인 |
| 이미 추적 중인 파일 | .gitignore로 해제되지 않음; 필요한 경로만 `git rm --cached`, 로컬 보존·diff 확인 |
| Git LFS | 제외 정책 우회에 사용하지 않음; 자원 공개 필요 시 별도 배포 방식 선택 |

## 5. Machine Paths and Execution Configuration

**우선순위: 명시한 CLI 인자 → 개인 설정 → 공통 설정.**

```yaml
# configs/local.example.yaml
paths:
  data_root: src/data
  manifest_root: src/data/manifests
  checkpoint_root: /replace/with/local-checkpoint-directory
  output_root: /replace/with/local-output-directory
tracking:
  enabled: false
  project: null
  entity: null
```

| 항목 | 규칙 |
| --- | --- |
| 실제 개인 값 | Git 제외 `configs/local.yaml`; 절대 경로·GPU 번호·개인 W&B entity를 공통 코드에 고정하지 않음 |
| 설정 로딩 | CLI가 지정한 실험 설정을 먼저 선택 → 개인 값 → 명시 CLI로 덮어씀; 예시 YAML 복사만으로 자동 적용되지 않음 |
| 내부 상대 경로 | 프로젝트 root 기준; 우연한 cwd·`../../data/` 의존 제거 |
| 데이터 파일의 경로 | 가능하면 dataset root 기준 상대 경로 + loader의 `data_root` |
| 패키지 리소스 | 패키지 위치 기준으로 로드 |
| 추적 기능 | 선택적 적용; 비활성화해도 학습·평가 가능 |
| 로컬 실행 기록 | 제외 경로에 원시 결과와 공개 가능한 설정·seed·데이터 버전·commit·환경 보관 |

### 5.1 W&B Experiment Tracking

```text
Workspace: <individual-or-team>
└── Project: <project-slug>
    └── Group: <experiment_id>
        └── Run: <job_type>__<condition>__seed<seed>__trial<trial>
```

| 단위 | 의미 / 기준 |
| --- | --- |
| Project | 독립 관리 프로젝트, 보통 저장소와 대응; 모델·데이터·인자 변경만으로 분리하지 않음 |
| Group | 하나의 비교 질문; 기본 ID `YYYYMMDD_<experiment-slug>_v<version>`, Asia/Seoul |
| 프로젝트별 기존 양식 | 기존 구조와 명시한 프로젝트 규칙 유지; Group/Run/실제 output 경로의 대응 기록 |
| Run | 고정 조건의 실행 1회; 조건·seed·trial·단계가 달라지면 분리 |
| job_type | 실제 작업: prepare/generate/train/evaluate/analyze 등 |
| Run ID | 표시 이름과 다른 고유 식별자; 설정을 확인한 의도된 Run만 resume |
| 기본 로컬 경로 | 새 프로젝트는 `results/<experiment_id>/<run_name>/`; 기존 결과를 자동 이동하지 않음 |

| 로깅 대상 | 범위 |
| --- | --- |
| 추적 정보 | experiment_id·condition·seed·trial·job_type·실제 설정·commit/dirty·데이터 버전/hash/split·모델 revision·환경·최종 상태·실제 output 경로 |
| 의존 관계 | 원천 데이터 Run ID·학습 Run ID로 연결 |
| 적용되지 않는 값 / 없는 근거 | N/A로 표시 / 임의 provenance 생성 금지 |
| 결과 | 진행·최종 지표, 작은 표, 제한된 고정 예시 |
| 제외 | 대규모 데이터·출력·가중치는 별도 저장 정책; 인증정보·비밀 키·비밀 메시지 업로드 금지 |
| 인증·로컬 기록 | 선택적 SDK; 개인 entity·인증은 비공개 설정/환경, run/cache/account/binary 파일 Git 제외 |
| 사전 확인 | 작은 Run에서 이름·config·지표·로컬 경로 연결 확인; 문서 규칙·SDK 구현·인증·업로드·재시작은 별개 단계 |

프로젝트 AGENTS에는 **이 절의 링크 + 로컬 매핑·추가 지표**만 기록한다. 개인 Notion AGENTS는 사용자용 공통 입구이며, Notion 안내의 로컬 사본을 만들지 않는다.

## 6. Migrating Existing Research Projects

| 순서 | 할 일 |
| --- | --- |
| 1. 확인 | AGENTS·README → Git root·branch·remote·기존 변경 확인 |
| 2. 분류 | 코드·설정·kernel·외부 코드 / 데이터·CSV·가중치·결과 분리 |
| 3. 매핑 | 기존→대상 경로, import·실행 명령·리소스 영향 정리 |
| 4. 보존 | 데이터·실험 산출물 유지; 코드·설정부터 정리 |
| 5. 연결 | 패키지 설치·CLI·import·리소스 경로 수정 |
| 6. 데이터 | 준비/로더 함수는 utils 등 실행 모듈, 실제 데이터는 src/data 등 제외 경로; 생성·외부 자원 준비 문서화 |
| 7. 제외 | .gitignore 적용, 필요한 실행 코드가 숨겨지지 않는지 확인 |
| 8. 검증 | 기존 검사·작은 입력으로 변경 전후 동작 비교 |
| 9. commit | 필요한 코드·설정·문서만 명시적으로 stage, diff 검토 |
| 10. push | 프로젝트별 commit/push → remote 내용·최소 실행 확인 |

**상위 폴더명 변경은 별도 작업이다.**

| 예: `paper/` → `research/` | 먼저 확인 |
| --- | --- |
| 영향 | 다른 연구의 경로 의존, 열린 shell, Docker mount, 실행 중 작업, IDE·도구 설정 |
| 호환 | 필요 시 임시 symlink 유지 후 의존 해소 시 제거 |
| 보존 | 허가 없이 실험 중단·컨테이너 재시작 금지; 다른 연구 내부 코드를 일괄 변경하지 않음 |

<details>
<summary>기존 연구에 적용할 때의 확인 예시</summary>

| 예시 | 확인 범위 |
| --- | --- |
| IDA → `ida-steganalysis` / `ida_steganalysis` | IDA·attention·baseline·학습·평가·시각화·audit·기존 수치 검사; 외부 이미지 필요한 검사와 분리 |
| JPEG-aware → `jpeg-aware-steganalysis` | standard/SF3Net/Two-stream 실행·설정, JPEG/DCT/ROI·transform·manifest 생성 |

예시는 고정 폴더명·이전 우선순위가 아니다. 실험 버전의 출처·목적·논문 기준 버전을 식별하고, 필수 정적 자원만 포함한다. 모델 폴더 내부 checkpoint·설치 도구·논문·리뷰·답변서·CSV·그림·기존 결과는 로컬 보존한다.

</details>

## 7. Verification and GitHub Upload

**변경에 맞는 검사를 수행하고, 실제 확인한 범위만 보고한다.**

| 확인 | 조건 |
| --- | --- |
| 설치·syntax·import·CLI·설정 로딩 | 사용한 환경 명시 |
| 기존 테스트·합성 입력 최소 실행 | 변경에 맞는 검사 선택 |
| 설치 후 정적 리소스 로드 | 필요한 자원이 패키지에 포함되는지 확인 |
| loader·model forward 작은 batch | 준비된 데이터·GPU·가중치 필요 여부 명시 |
| checkpoint·관련 평가 경로 | 기존 checkpoint가 있을 때 호환 확인 |
| 새 clone의 최소 실행 | 문서대로 설치·자원 준비·실행 가능한 범위 확인 |
| staged 파일 | CSV·그림·데이터·가중치·산출물·개인 값이 없는지 실제 파일과 diff 점검 |

<details>
<summary>Git 확인 명령 펼치기</summary>

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

실제 stage 경로와 테스트 명령은 프로젝트에 맞게 지정한다.

</details>

- [ ] 첫 업로드는 프로젝트 안에서만 `git init`; 상위 research/ 전체를 저장소로 만들지 않음.
- [ ] 기존 Git·upstream 이력 보존. 새 연구 저장소 제안은 명시적 공개 요청이 없으면 private 기본.
- [ ] 큰 산출물이 있는 상태에서 `git add .`부터 실행하지 않음.
- [ ] .gitignore와 실제 staged 파일·개인 값 모두 확인.
- [ ] 목적별 commit → push 후 remote branch·commit 확인.
- [ ] 문서 작성 / 코드 이전 / 검증 / commit / push 완료 상태를 구분해 보고.
- [ ] 수행하지 않은 GPU 검사·전체 학습·최종 성능 재현을 완료로 표시하지 않음.

## 8. Competition Code and Long-Term Maintenance

| 상황 | 관리 방법 |
| --- | --- |
| 대회 제출 | 같은 소스 구조 + 필요 시 scripts/submit.py; 제출물은 제외한 outputs/ 또는 submissions/ |
| 로컬 재현 기록 | split·모델·설정·commit·제출 점수 연결; 제출 CSV·전체 leaderboard 기록·최종 그림 업로드 금지 |
| 기준 버전 확정 | 검증한 대회 최종 제출·논문 제출·reference release commit에 tag |
| 새 코드·출력 경로 | 기존 역할별 폴더에 추가; .gitignore·실행 설정 함께 갱신 |
| 새 필수 자원 | 필요성·크기·출처 검토 후 포함 |
| 공통 구조 변경 | GUIDE 버전·이유 기록; 기존 프로젝트에는 필요한 부분만 검토·적용 |
| 프로젝트 AGENTS | GUIDE 링크·적용 버전·프로젝트별 규칙·검증 명령 |
| README | 지원 상태·실행 명령·외부 자원 준비 안내 |
| 실행·패키지 구조 변경 | 관련 검사 + 프로젝트별 완료 지침 수행 |

## 9. Document Maintenance

### Paper-Focused Research READMEs

| 내용 | 작성 기준 |
| --- | --- |
| 첫 화면 | 연구 문제 → 대응 논문 → 핵심 방법; 복사·이전 이력으로 시작하지 않음 |
| 논문 확인 | 기여 설명 전에 논문·공식 저자/출판 기록 확인; 필요 시 로컬 PDF/LaTeX 참고 |
| 메타데이터 | 제목·저자·venue·연도·DOI·출판 상태 검증; 폴더명·template·익명 draft로 게재 추정 금지 |
| 설명 | 원문을 그대로 복사하지 않고 작성; protocol vs model, 논문 주장 vs 코드 동작, 보고값 vs 직접 재현 구분 |
| 출처 | 확인된 논문/DOI·인용 정보만 포함; 공식 구현 여부도 검증 |
| 사용 안내 | 간단히 쓰고 상세 설치·데이터 준비·명령은 docs/ 연결 |
| 운영 기록 | 구조 이전·복사 이력·agent workflow·검증 로그는 docs/ 또는 AGENTS |
| 짧은 WIP 요청 | WIP 제목 + 주제·계획 2–3줄; 미검증 결과·완성 주장·긴 구조 이력 금지 |
| 로컬 원고 참고 | 읽기 허용이 GitHub 원고·그림·데이터·미공개 부록 업로드 허용을 뜻하지 않음 |

**문서 유지 원칙**

- 공통 GUIDE 원본은 profile 저장소의 이 파일; profile README에서 연결.
- 프로젝트마다 GUIDE를 복제하지 않고 적용 버전·추가 규칙만 기록.
- 개인 Notion의 다른 관리 문서는 별도 용도이며, GitHub 문서 갱신만으로 자동 변경되지 않음.

## Official References

- [Git ignore rules and tracked files](https://git-scm.com/docs/gitignore)
- [GitHub profile README](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/managing-your-profile-readme)
- [Python package src layout](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/)

## Changelog

| 날짜 | 버전 | 변경 |
| --- | --- | --- |
| 2026-11-10 | 1.7 | 표·트리·체크리스트·접기 양식, 한국어 요약으로 재구성; 기존 운영 정책 유지 |
| 2026-11-09 | 1.6 | 공통 W&B 계층·이름·메타데이터·로컬 경로 매핑·저장 경계 |
| 2026-11-04 | 1.5 | W&B 계정/run/history/binary·인증정보 제외 |
| 2026-11-04 | 1.4 | 연구 중심 README·1차 출처 검증·운영 문서 분리·짧은 WIP |
| 2026-11-04 | 1.3 | src/data는 실제 데이터, 로더는 utils, engine/wrapper는 선택 |
| 2026-11-04 | 1.2 | 저장소·패키지 이름 일반화, 구체적 연구는 예시로 분리 |
| 2026-11-04 | 1.1 | 당시 GUIDE 영어 번역, 기존 구조·정책·설정 유지 |
| 2026-11-04 | 1.0 | 독립 저장소·공통 구조·코드 중심 업로드·이전/검증 절차 |
