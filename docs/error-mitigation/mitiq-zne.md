# Zero-Noise Extrapolation

!!! info "Note"
    This feature is present only in openqaoa 0.2.5 and above!

Zero-Noise Extrapolation is an error mitigation technique which infers the effects of noise on the expectation value of an observable. 

Given a circuit $U$ and an observable $X$, this technique creates $n$ replicas of $U$, functionally equivalent to $U$, but each one having increased depth. These are called $\textbf{scaled circuits}$. The depth of each of the $0 \leq j <n$ scaled circuit is defined by its own $j$ noise level $\lambda_j$, therefore, each $j$ scaled circuit is affected by greater noise level. We keep track of noise levels $\{\lambda\}_{j=0}^{n}$ and the corresponding expectation values for $X$, i.e., $\{\hat{\mu}\}_{j=0}^{n}$. At this point, we can infer the expectation value corresponding to a zero-noise level, i.e., the correct expectation value when no noise is present.


![ZNE_workflow](/img/ZNE_workflow.png)
This figure shows a general workflow for ZNE, and it is based on Ref. 1.

### The Algorithm

This routine exploits the Mitiq framework, that provides a number of error mitigation techniques. In particular, the workflow is the following:

1. Decide error mitigation hyperparameters to control how gates are duplicated, what levels of noise to test, and how to extrapolate zero-noise expectation value.
2. At every optimization step of your QAOA algorithm, automatically the next actions would be performed:
    - get the current status of the $\textit{Ansatz}$, with the current values of the parameters as they are found by the optimizer
    - produce $n$ replicas of this circuit, based on your choice of hyperparameters
    - run this circuit and estimate expectation value for the cost hamiltonian
    - extrapolate zero-noise expectation value, which will be consider by QAOA optimizer to update search parameters

### How to do this within OpenQAOA?

In the workflows, this can be implemented as:
```Python
q = QAOA()
q.set_error_mitigation_properties(
    error_mitigation_technique='mitiq_zne',
    factory = 'Richardson', 
    scale_factors = [1,2,3]
)
```

Internally, OpenQAOA modifies the backend object to exploit the Mitiq framework and perform the gate duplication and the extrapolation:
```Python
if self.error_mitigation_properties.error_mitigation_technique == "mitiq_zne":
    self.backend = ZNEWrapper(
        backend=self.backend,
        factory=self.error_mitigation_properties.factory,
        scaling=self.error_mitigation_properties.scaling,
        scale_factors=self.error_mitigation_properties.scale_factors,
        order=self.error_mitigation_properties.order,
        steps=self.error_mitigation_properties.steps
    )
```

### What to expect?
Let's see the technique in practice by solving a MaxCut Problem for $n=6$ qubits. In particular, the instance of the problem is the following graph:

![MAXCUT_plot](/img/MAXCUT_plot.png)

We compare the evolution of the cost function as the number of function evaluations increases, in three cases:

- Noiseless environment, with the `qiskit.shot_simulator` (blue plot line).
- Noisy environment without ZNE, using a noise model with one-qubit and two-qubit depolarizing errors, based on IBM Quebec quantum computer (error probabilities retrieved on Jan 19, 2024) (red plot line). 
- Noisy environment with ZNE, using the same noise model as 2 (green plot line).

![ZNE_plot](/img/ZNE_plot.png)

## References
1. [Zero-Noise Extrapolation - Mitiq Documentation](https://mitiq.readthedocs.io/en/stable/guide/zne.html)
2. [Bultrini, D., et al.](https://doi.org/10.22331/q-2023-06-06-1034), Unifying and benchmarking state-of-the-art quantum error mitigation techniques, Quantum 7 (2023): 1034. 
3. [LaRose, R., et al.](https://doi.org/10.22331/q-2022-08-11-774), Mitiq: A software package for error mitigation on noisy quantum computers, Quantum 6 (2022): 774.

