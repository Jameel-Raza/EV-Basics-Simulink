# Project Overview

## Objective
This Simulink model demonstrates basic cell-level battery behavior and a simple battery management strategy relevant to EV systems.

## Concepts Covered
- Core electrical / EV concepts (voltage, current, state of charge)
- Basic system and controller behavior under nominal conditions

## Tools Required
- MATLAB
- Simulink
- Simscape Electrical (optional) or Power Systems libraries

## How to Run
1. Open MATLAB.
2. Set the Current Folder to this project directory.
3. Open the Simulink model file (`.slx`).
4. Click **Run**.
5. Observe scopes, logs, and output signals.

## Expected Behavior
Users should observe voltage and current trends, changes in state of charge (SOC), and controller responses; exact plots depend on the model configuration.

## Status
Draft — Under Review

## Notes
This model is intended for academic learning; it may be simplified and should be validated before use in production or research results.

---

# Active Cell BMS

## Overview
The Active Cell Battery Management System (BMS) model in Simulink represents a more advanced battery management strategy, focusing on individual cell monitoring and management within a battery pack. This model is relevant for understanding and designing sophisticated BMS solutions for electric vehicles (EVs) and other battery-operated systems.

## Objective
To simulate and analyze the behavior of an active cell BMS, which includes cell balancing, state of charge (SOC) estimation, and fault detection.

## Concepts Covered
- Cell-level voltage and temperature monitoring
- Active and passive cell balancing techniques
- SOC and state of health (SOH) estimation algorithms
- Fault detection and management strategies

## Tools Required
- MATLAB
- Simulink
- Simscape Electrical
- Control System Toolbox (for advanced control strategies)

## How to Run
1. Open MATLAB.
2. Set the Current Folder to this project directory.
3. Open the Simulink model file for the Active Cell BMS (`.slx`).
4. Configure the model parameters as needed for your study.
5. Click **Run** to simulate the model.
6. Analyze the results through scopes, logs, and output signals.

## Expected Behavior
The model should exhibit realistic BMS behavior, including cell balancing actions, SOC changes, and responses to simulated faults. Users can expect to see how an active BMS maintains battery performance and safety.

## Status
Draft — Under Review

## Notes
This model is intended for academic and research purposes. It provides a foundation for developing and testing advanced BMS algorithms and strategies. Validation with real battery data is essential before any practical application.