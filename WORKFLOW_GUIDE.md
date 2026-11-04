# WORKFLOW_GUIDE

> Upload only the code, shared configuration, dependencies, required static resources, and minimal documentation needed to run a project.
> Keep CSV files, figures, datasets, trained weights, experiment results, manuscripts, and logs locally or in separate storage.

- Created / updated: 2026-11-04 (Asia/Seoul)
- Version: 1.2
- Scope: Repository organization, research and competition code structure, uploads, and maintenance for personal GitHub projects.
- This document defines how to reorganize projects. Publishing it does not move local directories or upload research code automatically.

## 1. Repositories and Local Grouping Directories

Create a separate repository for each project whose installation, execution, versions, and visibility are managed independently.
For research, use one repository per paper or independent study by default. For competitions, use one per competition or independent edition.

```text
local-workspace/                      # This parent is not a Git repository
├── research/
│   └── <research-slug>/              # Independent Git repository
├── competitions/
│   └── <competition-slug>/           # Independent Git repository
├── apps/
│   └── <app-slug>/
├── tools/
│   └── <tool-slug>/
└── forks/
    └── <upstream-name>/
```

```text
github-account/                       # Each item below is a separate repository
├── <research-slug>
├── <competition-slug>
├── <app-slug>
├── <tool-slug>
└── <upstream-fork>
```

- Local directory names and remote repository names may differ. Decide separately whether to rename an existing local directory.
- Keep models, baselines, and ablations for the same study in one repository, organized through modules and configuration.
- Manage rounds of the same competition through configuration in one repository. Separate independent competitions or editions.
- A conference paper and its journal extension may share a repository and use tags, depending on how much code and experimentation they share.
- Keep independent apps, tools, and libraries out of research repositories.
- Use lowercase letters, numbers, and hyphens in repository names. Include identifying details, such as a competition year or edition, only when useful.
- Manage changes to external code through a fork or another approach that preserves its source, base commit, and modification history.
- Reorganize existing repositories and competition code individually when requested. This guide does not authorize a bulk migration.

## 2. Common Target Structure for Research and ML Code

Use the following roles and paths as the common structure for new studies and research projects being reorganized.
Do not create unused empty directories. The commands and filenames below describe the target layout; update the implementation and README together.

```text
<repository>/
├── README.md                         # Purpose, environment, setup, data preparation, execution, checks
├── AGENTS.md                         # Project-specific work and verification instructions
├── .gitignore
├── pyproject.toml                    # Package, dependency, and installation configuration
├── <lock-file>                       # Include the selected tool's lock file when used
├── configs/
│   ├── default.yaml                  # Shared defaults
│   ├── local.example.yaml            # Template for machine-specific paths
│   ├── experiments/                  # Experiment conditions, model selection, seeds
│   └── transforms/                   # Preprocessing and augmentation configuration, when needed
├── src/
│   └── <package_name>/               # Importable Python package (snake_case)
│       ├── __init__.py
│       ├── models/                   # Models, attention modules, baselines
│       ├── data/                     # Datasets, loaders, preprocessing, CSV generation logic
│       ├── engine/                   # Training, evaluation, inference logic
│       ├── analysis/                 # Required analysis and visualization generation code
│       ├── utils/                    # Shared helpers
│       └── resources/                # Small static kernels and other required resources
├── scripts/
│   ├── train.py                      # Thin execution entry point
│   ├── evaluate.py
│   ├── prepare_data.py               # Generate manifests and CSV files locally
│   └── analyze.py                    # Include only when needed
├── tests/                            # Behavior checks using small synthetic inputs
└── docs/                             # Minimal data preparation and dependency documentation
```

### File Placement Rules

| Existing role | Target location | Migration rule |
| --- | --- | --- |
| Model implementations under `arch/` | `src/<package_name>/models/` | Update imports and resource paths together |
| `utils/datasets.py`, preprocessing, loaders | `src/<package_name>/data/` | Keep processing code separate from data files |
| `utils/csv_generator/` | `src/<package_name>/data/` + `scripts/prepare_data.py` | Include generation code; exclude generated CSV files |
| Training logic in `train*.py` | `src/<package_name>/engine/` | Keep execution argument handling in scripts |
| Model evaluation logic in `test*.py` | `src/<package_name>/engine/` + `scripts/evaluate.py` | Distinguish evaluation from unit tests in `tests/` |
| Visualization and feature analysis logic | `src/<package_name>/analysis/` | Include generation code; exclude figures and analysis outputs |
| Shell scripts for repeated runs | `scripts/` | Resolve the project root and forward arguments |
| Required resources such as static SRM kernels | `src/<package_name>/resources/` | Include in package installation and load independently of the working directory |
| Datasets, CSV files, weights, logs, results | Git-ignored or external paths | Preserve existing local artifacts |
| Manuscripts, reviews, responses, figures | Git-ignored paths | Keep out of code repositories |

