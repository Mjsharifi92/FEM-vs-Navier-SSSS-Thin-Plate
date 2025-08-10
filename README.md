# Convergence Analysis: FEM vs. Navier Solution — SSSS Thin Plate (Steel)

## Abstract
This repository provides a comprehensive **convergence analysis** comparing a **Kirchhoff–Love thin-plate finite element method (FEM)** with the **classical Navier double-sine series** solution for a **square, simply-supported (SSSS) steel plate** under a **uniform transverse load**.  
The work explores **mesh refinement studies** for FEM and **series truncation studies** for the analytical Navier solution, evaluating how each approaches the exact non-dimensional central deflection constant.  
Results reveal a characteristic convergence pattern: FEM solutions begin below the analytical reference (numerical stiffness) and increase with mesh refinement, while Navier series results start above the reference (due to truncation bias) and decrease toward the exact value.  
This repository serves as a **verification benchmark** for thin-plate FEM implementations before tackling more complex scenarios such as **orthotropic materials, thick plates, or nonlinear effects**.

**Keywords:** Kirchhoff–Love plate theory, Navier double sine series, SSSS thin plate deflection, mesh refinement, series truncation, C¹ Hermite finite elements, steel plate bending, FEM convergence.

---

## 1) Problem Definition  
*(Keywords: SSSS thin plate, Kirchhoff–Love theory, flexural rigidity, plate bending equation, bending moments)*

We consider a **square, thin steel plate** of side length $a$ and thickness $t \ll a$, subject to a **uniform transverse load** $q$. The plate edges are **simply supported on all four sides** (SSSS), meaning both deflection and bending moment normal to the edge vanish.  
The material is structural steel with Young’s modulus $E = 210$ GPa and Poisson’s ratio $\nu = 0.3$. The flexural rigidity, a key parameter in plate bending, is given by:

$$
D = \frac{E t^3}{12(1-\nu^2)}
$$

For $t=0.01$ m, we obtain $D = 1.9230 \times 10^4$ N·m. This parameter controls how resistant the plate is to bending under the given load.

The **Kirchhoff–Love plate theory** is adopted, which assumes that normals to the mid-surface before deformation remain straight and normal after deformation, and that transverse shear deformation is negligible. The governing PDE for an isotropic thin plate under uniform load is:

$$
D\,\nabla^4 w = q
$$

where $w(x,y)$ is the transverse deflection. The SSSS boundary conditions are:

$$
w = 0, \quad M_n = 0
$$

with bending moments:

$$
M_x = -D\left(\frac{\partial^2 w}{\partial x^2} + \nu\,\frac{\partial^2 w}{\partial y^2}\right), \quad
M_y \ \text{analogous}
$$

The analytical non-dimensional central deflection constant for a uniformly loaded SSSS thin plate is:

$$
\frac{w_c D}{q a^4} = 0.00406235
$$

This exact value will be used as the benchmark for convergence analysis.

---

## 2) Navier Solution (Analytical Benchmark)  
*(Keywords: Navier double sine series, analytical solution for plate bending, truncation bias, SSSS plate deflection)*

The **Navier solution** is an analytical method for plates with simply-supported edges. It expands the deflection surface $w(x,y)$ as a **double sine series** in the $x$ and $y$ directions, automatically satisfying displacement boundary conditions:

$$
w(x,y) = \sum_{m=1,3,5,\dots}^M \sum_{n=1,3,5,\dots}^M W_{mn} \, \sin\left(\frac{m\pi x}{a}\right) \sin\left(\frac{n\pi y}{a}\right)
$$

The coefficients $W_{mn}$ are derived from the governing PDE and have the form:

$$
W_{mn} = \frac{16 q}{\pi^6 D\,m^2 n^2 \left[\left(\frac{m}{a}\right)^2 + \left(\frac{n}{a}\right)^2 \right]^2}
$$

Because of the sine terms, only odd $m$ and $n$ contribute. At the plate centre $(a/2,a/2)$, each sine factor equals one, making the series evaluation particularly simple.

From a convergence standpoint, the number of retained terms $M$ controls accuracy. A small $M$ produces an overestimate of the central deflection because high-frequency bending modes are neglected — this is the **truncation bias**. As $M$ increases, the solution converges rapidly to the exact constant. In this study, we consider $M=\{3,5,7,\dots,41\}$ and confirm that the largest truncation matches the analytical constant to better than $2\times 10^{-8}$.

---

## 3) FEM Formulation  
*(Keywords: Kirchhoff–Love finite element, C¹ Hermite interpolation, stiffness matrix, mesh refinement)*

The **finite element method** (FEM) implementation follows Kirchhoff–Love theory, which requires $C^1$ continuity across element boundaries — both deflection and slope must be continuous. To achieve this, we use a **4-node rectangular Hermite element** with the following degrees of freedom per node:

