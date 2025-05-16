# Homophilic Networks

**Author**: <a href='https://www.gesis.org/person/fariba.karimi'>Fariba Karimi</a>

**Version Date**: 09 June 2020

**Description**: Codes and notebooks related to generating homophilic networks and their properties.

**License**: <a href='https://creativecommons.org/licenses/by/4.0/'>Creative Commons (CC BY 4.0)</a>

[![Binder](https://notebooks.gesis.org/binder/badge.svg)](https://notebooks.gesis.org/binder/v2/gh/gesiscss/homophilic_networks/master)

# Change to Karimi Code

## 🔁 Update to `_pick_targets`: Ensuring Determinism and Reproducibility

In the original Karimi et al. model for building homophilic Barabási–Albert (BA) networks, the `_pick_targets` function selects existing nodes to connect to a new node. This is done using **preferential attachment** influenced by **homophily**.

### 🧪 Original Behavior

For each edge a new node wants to add:
- A random number is drawn.
- The new node may or may not connect to a candidate, depending on whether the draw falls below that node’s *unnormalized probability*.
- If no connection is made, the process retries up to a certain limit.

**Issue**:  
- The process is non-deterministic, even with fixed seeds.
- The probability dictionary does **not** update after each successful edge addition.
- Skipped nodes distort the overall target probability distribution.

See *Iteration 1* and *Iteration 2* outputs for illustrations of this behavior.

---

### ✅ Our Update: Normalized Sampling with Deterministic Selection

We updated `_pick_targets` to ensure:
- All `m` targets are selected without retries or failed rolls.
- Probabilities are **normalized** (sum to 1).
- A random number is drawn from \([0, 1]\) and mapped to a cumulative probability range (i.e., divide the unit interval into `len(targets)` regions).
- Each draw always selects a valid target.

This guarantees that:
- **Exactly `m` targets** are selected for each new node.
- The outcome is **fully reproducible** with a fixed RNG.
- Nodes are selected fairly based on up-to-date homophily-weighted preferential attachment.

---

### 📌 Why This Matters

This update makes the graph generation:
- Deterministic under fixed seeds.
- Free from bias due to repeated failed attempts.
- Easier to debug, explain, and visualize.

```python
# Core change
for k in target_prob_dict_copy:
    target_prob_dict_copy[k] /= prob_sum  # Normalize
...
rand_num = rng.random()
cumsum = 0
for k in target_list_copy:
    cumsum += target_prob_dict_copy[k]
    if rand_num < cumsum:
        targets.add(k)
        ...
