## Random Quantum Circuits in quantum supremacy demonstration

### Purpose

This dataset contains:
 * definitions of the Random Quantum Circuits (RQCs) used in our quantum
   supremacy demonstration,
 * lists of the bitstrings observed in the experimental executions of the
   corresponding circuits on the Sycamore processor,
 * lists of output state amplitudes computed by a quantum circuit simulator,
 * a table of system parameters by qubit in CSV format,
 * fidelity values plotted in figure 4 of [1].

See [1] and [2] for more details about the experiment.

### Background

#### Circuit parameters

RQCs used in the quantum supremacy demonostration are uniquely identified
using five parameters:
 * n: number of qubits (12, 14, ... 38, 39, ... 51, 53),
 * m: number of cycles (12, 14, 16, 18, 20),
 * s: seed for the pseudo-random number generator (0, 1, ..., 9),
 * e: number of elided gates (0, 6, 8, 11, 12, 15, 19, 22),
 * p: sequence of coupler activation patterns (EFGH, ABCDCDAB).

See figure S25 in [2] for the coupler activation patterns and section VII
for other details about the circuit parameters.

#### Circuit types

Circuit types listed in Table III in section VII in [2] can be identified
as follows:
 * simplifiable circuits: p=EFGH,
 * non-simplifiable circuits: p=ABCDCDAB,
 * full circuits: e=0,
 * elided circuits: e > 0,
 * patch circuits: not included.

Patch circuits are easily derived from full or elided circuits by removing
all two-qubit gates on the cut shown in red on figure S27 in [2].

For more details see section VII in [2].

### Content description

For each RQC there are four or five files in the dataset:
 * original RQC specification in QSIM format, named "circuit_\*.qsim",
 * derived RQC specificaton as python code using cirq, named "circuit_\*.py",
 * derived RQC specification in QASM format, named "circuit_\*.qasm",
 * bitstrings observed in experiments, named "measurements_\*.txt",
 * output state amplitudes, named "amplitudes_\*.txt" (not available for
   all circuits).

The asterisk \* in the names above stands for a string specifying all five
parameters identifying a RQC. For example, "circuit_n53_m14_s0_e0_pEFGH.qasm"
contains the definition of the 53-qubit, 14-cycle RQC with PRNG seed 0, no
elided gates and simplifiable sequence of coupler activation patterns
(i.e. EFGH) in the QASM format.

Files are grouped by parameters n and m into compressed tarballs. For example,
tarball "n50_m14.tar.gz" contains all RQCs and measurement files for circuits
with 50 qubits and 14 cycles.

### Circuit file formats

#### QSIM format

First line specifies the number of qubits n. Each subsequent line specifies
a single gate and consists of moment number, gate name and one or two qubits
as a number in 0..n-1 optionally followed by gate parameters.

The circuits use the following gates:
 * x_1_2: parameter-free, single-qubit pi/2 rotation around the X axis of the
   Bloch sphere, see equation (50) in section VII of [2],
 * y_1_2: parameter-free, single-qubit pi/2 rotation around the Y axis of the
   Bloch sphere, see equation (51) in section VII of [2],
 * hz_1_2: parameter-free, single-qubit pi/2 rotation around the X+Y axis of
   the Bloch sphere, see equation (52) in section VII of [2],
 * rz: single-qubit rotation around the Z axis of the Bloch sphere through the
   angle specified in radians by the gate's sole parameter,
 * fsim: two-qubit gate corresponding to the composition of the iSWAP and CPHASE
   gates and taking two parameters in radians: theta (the negative iSWAP angle)
   and phi (the CPHASE angle), see equation (53) in section VII of [2].

Note that the two-qubit gates executed in our experiments on Sycamore belong
to the five-parameter family of two-qubit gates that preserve the number of
0 and 1 states of the qubits. Each such gate can be decomposed into one fsim
gate and four rz gates. Therefore, one cycle consisting of one application of
single-qubit gates and one application of two-qubit gates is represented in the
file using four moments. The first moment contains x_1_2, y_1_2 and hz_1_2 gates.
The other three moments use rz and fsim gates to describe the two-qubit gates
used in the experiments. See section VII in [2] for more details about the
Sycamore gates and their decomposition.

Qubits are specified as numbers in 0..n-1 and hence do not directly indicate
qubit location on the device.

#### Python/cirq format