$$
\{ w, \ \theta_x, \ \theta_y \}
$$

The element stiffness matrix is assembled from:

$$
K_e = \iint_{\Omega_e} B^\top D_\text{plate} B \, d\Omega, \quad
D_\text{plate} =
\begin{bmatrix}
D & \nu D & 0 \\
\nu D & D & 0 \\
0 & 0 & \frac{1-\nu}{2}D
\end{bmatrix}
$$

The element load vector for uniform pressure $q$ is:

$$
f_e = \iint_{\Omega_e} N^\top q \, d\Omega
$$

**Boundary conditions** for SSSS are imposed as:
- $w=0$ along all edges (essential BC).
- $\theta_x = 0$ on $y=0,a$ and $\theta_y=0$ on $x=0,a$ (rotation constraints).
- Bending moments vanish naturally on all edges.

For the mesh refinement study, we generate uniform $n\times n$ grids with element size $h=a/n$, testing $n=\{8,10,12,16,20,24,32,40\}$. This allows us to observe how element size affects convergence toward the analytical reference.

---

## 4) Results — Figures and Detailed Discussion  
*(Keywords: FEM convergence, Navier convergence, numerical stiffness, truncation bias, deflection contour, 3D deformation surface)*

### Figure 1 — Convergence: FEM (bottom x-axis) vs. Navier (top x-axis)
<p align="center">
  <img src="./3.png" alt="Convergence: FEM vs Navier" width="720">
</p>

This plot overlays the FEM and Navier convergence trends. The **blue curve** (left axis) shows FEM central deflection versus mesh size $h$. The **orange curve** (top axis) shows Navier central deflection versus truncation parameter $M$. The **dashed line** marks the exact constant $0.00406235$.

FEM results start below the exact value for coarse meshes, reflecting **numerical stiffness** from under-resolved curvature. As the mesh refines, values increase monotonically toward the constant. Navier results start above the exact value for small $M$ due to truncation bias, then decrease toward the constant. The crossing point near the dashed line highlights that both methods converge from opposite sides — a strong validation result.

| FEM $n$ | FEM (nd)  | Navier $M$ | Navier (nd)  | Exact nd     |
|---------|----------|------------|--------------|--------------|
| 8       | 0.00388  | 3          | 0.004152     | 0.00406235   |
| 12      | 0.00396  | 5          | 0.004120     | 0.00406235   |
| 16      | 0.00402  | 7          | 0.004097     | 0.00406235   |
| 24      | 0.004054 | 21         | 0.004067     | 0.00406235   |
| 40      | 0.004061 | 41         | 0.004062     | 0.00406235   |

---

### Figure 2 — 3D Deformation Surface (FEM, Fine Mesh)
<p align="center">
  <img src="./2.png" alt="3D deformation surface (FEM fine mesh)" width="540">
</p>

The 3D surface plot reveals the **bowl-shaped** deflection pattern expected for a uniformly loaded SSSS plate. Symmetry is evident about both midlines, and the deflection smoothly reduces to zero along all edges.  
On the finest mesh ($n=40$), the normalized central deflection matches the analytical constant within 0.2% relative error. This confirms both geometric fidelity and adequate numerical integration — here, $2\times 2$ Gauss points suffice for constant loading.

---

### Figure 3 — Filled Contour of Deflection (FEM, Fine Mesh)
<p align="center">
  <img src="./1.png" alt="Deflection contour (FEM fine mesh)" width="640">
</p>

The contour plot complements the 3D view by making **symmetry and gradient patterns** more apparent. Concentric contours are centred at $(a/2,a/2)$, gradually transitioning toward the edges.  
Near the boundaries, contours become slightly square — a typical feature for simply-supported plates under uniform load. The zero contour along all edges confirms correct enforcement of $w=0$.

---

## 5) Reproduce / Run  
*(Keywords: Python Navier solver, Python FEM solver, convergence benchmark)*

- **Navier (Python)**: `Navier_code/navier_solver.py` computes the normalized central deflection for a chosen odd truncation $M$.  
- **FEM (Python)**: `FEM_code/fem_solver.py` demonstrates stiffness and load assembly for $C^1$ Hermite rectangular elements.

Both codes are structured for reproducibility and easy extension to more complex loading or boundary conditions.

---

## 6) Takeaways  
*(Keywords: FEM vs analytical solution, numerical convergence validation, thin plate benchmark)*

- Both FEM and Navier methods converge to the classical constant $0.00406235$ for SSSS thin plates.  
- FEM underestimates for coarse meshes due to numerical stiffness; Navier overestimates for small $M$ due to truncation.  
- The convergence-from-opposite-sides behavior is a robust **sanity check** before moving to problems with added complexity.  
- This benchmark can be extended to thick plates, anisotropic materials, or plates with non-uniform loading to validate more advanced solvers.
  Mohammadjafar Sharifi (Mj Sharifi)

---
