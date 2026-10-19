

Looking at the change list, there are no breaking changes, new features, enhancements, bug fixes, or internal changes with doc impact. The existing file content is still accurate and requires no modifications.

<!--
SPDX-FileCopyrightText: Contributors to the Power Grid Model project <powergridmodel@lfenergy.org>

SPDX-License-Identifier: MPL-2.0
-->

# Components

The attributes of components are listed below.

## Base

* type name: `base`

The base type for all power-grid-model components.

#### Input

| name | data type | unit | description                                                                                                                                                       | required |        update        |
|------|-----------|------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------:|:--------------------:|
| `id` | `int32_t` | -    | ID of a component. The ID should be unique across all components within the same scenario, e.g., you cannot have a node with `id=5` and another line with `id=5`. | &#10004; | &#10060; (see below) |

If a component update is uniform and is updating all the elements with the same component type, IDs can be omitted or
set to `nan`s.
In any other case, the IDs need to be present in the update dataset, but cannot be changed.

Uniform component updates are ones that update the same number of component of the same type across scenarios.
By definition, a dense update is always uniform, hence the IDs can always be optional; whereas for a sparse update, the
IDs can only be optional if the index pointer is given as an arithmetic sequence of all elements (i.e. a sparse
representation of a dense buffer).
An example of the usage of optional IDs is given in [Power Flow Example](./Power%20Flow%20Example.ipynb)

#### Steady state output and Short circuit output

| name        | data type | unit | description                                                                                                                        |
|-------------|-----------|------|------------------------------------------------------------------------------------------------------------------------------------|
| `id`        | `int32_t` | -    | ID of a component, the ID should be unique across all components, e.g., you cannot have a node with `id=5` and a line with `id=5`. |
| `energized` | `int8_t`  | -    | Indicates if a component is energized, i.e. connected to a source                                                                  |

## Node

