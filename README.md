# Convergence Analysis: FEM vs. Navier Solution — SSSS Thin Plate (Steel)

## 1. Introduction
This repository contains a validation study comparing Finite Element Method (FEM) and the Navier analytical solution for a thin, square, simply-supported steel plate (SSSS) subjected to uniform transverse loading.
The goal is to verify the FEM code against the benchmark analytical solution, observing convergence behavior as the mesh size or series truncation is refined.

---

## 2. Problem Setup

### Plate Geometry & Material
| Parameter        | Value       | Unit          | Description |
|------------------|------------|--------------|-------------|
| a                | 1.0        | m            | Plate side length |
| t                | 0.01       | m            | Plate thickness |
| E                | 210e9      | Pa           | Young’s modulus (Steel) |
| ν                | 0.3        | –            | Poisson’s ratio |
| q                | 1000       | N/m²         | Uniform transverse load |

---

### Flexural Rigidity
D = E t³ / [12(1 - ν²)]

Substituting values:

D = 210×10⁹ × (0.01)³ / [12(1 - 0.3²)] = 19230.77 Nm

---

## 3. Governing Equation
Thin plate bending equation:

D ∇⁴ w = q

Boundary conditions for SSSS:

w = 0, Mₙ = 0 on all edges

---

## 4. Navier Analytical Solution
For an SSSS plate, the Navier solution is:

w(x, y) = Σ_{m=1,3,...} Σ_{n=1,3,...} W_{mn} sin(mπx/a) sin(nπy/a)

W_{mn} = 16 q / [π⁶ D m² n² (m²/a² + n²/a²)²]

Exact normalized deflection: w_max D / (q a⁴) = 0.00406235

---

## 5. Finite Element Method (FEM) Implementation

- Kirchhoff Plate Bending Element (Q4, C¹ continuity)
- DOFs per node: { w, θₓ, θᵧ }
- Shape functions: Hermite polynomials
- Stiffness matrix from bending energy functional
- Load vector from uniform q
- Simply supported BCs applied explicitly

Mesh sizes tested: n = 8, 10, 12, 16, 20, 24, 32, 40

---

## 6. Results
See images in repo:

1.png — Convergence plot
2.png — FEM deformation 3D
3.png — Deflection contour

---

## 7. Conclusion
- FEM and Navier solutions converge to the same deflection value.
- FEM starts slightly underestimating; Navier starts slightly overestimating.
- Both are valid for thin plate bending verification.

---

## 8. Repository Structure
📂 FEM-vs-Navier-SSSS-Plate
 ├── README.md
 ├── 1.png   # Convergence plot (add manually)
 ├── 2.png   # FEM deformation (add manually)
 ├── 3.png   # Deflection contour (add manually)
 ├── FEM_code/
 ├── Navier_code/
 └── data/
