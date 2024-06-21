---
title: "Consensus-based Weights"
---

# Consensus-based Weights

This guide describes how to use the **consensus-based weights** feature (also called "liquid alpha"). 

With this feature, the dividends to a subnet validator are better correlated to the performance of the subnet miner on which the subnet validator is setting the weights. In this context, see also the documentation for the [Commit Reveal](./commit-reveal.md) feature, as both these features help the subnet validators in finding new subnet miners that perform well and bonding to them quickly. 

## Technical blog

- Blog post: [Consensus-based Weights](https://blog.bittensor.com/consensus-based-weights-1c5bbb4e029b).
- Subtensor document section: [Validator bonding](https://github.com/opentensor/subtensor/blob/main/docs/consensus.md#validator-bonding).

## Description

Currently while calculating the dividends to a subnet validator, a quantity called $\Delta B_{ij}$, defined as an **instantaneous bond value** of a subnet validator $i$ with the subnet miner $j$, is used.

A subnet validator maintains an instantaneous bond value on each subnet miner. However, while calculating a subnet validator's dividends, instead of directly using the instantaneous bond value $\Delta B_{ij}$, we use $B_{ij}^{(t)}$, an **exponential moving average (EMA)** of the bond value, weighted over the current epoch and the previous epoch. See the below equation for how this EMA is computed, where $t$ is the current epoch time and $t-1$ is the previous epoch time:

$$
B_{ij}^{(t)} = \alpha\cdot\Delta B_{ij}^{(t)} + (1-\alpha)\cdot B_{ij}^{(t-1)}
$$

This EMA, $B_{ij}^{(t)}$, helps in early discovery of promising subnet miners and prevents abrupt changes to the bond value. Normally any abrupt change in the bond value indicates exploitation. 

Finally, the **dividend $D_i$ to a subnet validator $i$** is calculated as:

$$
D_i = \sum_j B_{ij} \cdot I_j
$$ 

where $B_{ij}$ is the EMA bond value of the subnet validator $i$ with the subnet miner $j$, and $I_j$ is the subnet miner's incentive. See the subtensor document section, [Validator bonding](https://github.com/opentensor/subtensor/blob/main/docs/consensus.md#validator-bonding) for a rigorous mathematical treatment of this topic. 

### What changed with this feature

Without the consensus-based weights feature, the $\alpha$ in the above equation is set to `0.9`. With the consensus-based weights feature, this $\alpha$ value is made into a variable. An optimium value for the variable $\alpha$ is determined based on the current consensus in a given subnet. 

Using the new subnet hyperparameters that are described below, a subnet owner should experiment and discover the optimum $\alpha$ for their subnet. 

## How to use consensus-based weights

Here are summary steps to use the consensus-based weights feature. These steps are typically executed by a subnet owner:

1. To activate this feature, a subnet owner should set the `liquid_alpha_enabled` (bool) hyperparameter to `True`.
2. Next, the subnet owner should set the upper and lower bounds for $\alpha$ by using the two subnet hyperparameters, `alpha_low` (int) and `alpha_high` (int). 

### Default values, allowed ranges and value format

#### Default values

- Default value for `alpha_low` is `0.7`.
- Default value for `alpha_high` is `0.9`.

#### Allowed ranges

- The range for both `alpha_low` and `alpha_high` hyperparameters is `(0,1)`.
- However, until further notice, the `alpha_high` value must be greater than or equal to `0.8`, and
- The value of `alpha_low` must not be greater than or equal to `alpha_high`.

#### Value format

When you set the subnet hyperparameters `alpha_low` and `alpha_high`, you must pass their integer equivalents in `u16`. This applies whether you set these hyperparameters using the `btcli` command or in your Python code. These integer values are then converted by the subtensor into their corresponding decimal values in the `u16` format. 

Use the below conversion formula to determine the integer values for your desired decimal values for both `alpha_low` and `alpha_high` hyperparameters.

$$ 
\text{Integer value} = \text{(your-desired-decimal-value)} \times 65536
$$

Hence, for example:
- If you want `alpha_low` to be `0.1`, then you would pass `6554`, which is the rounded up value of `0.1 * 65536`. 
- If you want `alpha_high` to be `0.8`, then you would pass `52429`, which is the rounded up value of `0.8 * 65536`.

## Using in Python code

### Method signatures

See below the Python definitions for the consensus-based weights feature:

```python
import bittensor as bt
    # Enable consensus-based weights (liquid alpha) feature
    enabled_result = bt.subtensor.set_hyperparameter(
        wallet=wallet,
        netuid=netuid,
        parameter="liquid_alpha_enabled",
        value=value,
        wait_for_inclusion=True,
        wait_for_finalization=True,
    )
    # Set alpha_low
    alpha_low_result = bt.subtensor.set_hyperparameter(
        wallet=wallet,
        netuid=netuid,
        parameter="alpha_low",
        value=value,
        wait_for_inclusion=True,
        wait_for_finalization=True,
    )
    # Set alpha_high
    alpha_high_result = bt.subtensor.set_hyperparameter(
        wallet=wallet,
        netuid=netuid,
        parameter="alpha_high",
        value=value,
        wait_for_inclusion=True,
        wait_for_finalization=True,
    )
```

### Example Python code

Below is the example Python code showing how to use the above definitions for the commit reveal feature:

```python
    import bittensor as bt
    # Enable consensus-based weights (liquid alpha) feature
    enabled_result = bt.subtensor.set_hyperparameter(
        wallet=wallet,
        netuid=netuid,
        parameter="liquid_alpha_enabled",
        value=True,
        wait_for_inclusion=True,
        wait_for_finalization=True,
    )
    # Set alpha_low
    alpha_low_result = bt.subtensor.set_hyperparameter(
        wallet=wallet,
        netuid=netuid,
        parameter="alpha_low",
        value="6554", # decimal 0.1
        wait_for_inclusion=True,
        wait_for_finalization=True,
    )
    # Set alpha_high
    alpha_high_result = bt.subtensor.set_hyperparameter(
        wallet=wallet,
        netuid=netuid,
        parameter="alpha_high",
        value="52429", # decimal 0.8
        wait_for_inclusion=True,
        wait_for_finalization=True,
    )
```

---

## Using `btcli`

### Set the subnet hyperparameters

#### 1. Enable the consensus-based weights feature

**Syntax**

```bash
btcli sudo set hyperparameters --netuid <NETUID> --param liquid_alpha_enabled --value <True or False>
```

**Example**

For subnet 1 (`netuid` of `1`):

```bash
btcli sudo set hyperparameters --netuid 1 --param liquid_alpha_enabled --value True
```

#### 2. Set the `alpha_low` and `alpha_high`

**Example**

Setting the value of `alpha_low` to the decimal `0.1` for subnet 1 (`netuid` of `1`):

```bash
btcli sudo set hyperparameters --netuid 1 --param alpha_low --value 6554
```

Setting the value of `alpha_high` to the decimal `0.8` for subnet 1 (`netuid` of `1`):

```bash
btcli sudo set hyperparameters --netuid 1 --param alpha_high --value 52429
```