🇪🇸 [Leer en español](README.es.md)

# AXISYMMETRIC SOLAR DYNAMO MODEL

======================================

## PURPOSE

---

This program implements a large-scale axisymmetric dynamo model in spherical coordinates. The model evolves the poloidal and toroidal magnetic fields, includes field generation due to differential rotation and the alpha effect, and allows the magnetic force to modify the meridional flow. The aim is to study the time evolution of the magnetic field and the feedback between the field and the motions of the stellar fluid.

The scientific scope of the model corresponds to the work of Sraibman and Minotti (2019). The calculation sequence and the names of the terms used below refer specifically to the implementation in this folder and do not replace the full derivation presented in the paper.

## ORGANIZATION OF THE CALCULATION

---

1. `toy_prueba.py` creates the radial and angular grids, initializes the poloidal potential, the rotation profile and the background meridional flow, and coordinates the time integration. This is the main file of the program. It can be run directly with the parameters defined in the code and allows studying the case corresponding to solar parameters.

2. `alfan4.py` computes the kinematic alpha coefficient from the gradients of the differential rotation. `ws1alfaBL1.py` evolves an additional alpha contribution associated with the feedback.

3. `etauv4.py` estimates the local turbulent diffusivity using the Smagorinsky coefficient, the flow strain, the rotation and the magnetic field.

4. `calca_1.py` integrates the equation for the poloidal vector potential `A`. The components `Br` and `Btheta` are obtained from `A`. The equation includes transport terms, diffusion, subgrid contributions and the alpha source `B`.

5. `calcb4_1.py` integrates the equation for the toroidal field `Bphi`. It includes transport terms, diffusion and shear due to differential rotation, in addition to the subgrid closure defined in `Scero3.py`.

6. `leap2.py` computes the perturbation of the meridional flow induced by the magnetic force and adds it to the background meridional flow defined by `velmeri4.py`.

7. `toy_prueba.py` advances the fields in time, applies smoothing every given number of steps and checks for the appearance of possible numerical instabilities.

The numerical kernels use Numba when it is installed. `_jit.py` provides an interpreted fallback to run these kernels when Numba is not available.

## MAIN PARAMETERS

---

The main parameters used by the simulation are the following:

* `factalf`: relative amplitude of the alpha effect. Value used: 0.35.

* `cs`: Smagorinsky coefficient used to compute the turbulent diffusivity. Value used: 0.1.

* `suavizado`: interval between applications of smoothing to `A` and `B`. Value used: 200 steps.

* `n`: multiplier of the rotation `omega`. Value used: 1.

* `dibu`: interval between diagnostic plots. Value used: 50000 steps.

* `timesave`: sampling interval of the histories. Value used: 1000 steps.

* `maxpasos`: maximum number of simulation steps. Value used: 500000 steps.

* `delttime`: initial time step. It is recalculated at the start of the simulation according to the advective and diffusive limits.

* `rtope`: outer radius of the grid. Value used: `7e8 m`.

* `rcero`: inner radius of the domain. Value used: `0.55*rtope`.

* `rp`: radius up to which meridional flow penetration is allowed. Value used: `0.69*rtope`.

* `lamda`: characteristic length associated with the subgrid average. Value used: `0.1*rcero`.

* `puntostit`: number of angular intervals. The size of the angular interval is defined as `deltit = pi/puntostit`.

* `puntosr`: number of radial intervals. Value used: 63.

* `mu0`: magnetic permeability of vacuum.

* `sigma`: magnitude and sign of the imposed meridional flow. In `toy_prueba.py`, its value is `-7e-8`.

## AUXILIARY FILES

---

* `calca_1.py` and `calcanx.py`: compute the evolution of the poloidal field and handle the terms associated with the outer boundary.

* `calcb4_1.py`: computes the evolution of the toroidal field and generates plots of the different terms of its equation.

* `Scero3.py`: computes the subgrid term of the toroidal field equation.

* `etauv4.py`: estimates the local turbulent diffusivity.

* `omegaf.py` and `densidad.py`: compute the differential rotation and density profiles, respectively.

* `velmeri4.py`: computes the background meridional flow.

* `leap2.py`: computes the perturbation of the meridional flow induced by the magnetic field.

* `ws1alfaBL1.py`: evolves the feedback vorticity and the alpha term associated with the BL mechanism.

* `suavixy.py`, `suavixy2d.py` and `suavixy2dbl.py`: apply filters and the equatorial symmetry conditions used in the model.

* `guardbb.py`, `guardaur.py` and `guardautit.py`: perform time sampling at selected radii.

* `grafurut.py`: generates plots of the field and flow components in the meridional plane.

## OUTPUTS

---

The output folder is created in the same directory as `toy_prueba.py`. Its name is built from the values of `factalf`, `cs` and `n`.

The histories are saved as compressed NumPy `.npz` files:

* `brtaco.npz`: contains `tiempototal` and five histories of `Br`.

* `btittaco.npz`: contains five histories of `Btheta`.

* `b.npz`: contains five histories of the toroidal field.

* `ur.npz`: contains five histories of the perturbation `ur`.

* `utit.npz`: contains five histories of the perturbation `utheta`.

* `todo.npz`: contains the state and the serializable parameters of the simulation.

The series generated by `guardaur`, `guardautit` and `guardbb` correspond to five specific radial indices: 21, 22, 29, 39 and 49. Therefore, these histories do not represent a full radial grid.

The data of each history are recorded every `timesave` steps.

The `.npz` files can be opened, for example, with:

```python
with numpy.load("ur.npz") as datos:
    ur_en_radio_3 = datos["urt3"]
```

## REQUIREMENTS

---

Python is required, along with the **NumPy**, **SciPy** and **Matplotlib** libraries.

**Numba** is optional. When installed, it is used to speed up the numerical kernels. If Numba is not available, `_jit.py` provides an interpreted fallback.

## REFERENCE

---

Sraibman, L., & Minotti, F. (2019). *Large-scale model of the axisymmetric dynamo with feedback effects*. Solar Physics, 294(1), 14.

DOI: https://doi.org/10.1007/s11207-018-1350-1
