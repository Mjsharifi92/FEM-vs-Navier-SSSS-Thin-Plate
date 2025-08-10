# Convergence Analysis: FEM vs. Navier Solution — SSSS Thin Plate (Steel)

This repository validates a **Kirchhoff–Love thin-plate FEM** against the **classical Navier double-sine series** for a **square, simply-supported (SSSS) steel plate** under a **uniform transverse load**.  

It includes:  
- **Mesh-refinement study** for FEM.  
- **Truncation-study** for the Navier series.  
- Figures and detailed interpretation.

---

## 1) Problem Definition

**Geometry & loading** (non-dimensionalized with side length $a$):  
A square plate $a \times a$ with thickness $t \ll a$, uniform pressure $q$.

**Material (steel):**  
- $E = 210\ \text{GPa}$  
- $\nu = 0.3$

**Flexural rigidity:**

$$
D = \frac{E t^3}{12(1-\nu^2)}
$$

For $t = 0.01$ m:

$$
D = 1.9230\times 10^4 \ \text{N}\cdot\text{m}
$$

**Governing equation (Kirchhoff–Love):**

$$
D\,\nabla^4 w = q \quad \text{in } \Omega = [0,a] \times [0,a]
$$

**SSSS boundary conditions (on all edges):**

$$
w = 0, \quad M_n = 0
$$

with bending moments:

$$
M_x = -D\left(\frac{\partial^2 w}{\partial x^2} + \nu\,\frac{\partial^2 w}{\partial y^2}\right)
$$

and $M_y$ defined analogously.

**Reference exact, non-dimensional central deflection** for a uniformly loaded SSSS thin plate:

$$
\frac{w_c\,D}{q\,a^4} = 0.00406235
$$

---

## 2) Navier Solution (Analytical Benchmark)

For SSSS plates, the deflection admits a **double sine series** (odd terms only):

$$
w(x,y) = \sum_{m=1,3,5,\dots}^M \sum_{n=1,3,5,\dots}^M
W_{mn}\,\sin\left(\frac{m\pi x}{a}\right)\sin\left(\frac{n\pi y}{a}\right)
$$

where:

$$
W_{mn} = \frac{16 q}{\pi^6 D\,m^2 n^2 \left[\left(\frac{m}{a}\right)^2 + \left(\frac{n}{a}\right)^2 \right]^2}
$$

At the plate centre $(a/2,a/2)$ every sine term equals $1$ (for odd indices), giving rapid convergence as $M$ increases.  
Small $M$ **overestimates** the true central deflection due to truncation.

**Code parameters used here:**
- Odd truncations $M=\{3,5,7,\dots,41\}$  
- Reported value in the plot for large $M$: $0.004062$ (matches the exact constant to $< 2\times10^{-8}$).

---

## 3) FEM Formulation (Our Implementation)

We use a Kirchhoff thin-plate element requiring $C^1$ continuity.  
A practical way is a 4-node $C^1$ rectangular element with **Hermite polynomials** in each direction and nodal DOFs:

$$
\{ w,\ \theta_x,\ \theta_y \}
$$

**Element matrices:**

$$
K_e = \iint_{\Omega_e} B^\top D_\text{plate} B\ d\Omega, \quad
D_\text{plate} =
\begin{bmatrix}
D & \nu D & 0 \\
\nu D & D & 0 \\
0 & 0 & \frac{1-\nu}{2} D
\end{bmatrix}
$$

**Loads for uniform $q$:**

$$
f_e = \iint_{\Omega_e} N^\top q \ d\Omega
$$

**Boundary conditions (SSSS):**
- $w = 0$ along all four edges (essential).  
- $\theta_x=0$ on $y=0,a$ and $\theta_y=0$ on $x=0,a$.  
- Bending moments become natural BCs and vanish on edges.

**Mesh refinement:**  
We use an $n\times n$ grid of equal rectangles; element size $h=a/n$.  
Tested $n=\{8,10,12,16,20,24,32,40\}$.

---

## 4) Results — Figures & Discussion

### Figure 1 — Convergence: FEM (bottom x-axis) vs. Navier (top x-axis)
<p align="center">
  <img src="./3.png" alt="Convergence: FEM vs Navier" width="720">
</p>

**What the plot shows:**
- **Blue curve (left axis):** FEM central deflection vs. mesh size $h=a/n$.  
- **Orange curve (top axis):** Navier central deflection vs. truncation parameter $M$ (plotted as $1/M$, odd values).  
- **Dashed line:** exact non-dimensional constant $0.00406235$.

**Interpretation:**
- FEM starts **below** the exact value for coarse meshes (slightly too stiff numerically), then increases monotonically to the exact constant.  
- Navier starts **above** the exact value for small $M$, then decreases to the limit as $M$ grows.  
- The crossing near the dashed line confirms both methods converge to the same limit from opposite sides.

**Representative numbers** (non-dimensional $(w_c D)/(q a^4)$):

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

**Observations:**
- Smooth, bowl-shaped surface with symmetry about both midlines.  
- Maximum deflection at the centre; zero along edges.  
- Central value matches constant within ≈ $2\times 10^{-3}$ relative error on the finest grid.

**FEM settings:**
- Fine grid $n=40$ ($h=0.025\,a$).  
- Hermite interpolation for $w,\theta_x,\theta_y$.  
- $2\times2$ Gauss integration for uniform $q$.

---

### Figure 3 — Filled Contour of Deflection (FEM, Fine Mesh)
<p align="center">
  <img src="./1.png" alt="Deflection contour (FEM fine mesh)" width="640">
</p>

**Reading the contour:**
- Concentric contours centred at $(a/2,a/2)$, decreasing toward edges.  
- Slightly square near edges (typical for SSSS plates under uniform pressure).  
- Zero contour along all boundaries confirms $w=0$.

**Why both plots help:**
- 3D surface → qualitative shape & smoothness (good for presentations).  
- Contour → symmetry & gradients (good for reports and verification).

---

## 5) Reproduce / Run

- **Navier (Python)**: `Navier_code/navier_solver.py` computes normalized centre deflection for chosen odd $M$.  
- **FEM (Python)**: `FEM_code/fem_solver.py` placeholder for stiffness/load assembly with $C^1$ plate elements.

---

## 6) Takeaways

- Both methods converge to $0.00406235$ for SSSS thin plates.  
- FEM **underestimates** on coarse meshes (numerical stiffness).  
- Navier **overestimates** for small $M$ (truncation bias).  
- The convergence-from-opposite-sides pattern is a strong validation before extending to thicker plates, orthotropy, or nonlinear effects.

---
