# transformer_EE

The `transformer_EE` project is a lightweight, customizable framework designed to estimate neutrino energies using transformer encoder architectures.
It is particularly tailored for high-energy physics experiments like NOvA and DUNE, where precise energy reconstruction of neutrino interactions is crucial.([GitHub][1])

---

## Overview

### Purpose

`transformer_EE` serves as an alternative to traditional LSTM-based energy estimators, leveraging the capabilities of transformer encoders to predict:

* The energy of incoming neutrinos.
* The energy of outgoing leptons (e.g., muons).

This dual prediction is vital for analyzing both charged current (CC) and neutral current (NC) neutrino interactions.

### Input Data

The model processes reconstructed features from neutrino detectors, such as:

* Prong energies.
* Particle identification scores.
* Geometric and kinematic variables.

These inputs are typically stored in CSV files, with arrays represented as comma-separated strings.([GitHub][1])

---

## Model Architecture

At its core, `transformer_EE` employs a transformer encoder architecture, known for its self-attention mechanisms that capture complex dependencies within input data.
This is particularly beneficial for modeling the intricate patterns present in particle interactions.

---

## Code Structure

The repository is organized as follows:

* `transformer_ee/`: Contains the main source code, including model definitions, loss functions, and data loaders.
* `train_script.py`: Script to initiate model training.
* `batch_train_script.py`: Facilitates batch training processes.
* `model_export.py`: Handles exporting trained models for inference.
* `requirement.txt`: Lists all necessary Python packages.([GitHub][1])

---

## Configuration

Model configurations are managed via JSON files located in the `transformer_ee/config` directory.
Alternatively, configurations can be modified directly within the `train_script.py` file by adjusting the `input_d` dictionary.([GitHub][1])

---

## Logging and Monitoring

`transformer_EE` integrates with Weights & Biases (WandB) for experiment tracking.
To enable this feature:

1.
Install WandB:

   ```bash
   pip install wandb
   ```

([GitHub][1])

2.
Initialize WandB:

   ```bash
   wandb init
   ```



This setup allows for real-time monitoring of training metrics and model performance.([GitHub][1])

---

## Getting Started

### Prerequisites

Ensure the following are installed:

* Python 3.10 or higher.
* PyTorch 2.4.1 or higher.
* Pandas or Polars for data handling.
* NumPy.
* Matplotlib (version 3.5 or higher).([GitHub][1], [GitHub][2])

### Running the Model

To train the model:([GitHub][2])

```bash
python3 train_script.py
```



Ensure that your dataset is formatted correctly and placed in the appropriate directory.

---

## Applications in Physics

`transformer_EE` is designed for applications in neutrino physics, aiding in:

* Energy reconstruction in neutrino detectors.
* Event classification and selection.
* Improving the accuracy of neutrino oscillation measurements.([GitHub][1])

Its flexibility allows physicists to adapt the model to various experimental setups and data characteristics.([GitHub][1])

---

For more detailed information and updates, refer to the [transformer\_EE repository](https://github.com/wswxyq/transformer_EE).

---

[1]: https://github.com/wswxyq/transformer_EE