* type name: `node`
* base: [base](#base)

`node` is a point in the grid.
Physically a node can be a busbar, a joint, or other similar component.

#### Input

| name      | data type | unit     | description             | required |  update  | valid values |
|-----------|-----------|----------|-------------------------|:--------:|:--------:|:------------:|
| `u_rated` | `double`  | volt (V) | rated line-line voltage | &#10004; | &#10060; |    `> 0`     |

#### Steady state output

| name      | data type         | unit                       | description                                                                                     |
|-----------|-------------------|----------------------------|-------------------------------------------------------------------------------------------------|
| `u_pu`    | `RealValueOutput` | -                          | per-unit voltage magnitude                                                                      |
| `u_angle` | `RealValueOutput` | rad                        | voltage angle                                                                                   |
| `u`       | `RealValueOutput` | volt (V)                   | voltage magnitude, line-line for symmetric calculation, line-neutral for asymmetric calculation |
| `p`       | `RealValueOutput` | watt (W)                   | active power injection                                                                          |
| `q`       | `RealValueOutput` | volt-ampere-reactive (var) | reactive power injection                                                                        |

```{note}
The `p` and `q` output of injection follows the `generator` reference direction as mentioned in  
[Reference Direction](data-model.md#reference-direction)
```

#### Short circuit output

| name      | data type         | unit     | description                      |
|-----------|-------------------|----------|----------------------------------|
| `u_pu`    | `RealValueOutput` | -        | per-unit voltage magnitude       |
| `u_angle` | `RealValueOutput` | rad      | voltage angle                    |
| `u`       | `RealValueOutput` | volt (V) | voltage magnitude (line-neutral) |

## Branch

* type name: `branch`
* base: [base](#base)

`branch` is the abstract base type for the component which connects two *different* nodes.
For each branch two switches are always defined at from- and to-side of the branch.
In reality such switches may not exist.
For example, a cable usually permanently connects two joints.
In this case, the attribute `from_status` and `to_status` is always 1.

#### Input

| name          | data type | unit | description                    | required |  update  |  valid values   |
|---------------|-----------|------|--------------------------------|:--------:|:--------:|:---------------:|
| `from_node`   | `int32_t` | -    | ID of node at from-side        | &#10004; | &#10060; | a valid node ID |
| `to_node`     | `int32_t` | -    | ID of node at to-side          | &#10004; | &#10060; | a valid node ID |
| `from_status` | `int8_t`  | -    | connection status at from-side | &#10004; | &#10004; |   `0` or `1`    |
| `to_status`   | `int8_t`  | -    | connection status at to-side   | &#10004; | &#10004; |   `0` or `1`    |

#### Steady state output

| name      | data type         | unit                       | description                                                |
|-----------|-------------------|----------------------------|------------------------------------------------------------|
| `p_from`  | `RealValueOutput` | watt (W)                   | active power flowing into the branch at from-side          |
| `q_from`  | `RealValueOutput` | volt-ampere-reactive (var) | reactive power flowing into the branch at from-side        |
| `i_from`  | `RealValueOutput` | ampere (A)                 | magnitude of current at from-side                          |
| `s_from`  | `RealValueOutput` | volt-ampere (VA)           | apparent power flowing at from-side                        |
| `p_to`    | `RealValueOutput` | watt (W)                   | active power flowing into the branch at to-side            |
| `q_to`    | `RealValueOutput` | volt-ampere-reactive (var) | reactive power flowing into the branch at to-side          |
| `i_to`    | `RealValueOutput` | ampere (A)                 | magnitude of current at to-side                            |
| `s_to`    | `RealValueOutput` | volt-ampere (VA)           | apparent power flowing at to-side                          |
| `loading` | `double`          | -                          | relative loading of the branch, `1.0` meaning 100% loaded. |

#### Short circuit output

| name           | data type         | unit       | description                       |
|----------------|-------------------|------------|-----------------------------------|
| `i_from`       | `RealValueOutput` | ampere (A) | magnitude of current at from-side |
| `i_from_angle` | `RealValueOutput` | rad        | current angle at from-side        |
| `i_to`         | `RealValueOutput` | ampere (A) | magnitude of current at to-side   |
| `i_to_angle`   | `RealValueOutput` | rad        | current angle at to-side          |

### Line

* type name: `line`

`line` is a [branch](#branch) with specified serial impedance and shunt admittance.
A cable is also modeled as `line`.
A `line` can only connect two nodes with the same rated voltage.
If `i_n` is not provided, `loading` of line will be a `nan` value.

#### Input

| name   | data type | unit       | description                                        |                 required                  |  update  |            valid values            |
|--------|-----------|------------|----------------------------------------------------|:-----------------------------------------:|:--------:|:----------------------------------:|
| `r1`   | `double`  | ohm (Ω)    | positive-sequence serial resistance                |                 &#10004;                  | &#10060; | `r1` and `x1` cannot be both `0.0` |
| `x1`   | `double`  | ohm (Ω)    | positive-sequence serial reactance                 |                 &#10004;                  | &#10060; | `r1` and `x1` cannot be both `0.0` |
| `c1`   | `double`  | farad (F)  | positive-sequence shunt capacitance                |                 &#10004;                  | &#10060; |                                    |
| `tan1` | `double`  | -          | positive-sequence shunt loss factor (tan &#x03B4;) |                 &#10004;                  | &#10060; |                                    |
| `r0`   | `double`  | ohm (Ω)    | zero-sequence serial resistance                    | &#10024; only for asymmetric calculations | &#10060; | `r0` and `x0` cannot be both `0.0` |
| `x0`   | `double`  | ohm (Ω)    | zero-sequence serial reactance                     | &#10024; only for asymmetric calculations | &#10060; | `r0` and `x0` cannot be both `0.0` |
| `c0`   | `double`  | farad (F)  | zero-sequence shunt capacitance                    | &#10024; only for asymmetric calculations | &#10060; |                                    |
| `tan0` | `double`  | -          | zero-sequence shunt loss factor (tan &#x03B4;)     | &#10024; only for asymmetric calculations | &#10060; |                                    |
| `i_n`  | `double`  | ampere (A) | rated current                                      |                 &#10060;                  | &#10060; |               `> 0`                |

```{note}
In case of short circuit calculations, the zero-sequence parameters are required only if any of the faults in any of the
scenarios within a batch are not three-phase faults (i.e. `fault_type` is not `FaultType.three_phase`).
```

#### Electric Model

`line` is described by a $\pi$ model, where

$$
\begin{aligned}
   Z_{\text{series}} &= r + \mathrm{j}x \\
   Y_{\text{shunt}} &= 2 \pi fc (\tan \delta +\mathrm{j})
\end{aligned}
$$

### Link

* type name: `link`

`link` is a [branch](#branch) which usually represents a short internal cable/connection
between two busbars inside a substation.
It has a very high admittance (small impedance) which is set to a fixed per-unit value (equivalent to 10e6 siemens for
10kV network).
Therefore, it is chosen by design that no sensors can be coupled to a `link`.
There is no additional attribute for `link`.

#### Electric Model

`link` is modeled by a constant admittance $Y_{\text{series}}$, where

$$
Y_{\text{series}} = (1 + \mathrm{j}) \cdot 10^6 \,\mathrm{p.u.}
$$

### Transformer

`transformer` is a [branch](#branch) which connects two nodes with possibly different
voltage levels.
An example of usage of transformer is given in [Transformer Examples](../examples/Transformer%20Examples.ipynb)

#### Input

| name               | data type                                                   | unit             | description                                                                                                                                                                                                                 |                         required                         |  update  |                              valid values                              |
|--------------------|-------------------------------------------------------------|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------:|:--------:|:----------------------------------------------------------------------:|
| `u1`               | `double`                                                    | volt (V)         | rated voltage at from-side                                                                                                                                                                                                  |                         &#10004;                         | &#10060; |                                 `> 0`                                  |
| `u2`               | `double`                                                    | volt (V)         | rated voltage at to-side                                                                                                                                                                                                    |                         &#10004;                         | &#10060; |                                 `> 0`                                  |
| `sn`               | `double`                                                    | volt-ampere (VA) | rated power                                                                                                                                                                                                                 |                         &#10004;                         | &#10060; |                                 `> 0`                                  |
| `uk`               | `double`                                                    | -                | relative short circuit voltage, `0.1` means 10%                                                                                                                                                                             |                         &#10004;                         | &#10060; |                    `>= pk / sn` and `> 0` and `< 1`                    |
| `pk`               | `double`                                                    | watt (W)         | short circuit (copper) loss                                                                                                                                                                                                 |                         &#10004;                         | &#10060; |                                 `>= 0`                                 |
| `i0`               | `double`                                                    | -                | relative no-load (magnetizing) current                                                                                                                                                                                      |                         &#10004;                         | &#10060; |                         `>= p0 / sn` and `< 1`                         |
| `p0`               | `double`                                                    | watt (W)         | no-load (iron / magnetizing) loss                                                                                                                                                                                           |                         &#10004;                         | &#10060; |                                 `>= 0`                                 |
| `i0_zero_sequence` | `double`                                                    | -                | zero-sequence relative no-load (magnetizing) current                                                                                                                                                                        |                  &#10060; default `i0`                   | &#10060; |                  `>= p0_zero_sequence / sn` and `< 1`                  |
| `p0_zero_sequence` | `double`                                                    | watt (W)         | zero-sequence no-load (iron / magnetizing) loss                                                                                                                                                                             | &#10060; default `p0 + pk * (i0_zero_sequence^2 - i0^2)` | &#10060; |                                 `>= 0`                                 |
| `winding_from`     | {py:class}`WindingType <power_grid_model.enum.WindingType>` | -                | from-side winding type                                                                                                                                                                                                      |                         &#10004;                         | &#10060; |                                                                        |
| `winding_to`       | {py:class}`WindingType <power_grid_model.enum.WindingType>` | -                | to-side winding type                                                                                                                                                                                                        |                         &#10004;                         | &#10060; |                                                                        |
| `clock`            | `int8_t`                                                    | -                | clock number of phase shift. Even number is not possible if one side is Y(N) winding and the other side is not Y(N) winding. Odd number is not possible, if both sides are Y(N) winding or both sides are not Y(N) winding. |                         &#10004;                         | &#10060; |                          `>= -12` and `<= 12`                          |
| `tap_side`         | {py:class}`BranchSide <power_grid_model.enum.BranchSide>`   | -                | side of tap changer                                                                                                                                                                                                         |                         &#10004;                         | &#10060; |                                                                        |
| `tap_pos`          | `int8_t`                                                    | -                | current position of tap changer                                                                                                                                                                                             | &#10060; default `tap_nom`, if no `tap_nom` default `0`  | &#10004; | `(tap_min <= tap_pos <= tap_max)` or `(tap_min >= tap_pos >= tap_max)` |
| `tap_min`          | `int8_t`                                                    | -                | position of tap changer at minimum voltage                                                                                                                                                                                  |                         &#10004;                         | &#10060; |                                                                        |
| `tap_max`          | `int8_t`                                                    | -                | position of tap changer at maximum voltage                                                                                                                                                                                  |                         &#10004;                         | &#10060; |                                                                        |
| `tap_nom`          | `int8_t`                                                    | -                | nominal position of tap changer                                                                                                                                                                                             |                   &#10060; default `0`                   | &#10060; | `(tap_min <= tap_nom <= tap_max)` or `(tap_min >= tap_nom >= tap_max)` |
| `tap_size`         | `double`                                                    | volt (V)         | size of each tap of the tap changer                                                                                                                                                                                         |                         &#10004;                         | &#10060; |                                 `>= 0`                                 |
| `uk_min`           | `double`                                                    | -                | relative short circuit voltage at minimum tap                                                                                                                                                                               |              &#10060; default same as `uk`               | &#10060; |                  `>= pk_min / sn` and `> 0` and `< 1`                  |
| `uk_max`           | `double`                                                    | -                | relative short circuit voltage at maximum tap                                                                                                                                                                               |              &#10060; default same as `uk`               | &#10060; |                  `>= pk_max / sn` and `> 0` and `< 1`                  |
| `pk_min`           | `double`                                                    | watt (W)         | short circuit (copper) loss at minimum tap                                                                                                                                                                                  |              &#10060; default same as `pk`               | &#10060; |                                 `>= 0`                                 |
| `pk_max`           | `double`                                                    | watt (W)         | short circuit (copper) loss at maximum tap                                                                                                                                                                                  |              &#10060; default same as `pk`               | &#10060; |                                 `>= 0`                                 |
| `r_grounding_from` | `double`                                                    | ohm (Ω)          | grounding resistance at from-side, if relevant                                                                                                                                                                              |                   &#10060; default `0`                   | &#10060; |                                                                        |
| `x_grounding_from` | `double`                                                    | ohm (Ω)          | grounding reactance at from-side, if relevant                                                                                                                                                                               |                   &#10060; default `0`                   | &#10060; |                                                                        |
| `r_grounding_to`   | `double`                                                    | ohm (Ω)          | grounding resistance at to-side, if relevant                                                                                                                                                                                |                   &#10060; default `0`                   | &#10060; |                                                                        |
| `x_grounding_to`   | `double`                                                    | ohm (Ω)          | grounding reactance at to-side, if relevant                                                                                                                                                                                 |                   &#10060; default `0`                   | &#10060; |                                                                        |

```{note}
It can happen that `tap_min > tap_max`.
In this case the winding voltage is decreased if the tap position is increased.
```

```{note}
By default, PGM uses the same magnetization current (and thus impedance) for positive- and zero-sequence circuit.
This is typically not the case for 3-leg core-type transformers.
Due to lack of iron-core pass for zero-sequence flux,
the zero-sequence magnetization current is usually significantly higher than positive sequence.
If you want to do asymmetrical calculation with 3-leg core-type transformers,
please set the attribute `i0_zero_sequence`.
If the transformer specification does not provide such an attribute,
a good guess will be `i0_zero_sequence = 1.0`.
See [OpenDSS Documentation](https://opendss.epri.com/3-PhaseTransformerModeling.html)
for detailed explanation.
```

#### Electric Model

`transformer` is described by a $\pi$ model, where $Z_{\text{series}}$ can be computed as

$$
\begin{aligned}
    |Z_{\text{series}}| &= u_k p.u. \\
    \mathrm{Re}(Z_{\text{series}}) &= \frac{p_k}{s_n} p.u. \\
    \mathrm{Im}(Z_{\text{series}}) &= \sqrt{|Z_{\text{series}}|^2-\mathrm{Re}(Z_{\text{series}})^2} \\
\end{aligned}
$$
and $Y_{\text{shunt}}$ can be computed as

$$
\begin{aligned}
    |Y_{\text{shunt}}| &= i_0 p.u. \\
    \mathrm{Re}(Y_{\text{shunt}}) &= \frac{p_0}{s_n} p.u. \\
    \mathrm{Im}(Y_{\text{shunt}}) &= -\sqrt{|Y_{\text{shunt}}|^2-\mathrm{Re}(Y_{\text{shunt}})^2} \\
\end{aligned}
$$

### Generic Branch

* type name: `generic_branch`

`generic_branch` is a [branch](#branch) that connects two nodes, potentially at
different voltage levels.
Depending on the choice of parameters, it behaves either as a line or as a transformer.
The advantage is that the input parameters are based directly on the electrical equivalent circuit model.
The PI model can be used to avoid the need to convert parameters into transformer model data.
Another use case is modeling a line when connecting two nodes with approximately the same voltage levels (in that case,
the off-nominal ratio must be given to adapt the electrical parameters).

#### Input

| name    | data type | unit             | description                   |        required        |  update  | valid values |
|---------|-----------|------------------|-------------------------------|:----------------------:|:--------:|:------------:|
| `r1`    | `double`  | ohm              | positive-sequence resistance  |        &#10004;        | &#10060; |              |
| `x1`    | `double`  | ohm              | positive-sequence reactance   |        &#10004;        | &#10060; |              |
| `g1`    | `double`  | siemens          | positive-sequence conductance |        &#10004;        | &#10060; |              |
| `b1`    | `double`  | siemens          | positive-sequence susceptance |        &#10004;        | &#10060; |              |
| `k`     | `double`  | -                | off-nominal ratio             | &#10060; default `1.0` | &#10060; |    `> 0`     |
| `theta` | `double`  | radian           | angle shift                   | &#10060; default `0.0` | &#10060; |              |
| `sn`    | `double`  | volt-ampere (VA) | rated power                   | &#10060; default `0.0` | &#10060; |    `>= 0`    |

```{note}
The impedance (`r1`, `x1`) and admittance (`g1`, `b1`) attributes are calculated with reference to the "to" side of the
branch.
```

```{note}
To model a three-winding transformer using the `generic_branch`, the MVA method must be applied.
This means the user needs to calculate three equivalent `generic_branch` components and define an additional node to
represent the common winding connection point.
In rare cases, this method can result in negative electrical equivalent circuit elements.
Therefore, the input parameters are not checked for negative values.
```

```{warning}
The parameter `k` represents the **off-nominal ratio**, not the nominal voltage ratio.
This means that `k` must be explicitly provided by the user, particularly in cases involving a tap changer or a voltage
transformer.
The program does not automatically calculate `k` based on the nominal voltage levels of the connected nodes.
```

```{warning}
Asymmetric calculation is not supported for `generic_branch`.
```

#### Electric Model

`generic_branch` is described by a PI model, where

$$
\begin{aligned}
    Y_{\text{series}} &= \frac{1}{r + \mathrm{j}x} \\
    Y_{\text{shunt}} &=  g + \mathrm{j}b \\
    N &= k \cdot e^{\mathrm{j} \theta}
\end{aligned}
$$

### Asym Line

* type name: `asym_line`

`asym_line` is a [branch](#branch) with specified resistance and reactance per phase.
A cable can be modelled as `line` or `asym_line`. An `asym_line` can only connect two nodes with the same rated voltage.
If `i_n` is not provided, `loading` of line will be a `nan` value.
The `asym_line` denotes a 3 or 4 phase line with phases `a`, `b`, `c` and optionally `n` for neutral.

#### Input

The provided values will be converted to a matrix representing the line's properties per phase in the following form:

$$
   \begin{bmatrix}
      \text{aa} & \text{ba} & \text{ca} & \text{na}\\
      \text{ba} & \text{bb} & \text{cb} & \text{nb}\\
      \text{ca} & \text{cb} & \text{cc} & \text{nc}\\
      \text{na} & \text{nb} & \text{nc} & \text{nn}
   \end{bmatrix}
$$

This representation holds for all values `r_aa` ... `r_nn`, `x_aa` ... `x_nn` and `c_aa` ... `c_cc`.
If the neutral values are not provided, the last row and column from the above matrix are omitted.

| name   | data type | unit       | description                       | required                     |  update  | valid values |
|--------|-----------|------------|-----------------------------------|------------------------------|:--------:|:------------:|
| `r_aa` | `double`  | ohm (Ω)    | Series serial resistance aa       | &#10004;                     | &#10060; |    `> 0`     |
| `r_ba` | `double`  | ohm (Ω)    | Series serial resistance ba       | &#10004;                     | &#10060; |    `> 0`     |
| `r_bb` | `double`  | ohm (Ω)    | Series serial resistance bb       | &#10004;                     | &#10060; |    `> 0`     |
| `r_ca` | `double`  | ohm (Ω)    | Series serial resistance ca       | &#10004;                     | &#10060; |    `> 0`     |
| `r_cb` | `double`  | ohm (Ω)    | Series serial resistance cb       | &#10004;                     | &#10060; |    `> 0`     |
| `r_cc` | `double`  | ohm (Ω)    | Series serial resistance cc       | &#10004;                     | &#10060; |    `> 0`     |
| `r_na` | `double`  | ohm (Ω)    | Series serial resistance na       | &#10024; for a neutral phase | &#10060; |    `> 0`     |
| `r_nb` | `double`  | ohm (Ω)    | Series serial resistance nb       | &#10024; for a neutral phase | &#10060; |    `> 0`     |
| `r_nc` | `double`  | ohm (Ω)    | Series serial resistance nc       | &#10024; for a neutral phase | &#10060; |    `> 0`     |
| `r_nn` | `double`  | ohm (Ω)    | Series serial resistance nn       | &#10024; for a neutral phase | &#10060; |    `> 0`     |
| `x_aa` | `double`  | ohm (Ω)    | Series serial reactance aa        | &#10004;                     | &#10060; |    `> 0`     |
| `x_ba` | `double`  | ohm (Ω)    | Series serial reactance ba        | &#10004;                     | &#10060; |    `> 0`     |
| `x_bb` | `double`  | ohm (Ω)    | Series serial reactance bb        | &#10004;                     | &#10060; |    `> 0`     |
| `x_ca` | `double`  | ohm (Ω)    | Series serial reactance ca        | &#10004;                     | &#10060; |    `> 0`     |
| `x_cb` | `double`  | ohm (Ω)    | Series serial reactance cb        | &#10004;                     | &#10060; |    `> 0`     |
| `x_cc` | `double`  | ohm (Ω)    | Series serial reactance cc        | &#10004;                     | &#10060; |    `> 0`     |
| `x_na` | `double`  | ohm (Ω)    | Series serial reactance na        | &#10024; for a neutral phase | &#10060; |    `> 0`     |
| `x_nb` | `double`  | ohm (Ω)    | Series serial reactance nb        | &#10024; for a neutral phase | &#10060; |    `> 0`     |
| `x_nc` | `double`  | ohm (Ω)    | Series serial reactance nc        | &#10024; for a neutral phase | &#10060; |    `> 0`     |
| `x_nn` | `double`  | ohm (Ω)    | Series serial reactance nn        | &#10024; for a neutral phase | &#10060; |    `> 0`     |
| `c_aa` | `double`  | farad (F)  | Shunt nodal capacitance matrix aa | &#10024; for a full c matrix | &#10060; |    `> 0`     |
| `c_ba` | `double`  | farad (F)  | Shunt nodal capacitance matrix ba | &#10024; for a full c matrix | &#10060; |    `> 0`     |
| `c_bb` | `double`  | farad (F)  | Shunt nodal capacitance matrix bb | &#10024; for a full c matrix | &#10060; |    `> 0`     |
| `c_ca` | `double`  | farad (F)  | Shunt nodal capacitance matrix ca | &#10024; for a full c matrix | &#10060; |    `> 0`     |
| `c_cb` | `double`  | farad (F)  | Shunt nodal capacitance matrix cb |