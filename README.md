# Graph State Tools

## Implementation

### Graph States

We implement uniformly weighted graph states in Python. We define a graph state using (1) $n$, the number of qubits, (2) the adjacency matrix of the graph state, (3) and $\phi$, the uniform weight of the graph state. We can demonstrate the I/O with a simple $3$-qubit triangle state.

```python
# Initialization
triangle = GraphState(n=3, adj_mat=[[0,1,1],[1,0,1],[1,1,0]], phi = math.pi)

# Get state vector
triangle.state_vector
```

**Output:**

```
array([ 0.35355339+0.00000000e+00j,  0.35355339+0.00000000e+00j,
        0.35355339+0.00000000e+00j, -0.35355339+4.32978028e-17j,
        0.35355339+0.00000000e+00j, -0.35355339+4.32978028e-17j,
       -0.35355339+4.32978028e-17j, -0.35355339+1.29893408e-16j])
```

Other attributes available include:

- `triangle.unitary` -- Unitary matrix representation of the graph state.
- `triangle.basis` -- Basis vectors of the graph state.
- `triangle.eigenvalues` -- Eigenvalues of the graph state, ordered corresponding to `triangle.basis`.
- `triangle.density_matrix` -- Density matrix representation in the computational basis.

The Python implementation is done with a dataclass called `GraphState`. A dataclass is just a regular class optimized for storing information rather than particularly complex initialization logic. Using a dataclass helps avoid writing boilerplate code and is easy to read.

### Graph State Unitary (`GraphState.unitary`)

A uniformly weighted graph state is defined as

$$\ket{G} = \prod_{(i,j)\ \in E} CP_{\phi}(i,j)\ket{+}^N.$$

The graph state unitary is defined as the unitary matrix $U$ for which

$$\ket{G} = U \ket{+}^{\otimes n}.$$

So,

$$U = \prod_{(i,j)\ \in E} CP_{\phi}(i,j) \mathbb{I}^{\otimes n}.$$

We can efficiently compute $U$ because $CP_{\phi}$ is diagonal in the computational basis. Note that $CP_{\phi}(i,j)$ applies a phase of $e^{i \phi}$ when both qubits $i$ and $j$ are in the $\ket{1}$ state. Starting with an identity matrix, we can iterate through each edge $(i,j)$ and multiply the diagonal entries corresponding to every computational basis state in which qubits $i$ and $j$ are both in the $\ket{1}$ state by $e^{i \phi}$. This procedure avoids explicit matrix multiplication.

### Graph State Vector (`GraphState.state_vector`)

The graph state vector is defined as

$$\ket{G} = \prod_{(i,j)\ \in E} CP_{\phi}(i,j)\ket{+}^N = U \ket{+}^{\otimes n},$$

where $U$ is the graph state unitary from the previous section. So, we can simply multiply $U$ by $\ket{+}^{\otimes n}$.

### Graph State Basis (`GraphState.basis`)

The set of states $\ket{W} = \sigma_Z^W |G\rangle$ forms a basis for $(\mathbb{C}^2)^{\otimes n}$. Note that $\sigma_Z^W \ket{+}^{\otimes n}$ is an orthonormal basis of pure product states, as well as that $\sigma_Z$ and $CP_{\phi}$ commute. The set of graph-state basis vectors $\{\ket{W}\}$ can be computed as

$$\ket{W} = U (\sigma_z^W \ket{+}^{\otimes n}),$$

where $U$ is the graph-state unitary and the operator $\sigma_z^W$ is defined as $\sigma_z^{w_1}\otimes\sigma_z^{w_2}\otimes\dots\otimes\sigma_z^{w_n}$, with $W = (w_1,w_2,\dots,w_n) \in \{0,1\}^n$.

### Graph State Density Matrix (`GraphState.density_matrix`)

The density matrix in the computational basis is simply computed as

$$\rho = \lvert G \rangle \langle G \rvert,$$

where $\ket{G}$ is the graph state vector.

### Graph State Eigenvalues (`GraphState.eigenvalues`)

The eigenvalues $\lambda_W$ are computed as

$$\lambda_W = \bra{W} \rho \ket{W},$$

where eigenvectors $\ket{W}$ are given from the basis and density matrix $\rho$ from the previous section.

### Graph State Stabilizers (`GraphState.stabilizers`)

For each vertex $a \in V$, the generator $K_a$ of the stabilizer $\mathcal{S}$ is computed as

$$K_a = \sigma_x^a \sigma_z^{N_a}.$$

Note that the graph state vector $\ket{G}$ is the unique, common eigenvector in $(\mathbb{C}^2)^V$ to the set of independent commuting observables $\{K_a | a \in V\}$.

### Graph State MIS2 (`GraphState.MIS2`)

MIS2 refers to the maximum independent set of distance $2$ in $G$. An independent set of distance $2$ contains no two vertices which are neighbors or share a neighbor. The MIS2 can be computed with the following ILP:

```
maximize    ∑ x[i]
such that   x[i] + x[j] ≤ 1  for all i,j such that  𝐀̄[i][j] = 1
```

Here, $\bar{A}$ is the off-diagonal terms of $A^2 + A$ (the diagonal is set to zero). The solution is the resulting subset of $x$ such that $x[i] = 1$.

### Fidelity Between Graph States (`GraphState.fidelity_to`)

Given another `GraphState` $H$, the fidelity is defined as

$$F(G,H) = \left| \left\langle G \mid H \right\rangle \right|^2$$.

### Graph State Maximum Schmidt Coefficient (`GraphState.max_schmidt_coeff`)

This property refers to the maximum (squared) Schmidt coefficient of the graph state over all bipartitions of the $n$-qubit system.

For a pure state $\ket{\psi}$, the Schmidt decomposition across a bipartition $A | B$ has coefficients ${ \lambda_i }$. The largest $\lambda_i^2$ corresponds to the largest eigenvalue of the reduced density matrix of either subsystem.
