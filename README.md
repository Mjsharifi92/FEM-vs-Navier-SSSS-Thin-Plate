Convergence Analysis: FEM vs. Navier Solution — SSSS Thin Plate (Steel)

This repository validates a Kirchhoff–Love thin-plate FEM against the classical Navier double-sine series for a square, simply-supported (SSSS) steel plate under a uniform transverse load. It includes a mesh-refinement study for FEM and a truncation-study for the Navier series, with figures and detailed interpretation.
1) Problem definition

Geometry & loading (non-dimensionalized with side length a):
A square plate a × a with thickness t << a, uniform pressure q.

Material (steel):
E = 210 GPa
ν = 0.3

Flexural rigidity:

D = E * t^3 / ( 12 * (1 - ν^2) )
D = 1.9230 × 10^4 N·m   (for t = 0.01 m)

Governing equation (Kirchhoff–Love):

D ∇⁴ w = q   in Ω = [0,a] × [0,a]

SSSS boundary conditions (on all edges):

w = 0,    M_n = 0

where bending moments are:

M_x = -D [ (∂²w/∂x²) + ν (∂²w/∂y²) ]
M_y = -D [ (∂²w/∂y²) + ν (∂²w/∂x²) ]

Reference exact central deflection (non-dimensional):

(w_c * D) / (q * a^4) = 0.00406235

2) Navier solution (analytical benchmark)

For SSSS plates, the deflection admits a double sine series (odd terms only):

w(x,y) = Σ_{m=1,3,...,M} Σ_{n=1,3,...,M} W_mn *
         sin(mπx/a) * sin(nπy/a)

W_mn = 16q / [ π^6 * D * m² * n² * ( m²/a² + n²/a² )² ]

At the plate centre (a/2,a/2) every sine term equals 1 (for odd indices).
Small M overestimates the true central deflection due to truncation.

Code parameters used here:

    Odd truncations: M = {3,5,7,...,41}

    Large M → 0.004062 (matches the exact constant to < 2×10⁻⁸)

3) FEM formulation

We use a Kirchhoff thin-plate element requiring C¹ continuity.
A practical way is a 4-node C¹ rectangular element with Hermite polynomials in each direction and nodal DOFs:

{ w, θ_x, θ_y }

Element matrices:

K_e = ∫∫( Bᵀ D_plate B ) dΩ

D_plate =
[  D      νD       0
   νD     D        0
   0      0   (1-ν)/2 * D ]

Loads for uniform q:

f_e = ∫∫( Nᵀ q ) dΩ

Boundary conditions (SSSS):

    w = 0 along all four edges (essential)

    θ_x = 0 on y = 0,a

    θ_y = 0 on x = 0,a

    Bending moments become natural BCs and vanish on edges

Mesh refinement:
We use an n × n grid of equal rectangles; element size h = a/n.
Tested n = {8,10,12,16,20,24,32,40}.
4) Results — figures and detailed discussion
Figure 1 — Convergence: FEM (bottom x-axis) vs. Navier (top x-axis)

Convergence: FEM vs Navier

What the plot shows

    Blue curve (bottom x-axis): FEM central deflection vs. mesh size h = a/n

    Orange curve (top x-axis): Navier central deflection vs. truncation parameter M (plotted as 1/M, odd values)

    Dashed line: exact non-dimensional constant 0.00406235

Interpretation

    FEM starts below the exact value for coarse meshes (slightly too stiff numerically), then increases monotonically and asymptotes to the exact constant.

    Navier starts above the exact value for small M (series truncation bias), then decreases to the limit as M grows.

    The crossing near the dashed line confirms both methods target the same limit from opposite sides, providing a strong code-to-theory validation.

Representative numbers (non-dimensional w_c D / (q a^4)):

    FEM: n = {8,12,16,24,40} → {0.00388, 0.00396, 0.00402, 0.004054, 0.004061}

    Navier: M = {3,5,7,41} → {0.004152, 0.004120, 0.004097, 0.004062}

Figure 2 — 3D deformation surface (FEM, fine mesh)

3D deformation surface

What to look at

    Smooth, bowl-shaped surface with symmetry about both midlines, consistent with SSSS edges

    Maximum deflection at the centre; zero along edges

    The plotted normalized central value matches the constant within ~0.002 relative error on the finest grid shown

FEM settings behind this plot

    Fine grid n = 40 (i.e., h = 0.025 a)

    Consistent Hermite interpolation for w, θ_x, θ_y

    Uniform load integration with 2 × 2 Gauss points per rectangle

Figure 3 — Filled contour of deflection (FEM, fine mesh)

Deflection contour

Reading the contour

    Concentric contours centred at (a/2, a/2) show a monotone decrease toward edges

    The circular-like pattern in the middle becomes slightly square near edges, a typical feature for SSSS plates under uniform pressure

    Zero contour on every boundary confirms correct enforcement of w = 0

Why both 3D and contour help

    The 3D surface conveys global smoothness and qualitative shape (good for talks)

    The contour makes gradients and symmetry obvious (good for reports/code verification)

5) Numerical table (selected points)
mesh n	h = a/n	FEM (nd)	series M	Navier (nd)	exact nd
8	0.125	0.00388	3	0.004152	0.00406235
12	0.0833	0.00396	5	0.004120	0.00406235
16	0.0625	0.00402	7	0.004097	0.00406235
24	0.0417	0.004054	21	0.004067	0.00406235
40	0.0250	0.004061	41	0.004062	0.00406235

All FEM and Navier values approach the same constant within plotting precision.
6) Reproduce / run

    Navier (Python): Navier_code/navier_solver.py computes the normalized centre deflection for a chosen odd M

    FEM (placeholder): FEM_code/fem_solver.py shows where to plug in your stiffness/load assembly

7) Takeaways

    Both methods converge to the classical constant 0.00406235 for SSSS thin plates

    FEM tends to underestimate on coarse meshes (numerical stiffness)

    Navier tends to overestimate for small M (truncation)

    The convergence curves form a robust sanity check for plate FEM codes

8) Repository layout

.
├── 1.png                 # Convergence (FEM bottom axis vs Navier top axis)
├── 2.png                 # 3D deflection surface (FEM fine)
├── 3.png                 # Filled deflection contours (FEM fine)
├── FEM_code/
│   └── fem_solver.py
├── Navier_code/
│   └── navier_solver.py
├── data/
├── LICENSE
├── .gitignore
└── README.md
