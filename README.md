# PV Adversarial Detection

Physics-aware vs statistical anomaly detection for solar PV systems.

An LSTM autoencoder trained on normal PV sensor windows is a good fault detector until someone attacks it. This project shows how a gradient attack can hide a fault from the autoencoder, why a naive attack still gets caught by a physics check, and why an attack that respects the physics does not. It includes a simulator, the detector, three attacks, a physics-consistency check and a hybrid detector, plus a Streamlit app to run scenarios interactively.

## Method

- **Simulator.** A single-diode PV model. Irradiance and cell temperature go in, voltage, current and power come out, with P = V x I holding at every step. Windows are 24 time steps of five signals: G, T, V, I and P.
- **Faults.** Partial shading, V-sensor drift and inverter faults are injected into the sensor readings.
- **AI detector.** An LSTM autoencoder trained on normal windows only. A window raises an alarm when its reconstruction error is above a threshold set on normal data.
- **Attacks.** FGSM (one gradient step) and PGD (repeated small steps) change the faulty window within a limit epsilon so that the reconstruction error drops. Physics-constrained PGD projects the window back onto the physics after every step.
- **Physics check.** Flags a window if P differs from V x I by more than a set tolerance, or if the current leaves a band around the single-diode I-V curve.
- **Hybrid detector.** Alarm if the AI detector or the physics check fires.

## Project layout

- `app.py` - Streamlit app (simulation and batch evaluation)
- `pvcore.py` - simulator, fault injection, autoencoder, attacks, physics checks, evaluation
- `notebooks/pv_simulated_env.ipynb` - base study on the simulated environment
- `notebooks/pv_advanced.ipynb` - multi-seed runs, baselines, adversarial training
- `notebooks/pv_real_data.ipynb` - experiments on a real labelled PV dataset

## Running the app

```
pip install -r requirements.txt
streamlit run app.py
```

The first launch trains the autoencoder (about 1-2 minutes) and saves it to `pv_models/`. Later launches load it directly.

The sidebar can also use your own weather CSVs (train and test files) instead of the synthetic data. Set the two file paths in the sidebar. The notebooks read their CSVs from `DATA_DIR`, defined in the first code cell, so change that to wherever your files are.

## Using the app

1. Pick a fault and leave the attack on "No attack". The AI detector and the physics check should both alarm.
2. Pick FGSM or PGD. The AI detector can be fooled, but the data is no longer physically consistent, so the physics check still alarms.
3. Pick physics-constrained PGD. The data stays consistent with the physics, so the physics check stays quiet.
4. Use the batch evaluation tab to run many scenarios and compare detection rates, with and without attack, for each detector.

Report numbers from your own runs of the app and notebooks.

## Limitations

- V, I and P are simulated and the faults are injected.
- The attacker is white-box and controls all five signals.
- The physics tolerances are chosen by hand.

## Future work

Adversarial training, per-site thresholds, real measured V and I data, tighter physics checks.
