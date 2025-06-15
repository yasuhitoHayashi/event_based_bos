# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the official implementation of "Event-based Background-Oriented Schlieren" (IEEE T-PAMI 2023), a computer vision research project that uses event cameras for optical flow estimation and background-oriented schlieren imaging.

The codebase implements various solvers for estimating dense optical flow from event data, with comparison against frame-based methods for evaluation.

## Development Commands

### Setup and Installation
```bash
# Using poetry (recommended for development)
poetry install

# Quick setup with venv
python3 -m venv ~/work/venv
source ~/work/venv/bin/activate
pip3 install scipy==1.9.1 optuna==2.10.1 opencv-python matplotlib plotly \
ffmpeg-python h5py hdf5plugin PyYAML Pillow sklearn scikit-image argparse openpiv \
torch torchvision black isort kaleido
```

### Code Quality
```bash
# Format code (black + isort)
make fmt

# Type checking (mypy)
make lint
```

### Running the Main Application
```bash
# Basic execution with config file
python3 bos_event.py --config_file ./configs/hot_plate1.yaml --eval

# Evaluation mode (compares against frame-based ground truth)
python3 bos_event.py --config_file ./configs/crushed_ice.yaml --eval
```

## Architecture Overview

### Core Components

**Main Entry Point**: `bos_event.py` - Main script that orchestrates the entire pipeline

**Data Loading** (`src/data_loader/`):
- `base.py`: Abstract base class for data loaders
- `ccs.py`: Loader for Co-Capture System (CCS) datasets
- `e2vid.py`, `helium.py`: Loaders for other event camera formats

**Solvers** (`src/solver/`):
- `base.py`: Abstract base class for all optimization solvers
- `generative_max_likelihood.py`: Generative maximum likelihood solver
- `patch_eklt*.py`: Various patch-based event Kanade-Lucas-Tomasi implementations
- `scipy_autograd/`: Wrapper for scipy optimizers with automatic differentiation

**Cost Functions** (`src/costs/`):
- Modular cost functions for optimization (flow_norm, image_gradient, hybrid, etc.)
- Each cost function implements the objective being optimized

**Event Processing**:
- `event_image_converter.py`: Converts event streams to images/representations
- `frame_flow_estimator.py`: Frame-based optical flow for ground truth comparison

### Key Design Patterns

1. **Plugin Architecture**: Solvers, data loaders, and cost functions use collections pattern for easy extension
2. **Configuration-Driven**: All experiments controlled via YAML config files in `configs/`
3. **Modular Optimization**: Solvers can use different optimizers (Optuna, scipy, torch) via wrapper classes

### Dataset Structure

The code expects datasets in this structure:
```
datasets/
  CCS/  # Co-Capture System datasets
    (sequence_name)/
      basler_0/
        frames.mp4
        config.yaml
      prophesee_0/
        events.hdf5
        trigger_events.txt
      homography.txt
```

### Configuration

All experiments are configured via YAML files in `configs/`. Key sections:
- `data`: Dataset and sequence specification
- `solver`: Method and optimization parameters
- `evaluation`: Time ranges and metrics for evaluation
- `common_params`: ROI cropping and processing parameters

See `configs/README.md` for detailed parameter documentation.

## Important Notes

- The main script supports both evaluation mode (--eval) and sequential estimation mode
- Event data is expected in HDF5 format for cross-platform compatibility
- The code uses both PyTorch and NumPy backends depending on the solver
- Visualization outputs are automatically saved to the configured output directory