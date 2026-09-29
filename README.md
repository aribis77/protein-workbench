# The Protein Workbench

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23021531.svg)](https://doi.org/10.5281/zenodo.23021531)

A structural analysis workbench that runs entirely in a browser. One self-contained
HTML file: no installation, no server, no account, and no coordinates ever leave the
machine it is opened on.

**Live: https://aribis77.github.io/protein-workbench/**

---

## What it is for

Software that enumerates contacts in a protein structure is mature, and so is software
that predicts the effect of a mutation. What neither class provides is any indication of
how much confidence a given contact deserves. A contact list derived from a deposited
model treats a hydrogen bond supported by unambiguous density identically to one built
into noise, a contact that persists through every member of an ensemble identically to
one seen in a single frame, and a contact within a correctly assembled biological unit
identically to one between molecules that never meet.

This tool attaches that evidence to the contact and then carries it through to the
conclusions. A residue's centrality in the interaction network is recomputed with every
edge weighted by how well its contacts are supported, so a hub that exists only through
contacts nobody can see is identified as one. A free energy reported for a site carries a
statement about whether the geometry it was computed on is actually visible in the data.

## Getting started

Open the link above and press **Load example**, then **Map contacts**. Everything else
follows from there. To use your own structure, drag a PDB or mmCIF file onto the window,
or fetch one by accession.

To run it offline or on your own machine, download `index.html` and open it in any
browser. It works from a local file with no network connection, apart from the fetch
buttons, which need one.

## What it does

**Interactions** — hydrogen bonds, salt bridges, hydrophobic contacts, π–π stacking,
cation–π, metal coordination, disulfides, isopeptide bonds, steric clashes, and
nucleic-acid chemistry including Watson–Crick pairing and base stacking. Biological
assemblies are built from the deposition's own records before analysis, because the
deposited coordinates are frequently not the biological unit.

**Evidence** — real-space correlation against a cryo-EM or crystallographic map, occupancy
across the models of an ensemble or the frames of a trajectory, and for predicted
structures both pLDDT and the pairwise predicted aligned error, which identifies contacts
between confidently placed residues whose relative position the prediction does not in
fact constrain.

**Mutation and energetics** — in-silico substitution with rotamer search and local
relaxation, an empirical score for triage, conformational sampling in torsion space, and
alchemical free energy by thermodynamic integration with Bennett's acceptance ratio on the
same samples. Uncertainty comes from independent repeats rather than from block averaging
within one run.

**Geometry** — solvent-accessible surface distances by Dijkstra search, which is the right
measurement for crosslink restraints and reach; cavities by flood fill and pockets by
protein–solvent–protein events; and a screen for geometric competence to form
Nε-(γ-glutamyl)lysine crosslinks.

**Networks** — betweenness, articulation points and communication paths on a graph weighted
by interaction energy, compared against degree-preserving randomisations so that a residue
is reported as *surprisingly* central rather than merely central.

**And also** — crystal-contact identification by space-group expansion, elastic network
modes, conservation from an alignment, water-mediated bridges, Poisson–Boltzmann
electrostatics with pKa shifts, glycan occlusion, mechanical load distribution, structural
alignment with a similarity dendrogram, AFM topograph simulation with orientation fitting,
batch analysis across many structures, and a session manifest that records a hash of every
input and result so a later run can be verified against it.

Results export as PDF, CSV, JSON, SVG and PDB.

## Validation

The numerical core is checked against cases with independently known answers rather than
against its own output. Representative examples:

| Quantity | Reference | Obtained |
|---|---|---|
| Elastic network eigenvalues, bead chain | path-graph Laplacian, analytic | agreement to 8 decimal places |
| Cavity volume, hollow shell | 2953 Å³ analytic interior | 2970 Å³ (0.6%) |
| Surface path, unobstructed | Euclidean distance | exact at all grid spacings |
| Rotamer populations | exact Boltzmann enumeration | total variation distance 0.014 |
| Bennett estimator, Crooks-consistent work | 1.200 kcal/mol | 1.203 kcal/mol |
| Poisson solver, charge in a sphere | q/(εr), analytic | within 1.1% |
| Born radius, isolated atom | van der Waals radius, analytic | 1.696 Å against 1.700 Å |
| AFM tip broadening | tip-centre locus, analytic | 80.6 Å against 80.6 Å |
| Neighbour joining, additive matrix | the tree it was built from | recovered exactly |
| Disulfide design, 1A8O Cys198–Cys218 | deposited: 2.04 Å, 94.1° | modelled: 1.96 Å, 99.9° |

Detection is additionally checked against annotations deposited in the coordinate files
but not used by the software — SSBOND records, sequence-derived bond counts, and
Watson–Crick pairing.

## Limitations

Stated plainly, because they bound what the results mean.

The backbone does not move during mutation, so substitutions requiring backbone
accommodation are underestimated. Solvent is implicit throughout. Protonation states are
fixed and histidine is treated as neutral, though the electrostatics module will tell you
which groups that assumption is wrong for. The empirical energy function is not
parameterised against a mutation thermodynamics database, so the sign and the ranking
within a series are the defensible outputs rather than the absolute value. The elastic
network is the isotropic form. Compressed XTC trajectories are not read; convert with
`gmx trjconv` first.

The crosslink competence screen is a geometric filter: it establishes that a pair is not
excluded by distance and burial, and says nothing about whether an enzyme would accept the
site. The mechanical analysis is a ranking of susceptibility, not a force in piconewtons.
The structural dendrogram is a clustering by shape and not a phylogeny. Each of these is
restated in the tool at the point where the result appears.

## Citing

A manuscript is in preparation. Until it appears, please cite the archived software
release: Biswas, A. *The Protein Workbench* (v1.0.1). Zenodo. https://doi.org/10.5281/zenodo.23021531
GitHub also renders `CITATION.cff` as a "Cite this repository" button.

## Licence

MIT. The bundled rendering library, 3Dmol.js, is BSD-3-Clause and is redistributed under
its own terms.
