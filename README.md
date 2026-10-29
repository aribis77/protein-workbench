# The Protein Workbench

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23021531.svg)](https://doi.org/10.5281/zenodo.23021531)

A structural analysis workbench that runs entirely in a browser. One self-contained
HTML file: no installation, no server, no account, and coordinates are processed locally unless the user explicitly uses a remote retrieval feature.

**Live: https://aribis77.github.io/protein-workbench/**

---


## URL-based integration with external knowledge graphs (experimental)

Protein Workbench remains a **general-purpose, independent browser application**. An external knowledge graph or analysis portal can open it with a structure accession and a protein substitution:

```text
https://aribis77.github.io/protein-workbench/?pdb=1ABC&mutation=H1786D&chain=A&analysis=combined
```

The PDB ID above is **illustrative, not a validated structure for H1786D**. Replace it with a suitable experimentally supported structure and verify that the mutation's residue number and wild-type identity match the structure. The optional `chain` parameter disambiguates chains. `analysis` accepts `interface_contacts`, `relative_binding_energy`, `conformational_sampling`, `conservation`, or `combined`. The optional `alignment_url` parameter accepts an HTTPS URL to an accessible FASTA/CLUSTAL **multiple-sequence alignment**. Remote loading depends on the host's cross-origin policy; automatic RCSB sequence-homologue discovery is available as an alternative. The automatic query-anchored alignment is not a full phylogenetic MSA.

After a successful PDB fetch and unambiguous residue mapping, the current integration can run contact mapping and mutant construction, empirical energy scoring for the relevant modes, and a **quick exploratory ensemble** for sampling/combined mode. Conservation now attempts automatic discovery of structurally characterized homologues through the RCSB PDB sequence-search service and constructs a query-anchored alignment. This is a preliminary PDB-derived conservation estimate, **not** a curated phylogenetic multiple-sequence alignment; if too few distinct homologues are found, conservation is explicitly unavailable. Users may instead provide a curated alignment. Combined mode downloads a machine-readable **analysis-status report**, not a validated, evidence-weighted causal mechanism or an automatically curated KG record. Results are not currently synchronized back into the originating KG.

**Scientific limits:** A PDB residue number is not necessarily a full-length protein sequence number; ambiguous mappings stop the automatic workflow. Empirical scores are not rigorous binding free energies. The quick ensemble is not production molecular dynamics. Alignment composition, redundancy, coverage and mapping determine conservation reliability. Missing components must be reported as not evaluated. Always review output and provenance before using it in a manuscript.

**Privacy and networking:** Local coordinate analysis remains browser-side, but using a PDB accession or an `alignment_url` makes the browser request public remote resources. URLs may be visible to the hosting and resource servers. Do not include confidential sequences or private identifiers in query parameters.

**Current status:** GitHub code committed; this URL-driven integration has not yet passed an independently documented end-to-end browser test. It is an experimental interface, not a stable public API.

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
within one run. Binding free energy uses the isolated chain as its reference, with an
interface score that includes desolvation of the buried area, and a hotspot scan ranks the
interface residues worth mutating on the bench, with each one's whole-structure cost and the
reasons a residue should not be touched (disulfides, metal sites, crystal contacts, glycans).

**Cryo-EM** — density support per residue and Q-scores per atom; density the model does not
explain, and model with no density behind it, at a resolution found from the map; rigid fitting
of a model into a map, or domain by domain for a predicted model using its predicted aligned
error; and export of the map region around a model.

**Atomic force microscopy** — preparation of measured topographs (spikes, tilt, scan-line
offsets), fitting the orientation of a structure or a cryo-EM map to an image, an upper bound
on the tip radius from the image itself, flexible fitting along the softest modes of an
anisotropic network, and ranking many candidate structures against many images, also from the
command line, resumable and in parallel.

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
| AFM tip bound, features sharper than the tip | the true radius | 10.00, 20.00, 30.00 Å for 10, 20, 30 Å tips |
| Anisotropic network modes | full eigendecomposition | eigenvalues within 3 × 10⁻¹¹ |
| Contour of a Gaussian sphere | 4πr² | 156.2 against 156.8 Å² (−0.4%) |
| Rigid fit, model turned 15° and moved 4.7 Å | its true place | 0.12 Å from it after fitting |
| Q-score, atom in density of the reference shape | 1 | 0.978 mean over 556 atoms |
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

Map fitting is a local refinement from where the model starts, not a global search. The
unexplained-density analysis compares the map with a density simulated from the model,
including its B-factors, so a model whose B-factors do not describe the map can leave
differences that are not missing atoms. AFM fitting models the tip's shape but not its force:
real molecules are compressed and flattened on the surface, which lowers measured heights by an
amount no setting here supplies, and the flexible fit can only follow motions the network's
softest modes contain.

The crosslink competence screen is a geometric filter: it establishes that a pair is not
excluded by distance and burial, and says nothing about whether an enzyme would accept the
site. The mechanical analysis is a ranking of susceptibility, not a force in piconewtons.
The structural dendrogram is a clustering by shape and not a phylogeny. Each of these is
restated in the tool at the point where the result appears.

## Citing

A manuscript is in preparation. Until it appears, please cite the archived software
release: Biswas, A. *The Protein Workbench* (v1.1.0). Zenodo. https://doi.org/10.5281/zenodo.23021531
GitHub also renders `CITATION.cff` as a "Cite this repository" button.

## Licence

MIT. The bundled rendering library, 3Dmol.js, is BSD-3-Clause and is redistributed under
its own terms.

### Quantitative combined export

Combined analysis now exports a `metrics` object with detected contact-change counts, residue conservation/coverage where available, empirical energy-score objects and exploratory sampling results, alongside completion status. These are method-specific measurements, **not** validated binding free energies or pathogenicity probabilities. Review numerical output and provenance before publication. This change has not yet undergone end-to-end live-browser validation.

### Automatic KG result return (experimental)

When Workbench is launched from the KG with a one-time random launch token and an approved return origin, its combined JSON report is sent back to the originating KG browser tab via `postMessage`. The KG validates the message origin, token, PDB and mutation before adding the result to its session-local variant report. Keep the KG tab open. This is a browser-to-browser handoff, not a server-side job queue or durable storage; it has not yet been validated in a live browser. Standalone Workbench remains independent and still exports JSON. Manual import/reset in the KG is under Advanced.
