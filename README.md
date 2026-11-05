# AI-Powered Clock Tree Synthesis (CTS)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/BOMMUGOWTHAMSAIREDDY97/CTS/blob/main/CTS.ipynb)

A Python-based simulation and optimization framework for **Clock Tree Synthesis (CTS)** in VLSI physical design, leveraging **Genetic Algorithms (GA)** to synthesize and optimize clock networks for minimum clock skew, dynamic power, and wirelength.

---

## 📌 Overview

**Clock Tree Synthesis (CTS)** is one of the most critical stages in the ASIC/VLSI physical design flow. Its primary objective is to distribute the clock signal from a single clock source to thousands of sequential clock sinks (flip-flops, registers) with:
- **Zero or Minimum Clock Skew**: Ensuring synchronous data capture across registers without setup or hold timing violations.
- **Controlled Insertion Delay (Latency)**: Minimizing the time taken for the clock signal to propagate from root to sinks.
- **Low Power Consumption**: Clock distribution networks typically account for 30%–40% of total chip dynamic power.
- **Minimal Routing Wirelength**: Reducing interconnect congestion and capacitive load.

This project implements an **AI-driven evolutionary approach** that optimizes buffer placement, buffer sizing, and tree topology on a simulated 2D chip floorplan.

---

## ⚙️ Architecture & Modeling

The project models the core electrical and physical elements of a clock distribution network:

### 1. Clock Sink (`ClockSink`)
Represents the clock pin of a sequential element (e.g., standard cell D flip-flop):
- **Coordinates `(x, y)`**: Spatial location within the floorplan grid (`0` to `50` $\mu$m).
- **Pin Capacitance**: Load capacitance varying from `0.1 pF` to `1.0 pF`.
- **Path & Delay Tracking**: Cumulative propagation delay from the clock root.

### 2. Clock Buffer (`Buffer`)
Models clock inverters/buffers inserted to boost clock slews and balance propagation delays:
- **Drive Strength (`1x`, `2x`, `4x`)**: Selectable drive sizing.
- **Input Capacitance**: Modeled proportionally as `0.1 * drive_strength` pF.
- **Buffer Delay**: Modeled inversely proportional to drive strength: `delay = 10 / drive_strength` ps.
- **Fanout Tree**: Direct hierarchical connections to child buffers or sink pins.

### 3. Clock Tree (`ClockTree`)
Represents the complete hierarchical clock distribution network:
- **Skew Calculation**:
  ```text
  Skew = max(Sink Delays) - min(Sink Delays)
  ```
- **Dynamic Power Model**:
  ```text
  P_dynamic = 0.5 * C_total * Vdd^2 * f
  ```
  Evaluates power dissipated across both sink capacitive loads and buffer stage fanouts.
- **Routing Wirelength**: Calculates cumulative Manhattan/Euclidean routing lengths across buffer-to-buffer and buffer-to-sink interconnects.

---

## 🧬 Genetic Algorithm Optimization

The optimization engine employs evolutionary computation to explore the discrete combinational space of buffer placement, sizing, and clustering:

### Evolutionary Cycle
```
[ Random Population ] ──> [ Delay & Skew Evaluation ] ──> [ Multi-Objective Fitness ]
         ▲                                                               │
         │                                                               ▼
[ Mutation Operators ] <── [ Offspring Cloning ] <── [ Tournament Selection (k=3) ]
```

### Multi-Objective Fitness Evaluation
The fitness function simultaneously balances timing, power, and routing metrics:
```text
Fitness = 1 / (1 + Skew_Penalty^2 + Power_Penalty^2 + Wirelength / 100)
```
- **Skew Penalty**: Penalizes clock skew exceeding target threshold (`SKEW_THRESHOLD = 100 ps`).
- **Power Penalty**: Penalizes dynamic power exceeding threshold (`POWER_THRESHOLD = 15 mW`).
- **Wirelength Term**: Regularizes interconnect length to avoid redundant wire detours.

### Mutation Operators
During each generation, candidate clock trees undergo targeted mutations:
1. **`add_buffer`**: Inserts a new buffer into the network, dynamically clustering nearby sinks and linking to the buffer hierarchy.
2. **`move_buffer`**: Perturbs buffer physical coordinates `(x ± Δx, y ± Δy)` to rebalance path delays.
3. **`change_strength`**: Resizes buffer drive strengths (`1x` $\leftrightarrow$ `2x` $\leftrightarrow$ `4x`) to trade off delay against capacitive load.

---

## 📊 Visualizations & Outputs

The notebook generates two key visual reports:

1. **Optimized 2D Clock Tree Floorplan**:
   - **Red Circles (`ro`)**: Clock sinks with their pin capacitance annotations (e.g., `0.5pF`).
   - **Blue Squares (`bs`)**: Clock buffers with drive strength annotations (e.g., `2x`, `4x`).
   - **Blue Lines**: Buffer-to-buffer distribution backbone routing.
   - **Red Lines**: Final-stage buffer-to-sink leaf routing.
   - **Title Metrics**: Displays final optimized clock skew (ps) and total dynamic power (mW).

2. **Fitness Progression Convergence Curve**:
   - Plots average population fitness across 30 generations to observe convergence and stability.

---

## 🚀 Getting Started

### Run via Google Colab (Recommended)
Click the badge above or navigate to [`CTS.ipynb`](https://github.com/BOMMUGOWTHAMSAIREDDY97/CTS/blob/main/CTS.ipynb) and select **Open in Colab** to execute the simulation interactively in your browser.

### Run Locally

1. **Clone the repository**:
   ```bash
   git clone https://github.com/BOMMUGOWTHAMSAIREDDY97/CTS.git
   cd CTS
   ```

2. **Install dependencies**:
   ```bash
   pip install numpy matplotlib jupyter
   ```

3. **Launch the Jupyter Notebook**:
   ```bash
   jupyter notebook CTS.ipynb
   ```

---

## 📁 Repository Structure

```
CTS/
├── CTS.ipynb        # Jupyter notebook with complete CTS model, Genetic Optimizer & visualizer
└── README.md        # Technical project summary and documentation
```