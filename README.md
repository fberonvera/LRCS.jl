# LRCS.jl

Julia implementation for detecting Lagrangian rotating contracting structures
(LRCS) in unsteady two-dimensional velocity fields.


LRCS are material regions combining:

- elevated accumulated intrinsic rotation, quantified by the
  Lagrangian-averaged vorticity deviation (LAVD);
- finite-time material contraction.

The method is designed to identify rotating contracting regions without
requiring convexity or shape preservation of their material boundaries.

This repository implements the detection procedure described in:

F. J. Beron-Vera (2026),
"Lagrangian rotating contracting structures,"
*Chaos*, submitted.

Preprint: https://arxiv.org/abs/2604.25036

---

## Author

Francisco J. Beron-Vera

---

## Overview

LAVD measures accumulated intrinsic rotation along trajectories, but elevated
LAVD or closed LAVD contours alone do not imply a coherent vortex.

LRCS detection therefore combines LAVD with a direct material-contraction
criterion.

Typical workflow:

velocity field
    ->
trajectory integration
    ->
LAVD computation
    ->
candidate boundary extraction
    ->
material advection
    ->
contraction test
    ->
boundary selection
    ->
LRCS identification

No new diagnostic is introduced: LAVD provides rotational information, while
material contraction supplies the additional dynamical criterion.

---

## Key idea

LAVD proposes candidate rotating regions; material contraction determines
their dynamical admissibility.

For a material region U(t), a candidate must satisfy

A(t1) / A(t0) < 1,

where A(t) is the area enclosed by its material boundary.

Among contracting candidates, the boundary is selected by maximizing the
mean LAVD excess above its boundary level,

E(Gamma) = (1 / |U|) integral_U (LAVD - c)_+ dA,

where c is the LAVD value on the candidate boundary Gamma.

This favors regions whose interiors contain elevated accumulated intrinsic
rotation relative to their boundary without imposing convexity or
shape-preservation requirements.

---

## Files

lrcs.jl        Core implementation
lrcs_run.jl    Driver script
data/irma.nc   Example Hurricane Irma velocity data
README.md      Documentation

---

## Requirements

- Julia (>= 1.8 recommended)
- NetCDF support for velocity input
- Standard numerical libraries

---

## Quick start

Run:

    julia lrcs_run.jl

This uses the included Hurricane Irma example:

    data/irma.nc

---

## Usage

Typical workflow:

1. Define specifications in the driver:
   - DataSpec
   - DomainSpec
   - NumericsSpec
   - OutputSpec

2. Execute:

       include("lrcs_run.jl")

3. Outputs are written to the paths defined in OutputSpec.

---

## Input data

The code expects:

- velocity components on a grid;
- time-resolved velocity data;
- consistent spatial coordinates.

Typical units:

- velocity: m/s
- time: seconds
- coordinates: degrees (internally converted to meters)

---

## Output

The code produces:

- LAVD field at the initial time;
- selected LRCS boundary at t0;
- advected material boundary at t1;
- optional intermediate boundary positions;
- diagnostic quantities including:
  - A(t0)
  - A(t1)
  - contraction ratio A(t1)/A(t0)

Outputs may include NetCDF files and figures depending on configuration.

---

## Internal workflow

### Candidate generation

LAVD is computed on the initial-condition grid. Closed LAVD level sets provide
candidate material boundaries. Candidate generation does not impose a
convexity requirement.

### Material contraction

Each candidate boundary is advected over the prescribed time interval. The
area enclosed by the material curve is evaluated at the initial and final
times.

Only candidates satisfying

A(t1) < A(t0)

are retained.

### Boundary selection

Among contracting candidates, the code favors the boundary maximizing the
area-normalized LAVD excess above its boundary level,

E(Gamma) = (1 / |U|) integral_U (LAVD - c)_+ dA.

Here c is the LAVD level defining Gamma and U is its enclosed region.

If needed, a mild inward buffer can be applied to reduce filamentation of an
outer material boundary.

### Material advection

Material curves are advected using a fourth-order Runge-Kutta scheme and are
periodically redistributed to maintain adequate numerical resolution during
deformation.

---

## Main components

Specifications:

- DataSpec: input file and variable names
- DomainSpec: spatial domain and time interval
- NumericsSpec: numerical parameters
- OutputSpec: output control

Core functions:

Velocity and trajectories:
- build_velocity_interpolant
- rk4_step
- advect_particles

LAVD:
- compute_lavd
- compute_vorticity

Boundary extraction and selection:
- find_lavd_peaks
- extract_contours
- select_lrcs_curve

Material advection:
- advect_closed_curve
- redistribute_curve

Diagnostics:
- polygon_area

---

## Notes

- LAVD measures accumulated intrinsic rotation, not material coherence.
- Closed LAVD contours do not by themselves identify coherent vortices.
- Not all rotationally enriched regions undergo material contraction.
- LRCS boundaries may deform substantially during the observation interval.
- Convexity and shape preservation are not required.
- Instantaneous streamline geometry is not used as a coherence criterion.
- The LRCS construction is material and objective.

---

## Code development

The Julia implementation was adapted from MATLAB codes developed by
F. J. Beron-Vera. ChatGPT was used to assist with the translation,
debugging, and documentation of the Julia code. The numerical methodology,
detection strategy, and scientific design are those of the author.

---

## Status

This repository accompanies ongoing research and is currently under development. Numerical parameters and interfaces may continue to evolve as the implementation is tested across additional velocity fields.