Do not change algorithms, default training conditions, data splits, preprocessing order, or checkpoint compatibility arbitrarily during migration.
When needed, temporarily retain thin compatibility wrappers for existing train.py/test.py commands.
Do not choose or delete experimental versions based only on their version numbers.
Document the mapping between supported models and configurations in the README.

Web projects and apps should follow their framework's standard layout, such as app/ or public/.
Do not force research-specific directories such as engine/ onto every project. Apply the same separation of executable code, shared configuration, personal values, and generated artifacts.

## 3. What to Upload to GitHub

| Include | Exclude |
| --- | --- |
| Training, evaluation, inference, preprocessing, and analysis source code | Raw and processed datasets |
| Execution scripts and manifest generation code | CSV files and csv/ directories |
| Shared YAML/JSON configuration and local configuration templates | figures/ and generated plots or images |
| Dependencies, package configuration, and required lock files | Pretrained weights, training checkpoints, ensemble weights |
| Small static kernels and resources required for execution | results/, outputs/, reports/, experiment record artifacts |
| Tests primarily using synthetic inputs | W&B, TensorBoard, execution logs, caches |
| README and minimal operational documentation | Manuscripts, reviews, response letters, presentation materials |
| Required third-party licenses, copyright notices, and citation information | Personal configuration, tokens, installed tools, environment binaries |

- Exclude even small CSV files and final figures under this upload policy.
- When execution requires CSV files, document their schema, data preparation steps, generation commands, split rules, and seeds.
- If an existing CSV file is the only source of a split, preserve it locally first. Do not claim reproducibility if the same split cannot be reconstructed.
- Explain the actual preparation steps and access conditions for non-public data or checkpoints. Cloning the code does not provide every required resource.
- Do not exclude .npy, .json, or .yaml files solely by extension. Static kernels and shared configuration may be required for execution.
- Python code that generates figures may be included, but generated images, feature arrays, and metric files must go to artifact directories.
- Move execution logic from notebooks into modules and scripts. Exploratory notebooks and notebooks containing outputs are excluded by default.
- Exclusion defines what Git tracks. Do not delete local originals or overwrite experiment results.

## 4. Common .gitignore Template

Use the following baseline for research and ML repositories. Patterns are interpreted relative to the location of .gitignore.
Place it at the repository root and check for additional project-specific output paths.

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

- Single-name patterns without a trailing /, such as csv, figures, and ckpt, match directories, files, and symbolic links with that name.
- /results applies only at the repository root. Add explicit paths if results are generated elsewhere.
- The root-only /data rule does not exclude data processing code under src/<package_name>/data/.
- Do not exclude entire scripts/ directories or models/ directories containing model implementations.
- Some checkpoints have no extension. Check both the output directory rules and the actual staged files.
- .gitignore does not untrack files that Git already tracks. When necessary, apply git rm --cached only to the relevant paths, and verify the local files and diff.
- Do not use Git LFS to bypass this artifact exclusion policy. Choose a separate distribution approach if resource publication becomes necessary.

## 5. Machine Paths and Execution Configuration

Do not hardcode machine-specific absolute paths, GPU indices, or personal W&B entities in shared code.
Keep personal paths out of shared defaults and list the required keys in local.example.yaml.

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

- Put actual values in configs/local.yaml, which is excluded from Git.
- Copying an example YAML file does not load it automatically. Implement loading in the entry point.
- Use the same precedence everywhere: command-line arguments → local configuration → shared defaults.
- First select the experiment configuration file specified on the command line, then apply local values and explicit arguments over that shared configuration.
- Resolve repository-internal relative paths from the project root.
- Where possible, store paths in CSV files relative to the dataset root and let loaders combine them with data_root.
- Resolve package resources, such as model kernels, relative to the package.
- Remove dependencies on ../../data/ or an accidentally correct current working directory.
- Make external tracking, such as W&B, optional. Training and evaluation must also work with tracking disabled.
- Save raw results to excluded paths. Keep public-safe configuration, seeds, data versions, commits, and environment details in local run records.

## 6. Migrating Existing Research Projects

1. Read the project's AGENTS.md and README. Check the Git root, branch, remote URL, and existing changes.
2. Classify code, configuration, kernels, external code, datasets, CSV files, weights, and results.
3. Map current paths to target paths and identify affected imports, execution commands, and resource references.
4. Preserve datasets and experiment artifacts while reorganizing code and configuration first.
5. Configure package installation and scripts entry points, then update internal imports and resource references.
6. Connect local path configuration, CSV generation, and external resource preparation.
7. Apply .gitignore and check that required executable code is not hidden inside excluded directories.
8. Compare behavior before and after migration using existing checks and small inputs.
9. Explicitly stage only the required code, configuration, and documentation, then review the diff.
10. Commit and push each project separately, then verify remote contents and minimal execution.