Each python file defines two variables: QUBIT_ORDER and CIRCUIT. The former is
a python list object containing `cirq.GridQubit` objects initialized with the
row and column of each qubit on the device. The latter is a `cirq.Circuit`
object initialized with all gate operations contained in the circuit. The
files have been tested using cirq-dev version 0.7.0.dev20191122052011.

The following code snippet illustrates how one can use cirq and our circuit
definitions to compute output state amplitudes:

```
$ python -i circuit_n16_m14_s0_e0_pEFGH.py
>>> cirq.final_wavefunction(CIRCUIT, qubit_order=QUBIT_ORDER)
array([ 0.00263724+0.00337646j,  0.0009332 +0.00111853j,
       -0.0007809 +0.00386362j, ...,  0.00574739-0.00027827j,
       -0.00254766+0.00299345j,  0.00396056+0.00312335j], dtype=complex64)
```

See [3] for more details about cirq.

#### QASM format

Each QASM file has been generated using cirq and specifies the RQC decomposed
into CNOT and single-qubit gates. The files have been generated using cirq.

See [4] for more details about the format.

### Measurements file format

Each line contains the bitstring obtained in a single execution of the RQC on
Sycamore. The first, left-most position corresponds to the qubit 0 in QSIM
format and the first qubit in the QUBIT_ORDER list in the python/cirq files.
The number of lines in a measurement file is equal to the number of samples
taken in the corresponding experiment on Sycamore.

### Amplitudes file format

Each line contains a bitstring observed in an experiment and the real and
imaginary parts of the corresponding output state amplitude separated by one
or more whitespace characters. The amplitudes have been computed by simulating
the corresponding RQC on a classical computer. Note that this data is not
available for all circuits.

### Fidelity

The dataset also includes two data files in CSV format with fidelity estimates
plotted in figure 4 of [1]. Fidelity of patch circuits has been estimated as
the product of log XEB estimates for two circuit partitions. Fidelity of elided
and full circuits has been estimated using linear XEB due to its somewhat lower
variance in low fidelity regime. The files also include 5-sigma confidence level
which accounts for systematic and statistical uncertainties. Finally, the files
contain fidelity prediction computed using the model described in VIII E of [2].

### Content listing

The dataset includes the following files:

```
n12_m14.tar.gz (100 files)
n14_m14.tar.gz (100 files)
n16_m14.tar.gz (100 files)
n18_m14.tar.gz (100 files)
n20_m14.tar.gz (100 files)
n22_m14.tar.gz (100 files)
n24_m14.tar.gz (100 files)
n26_m14.tar.gz (100 files)
n28_m14.tar.gz (100 files)
n30_m14.tar.gz (100 files)
n32_m14.tar.gz (100 files)
n34_m14.tar.gz (100 files)
n36_m14.tar.gz (100 files)
n38_m14.tar.gz (100 files)
n39_m14.tar.gz (90 files)
n40_m14.tar.gz (90 files)
n41_m14.tar.gz (90 files)
n42_m14.tar.gz (90 files)
n43_m14.tar.gz (90 files)
n44_m14.tar.gz (90 files)
n45_m14.tar.gz (90 files)
n46_m14.tar.gz (90 files)
n47_m14.tar.gz (90 files)
n48_m14.tar.gz (90 files)
n49_m14.tar.gz (90 files)
n50_m14.tar.gz (90 files)
n51_m14.tar.gz (90 files)
n53_m12.tar.gz (90 files)
n53_m14.tar.gz (190 files)
n53_m16.tar.gz (90 files)
n53_m18.tar.gz (90 files)
n53_m20.tar.gz (90 files)
som_params_by_qubit.csv
fidelity_4a.csv
fidelity_4b.csv
README.md
```

### References

[1] F. Arute *et. al*, "Quantum supremacy using a programmable
    superconducting processor", Nature **574**, 505 (2019),
    https://doi.org/10.1038/s41586-019-1666-5

[2] F. Arute *et. al*, Supplementary information for “Quantum
    supremacy using a programmable superconducting processor”, Nature **574**, 505 (2019),
    https://doi.org/10.1038/s41586-019-1666-5, preprint: arXiv:1910.11333

[3] Cirq: A Python framework for creating, editing, and invoking Noisy
    Intermediate Scale Quantum (NISQ) circuits,
    https://github.com/quantumlib/Cirq

[4] Cross, Andrew W.; Bishop, Lev S.; Smolin, John A.; Gambetta, Jay M. "Open
    Quantum Assembly Language", arXiv:1707.03429
