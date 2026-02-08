# AccuSleePy

## Description

AccuSleePy is set of graphical user  interfaces for scoring rodent sleep
using EEG and EMG recordings.
It offers the following improvements over the MATLAB version (AccuSleep):

- Up to 10 brain states can be configured through the user interface
- Classification models can be trained through the user interface
    - Model files contain useful metadata (brain state configuration,
      epoch length, number of epochs)
    - Models optimized for real-time scoring can be trained
- Confidence scores can be saved and visualized
- Lists of recordings can be imported and exported for repeatable batch processing
- Undo/redo functionality in the manual scoring interface

If you use AccuSleep in your research, please cite our
[publication](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0224642):

Barger, Z., Frye, C. G., Liu, D., Dan, Y., & Bouchard, K. E. (2019). Robust, automated sleep scoring by a compact neural network with distributional shift correction. *PLOS ONE, 14*(12), 1–18.

The data and models associated with AccuSleep are available at https://osf.io/py5eb/

Please contact zekebarger (at) gmail (dot) com with any questions or comments about the software.


## Installation

- (recommended) create a new virtual environment (using
[venv](https://docs.python.org/3/library/venv.html),
[conda](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html),
etc.) with python >=3.11,<3.14
- Canonical one-command install (from repo root): `pip install -e .`
- Optional GUI dependencies: `pip install -e ".[gui]"`
- (optional) if you have a CUDA device and want to speed up model training, install a CUDA-enabled PyTorch build from [pytorch.org](https://pytorch.org/)
- (optional) download a classification model from https://osf.io/py5eb/ under /python_format/models/


## Usage

`python -m accusleepy` will open the primary interface.

Your settings are saved to a platform-specific location
(e.g., `~/Library/Application Support/accusleepy/config.json` on macOS)

[Guide to the primary interface](accusleepy/gui/text/main_guide.md)

[Guide to the manual scoring interface](accusleepy/gui/text/manual_scoring_guide.md)

## Data Card

- Dataset source: AccuSleep public data/models at https://osf.io/py5eb/
- Modalities: EEG and EMG recordings plus per-epoch sleep-stage labels
- Core files:
  - `recording.parquet` (or `.csv`) with columns `eeg` and `emg`
  - `labels.csv` with column `brain_state` and optional `confidence_score`
- Default label space:
  - REM = 1, Wake = 2, NREM = 3
  - Undefined label is `-1`
- Epoch assumptions:
  - Epoch length is configurable (default 2.5 s in bundled config)
  - Training and inference epoch lengths must match the model metadata
- Calibration assumptions:
  - Calibration files are subject/condition specific
  - Reuse calibration only for recordings from the same subject/conditions
- Data quality checks used in the demo notebook pipeline:
  - EEG/EMG non-empty and aligned in length
  - `brain_state` column exists
  - all scored classes appear in labeled data
  - inferred sampling rate in plausible range

## Suggested Training/Evaluation Data Size And Resources

Observed demo shape in this repo outputs (`2026-02-07` runs):
- 50 validated recordings
- each recording is 4 hours, 5,760 epochs at 2.5 s/epoch
- one-recording training image generation produced 4,608 training images + 1,152 calibration images

Suggested tiers:
- Quick sanity tier:
  - Data: 1 train recording + 1 eval recording (about 11,520 epochs total)
  - Hardware: modern 4-core CPU, 8 GB RAM, no GPU required
  - Storage: at least 5 GB free (images + reports + models)
- Recommended baseline tier:
  - Data: 10-20 train recordings + 3-5 eval recordings
  - Hardware: 8+ CPU cores, 16 GB RAM, optional GPU with 8 GB VRAM
  - Storage: 20-50 GB free (intermediate training images can dominate)
- Large/production-like tier:
  - Data: 40+ train recordings + 8-10 eval recordings
  - Hardware: 12+ CPU cores, 32 GB RAM, GPU with 12+ GB VRAM recommended
  - Storage: 80+ GB free for repeated experiments and artifacts

## Developer guide
If you want to contribute to the project or modify the code for your own use,
please consult the [developer guide](accusleepy/gui/text/dev_guide.md).

## Changelog

- 0.11.1: Add integration tests
- 0.11.0: Store config file in a user directory
- 0.10.0-0.10.1: Improved zoom behavior, updated dependencies
- 0.7.1-0.9.3: Bugfixes, code cleanup, additional config settings
- 0.7.0: More settings can be configured in the UI
- 0.6.0: Confidence scores can now be displayed and saved. Retraining your models is recommended
    since the new calibration feature will make the confidence scores more accurate.
- 0.5.0: Performance improvements
- 0.4.5: Added support for python 3.13, **removed support for python 3.10.**
- 0.1.0-0.4.4: Early development versions

## Screenshots

Primary interface
![AccuSleePy primary interface](accusleepy/gui/images/primary_window.png)

Manual scoring interface
![AccuSleePy manual scoring interface](accusleepy/gui/images/viewer_window.png)

## Acknowledgements

We would like to thank [Franz Weber](https://www.med.upenn.edu/weberlab/) for creating an
early version of the manual labeling interface. The code that
creates spectrograms comes from the
[Prerau lab](https://github.com/preraulab/multitaper_toolbox/blob/master/python/multitaper_spectrogram_python.py)
with only minor modifications.
Jim Bohnslav's [deepethogram](https://github.com/jbohnslav/deepethogram) served as an
incredibly useful reference when reimplementing this project in python.
The model calibration code added in version 0.6.0 comes from Geoff Pleiss'
[temperature scaling repo](https://github.com/gpleiss/temperature_scaling).

## Local Modification Notes

- Modification author: `yuanlong-o`
- Date: `2026-02-08`
- Changes in this update:
  - README: added one-command install guidance, data card, and suggested training/evaluation data-size and resource tiers
  - `pyproject.toml`: changed to headless-default install, added optional `gui` extra, and pinned `torch`/`torchvision` to tested ranges
  - `.gitignore`: explicitly ignores `outputs/` and `notebooks/wandb/`
  - Device fallback: uses explicit `cuda` -> `mps` -> `cpu` logic in `accusleepy/classification.py` and `accusleepy/temperature_scaling.py`
