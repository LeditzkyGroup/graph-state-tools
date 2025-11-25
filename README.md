# GraphState

A lightweight Python utility for constructing and analyzing qubit graph states.
```
# Example: 3-qubit line graph 1—2—3
adj = [
    [0, 1, 0],
    [1, 0, 1],
    [0, 1, 0]
]

G = GraphState(n=3, adj_mat=adj)

print("State vector:", G.state_vector)
print("Density matrix:", G.density_matrix)
print("Eigenvalues:", G.eigenvalues)
print("Stabilizers:", G.stabilizers)

print("Distance-2 MIS:", G._MIS2)

print("Max squared Schmidt coefficient:", G.max_schmidt_coeff())

# Fidelity with another graph state
H = GraphState(n=3, adj_mat=adj)
print("Fidelity:", G.fidelity_to(H))
```