### Renaming a Local Parent Directory

Treat a parent path change, such as paper/ → research/, separately from reorganizing an individual repository.
First inspect path dependencies in other studies, open shells, Docker bind mounts, running tasks, and IDE or tool configuration.
When needed, temporarily retain a compatibility symbolic link at the old path and remove it after dependencies have been resolved.
Do not stop running experiments or restart containers without authorization.
Do not make bulk changes to other studies' internal code.

### Examples: Research Project Migrations

The following projects illustrate how to apply the common layout. Their names are examples, not required directory names or a fixed migration priority.
For example, a repository named `ida-steganalysis` can use `ida_steganalysis` as its Python package name.

| Example study | Example repository name | Main migration scope |
| --- | --- | --- |
| IDA | ida-steganalysis | IDA, attention modules, baselines, training, evaluation, visualization, audit code, existing numerical checks |
| JPEG-aware Steganalysis | jpeg-aware-steganalysis | Model-specific training and evaluation, JPEG/DCT/ROI processing, transform configuration, manifest generation |

For IDA, distinguish existing numerical checks from checks requiring external images.
For JPEG-aware, document the standard, SF3Net, and Two-stream execution paths and their model/transform configurations.
Preserve the origin and purpose of experimental versions, and identify the reference version corresponding to the paper.
Include required static resources such as SRM kernels; exclude checkpoints inside model directories and installed tool binaries.
Preserve manuscripts, reviews, response letters, CSV files, figures, and all existing results locally.

## 7. Verification and GitHub Upload

Check the following in the verified environment. State the conditions and completed scope for checks requiring datasets, GPUs, or weights.

- Package installation, source syntax and imports, script arguments, and configuration loading
- Existing tests and minimal execution with synthetic inputs
- Loading required static resources after installation
- Data loader and model forward execution with a small batch of prepared data
- Checkpoint loading compatibility and relevant evaluation paths when existing checkpoints are available
- Absence of CSV files, figures, datasets, weights, artifacts, and personal configuration from staged files
- Installation, resource preparation, and minimal execution from a fresh clone using the documentation

The commands below are for inspection. Specify actual staging paths and test commands for each project.

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

- For a first upload, initialize a separate repository inside each research project. Do not run git init over the entire research/ directory.
- Inspect and preserve existing Git history and upstream repository history.
- Set visibility according to the upload request and project conditions. Default new research repository proposals to private unless public access is explicitly requested.
- Do not start with git add . while large artifact directories are present.
- Inspect the actual staged files alongside .gitignore. Check the diff for sensitive personal values.
- Group commits by purpose and verify the remote branch and commit after pushing.
- Report document creation, code migration, verification, commit, and push as distinct completion states.
- Do not report GPU checks, full training, or final performance reproduction as completed when they were not performed.

## 8. Competition Code and Long-Term Maintenance

Use the same source layout for competition and ML projects, adding scripts/submit.py when needed.
Generate submission files under outputs/ or submissions/ and add the corresponding path to .gitignore.
Keep local records linking data splits, models, configuration, commits, and submission scores.
Do not upload submission CSV files, full leaderboard records, or final figures with the code.
Tag verified commits for final competition submissions, paper submissions, and reference releases.

- Add new code under the existing directory corresponding to its role.
- Update .gitignore and execution configuration whenever a new output path is introduced.
- Review the need, size, and source of each new required resource before including it.
- Record the guide version and reason whenever the shared layout changes.
- Template changes do not propagate automatically to existing repositories. Review and apply only the relevant updates.
- In each project's AGENTS.md, record the guide link, applied version, and project-specific verification commands.
- Maintain supported status, execution commands, and external resource preparation steps in the project README.
- Run relevant checks after execution or package layout changes, and follow any additional project completion instructions.

## 9. Document Maintenance

The source of truth for this public guide is WORKFLOW_GUIDE.md in the profile repository.
Link to this file from the profile README.
Do not repeatedly copy the shared guide into individual research projects. Record the applied version and project-specific additions instead.
Other management documents in personal Notion serve separate purposes and are not automatically modified by this upload.

## Official References

- [Git ignore rules and tracked files](https://git-scm.com/docs/gitignore)
- [GitHub profile README](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/managing-your-profile-readme)
- [Python package src layout](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/)

## Changelog

- 2026-11-04 · v1.0: Defined project-specific repositories, a common research and competition source layout, code-focused uploads, CSV/figures exclusions, and migration and verification procedures.
- 2026-11-04 · v1.1: Translated the guide into English while preserving its structure, upload policy, and configuration examples.
- 2026-11-04 · v1.2: Generalized repository and package placeholders in the standard layouts; kept concrete research names in a clearly labeled examples section.
