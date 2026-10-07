# The Protein Workbench — user manual

Version 1.0 · September 2026

This manual describes everything the tool does, in the order the interface presents it.
Each section says what the analysis is, what it needs, and what the result does and does
not license. Nothing here requires you to read it in order; the tool is usable from the
first two buttons and the rest can be found when you need it.

---

## 1 · Getting a structure in

The control column runs down the left. Panel 1 takes a structure in any of six ways: a
file picker, drag and drop anywhere on the window, a four-character accession fetched from
the Protein Data Bank, a UniProt accession fetched from the AlphaFold database, coordinates
pasted as text, and a bundled example that needs no network at all. PDB and mmCIF are both
read, including author chain and sequence identifiers, alternate locations, and multiple
models.

Three switches govern what is kept. Waters are discarded by default and must be retained
if you intend to look at water-mediated bridges. Hydrogens are removed by default, because
every geometric criterion downstream is parameterised for heavy atoms; leaving them in
produces large numbers of spurious clashes, since an amide hydrogen sits well inside the
van der Waals sum of its acceptor. Heteroatoms are kept by default, which matters for
metals, ligands and modified residues.

**The structural unit.** If the deposition carries assembly records, a selector appears
offering the biological assemblies alongside the deposited coordinates. This is not a
cosmetic choice. The deposited coordinates of the HIV-1 capsid domain 1A8O comprise one
chain with no inter-chain contacts whatsoever, while its biological dimer has seventy-one
across twenty-one residue pairs. An interface analysis run on the deposited file would
return an empty result that is easily mistaken for a negative finding. Switching the unit
clears everything derived from the previous one, because carrying a contact list across a
change of residue numbering silently corrupts it.

**Maps.** Load a map (MRC or CCP4, optionally gzipped) and it is read but not yet drawn. Press
**Show map** to see it as a mesh contoured at a level you choose, in units of the map's own standard
deviation above its mean, and carved to a shell around the model (6 Å by default) because a
whole-map surface at cryo-EM sizes runs to millions of triangles. Tick *Solid and transparent* for a
surface instead of a mesh. The line under the controls reports how many atoms lie inside the
surface at that level, which is the quickest way to choose a level: nearly all of the model inside
at a sensible contour is what a well-fitted model looks like, and a low figure means the level is
too high or the map does not match the model.

With no structure loaded the whole map is contoured, and a raw cryo-EM map is mostly noise, which
at 2 σ breaks into tens of thousands of small separate specks. **Hide small isolated specks**, on
by default, labels the connected pieces of the surface and draws only those that are a worthwhile
fraction of the largest, so the molecule is shown and the dust is not; the line under the controls
says how many specks were hidden. Noise that physically touches the molecule is part of its
connected piece and stays, so a very noisy map still looks fuzzy at the edge. Loading your model as
well restricts the drawing to a shell around it, which removes most of the noise before anything
else is done. Very large maps are sampled every second or third voxel to keep the browser
responsive, and the line says so.

Two failures are reported rather than drawn. If no atom of the model lies inside the map, the
model and the map are not in the same frame, and the usual cause is a wrong origin. If the map
has no variation at all, it is empty or the wrong file. A level that falls in the noise, which
would produce a shredded surface of hundreds of thousands of fragments, is refused with advice to
raise it. The surface is drawn from the same grid mapping that the density scoring uses, so what
you see is what is scored.

**What the model leaves unexplained.** *Find unexplained density* simulates the model's density on
the map's own grid, with the same kernel the density scoring uses, scales it to the map by least
squares near the atoms, and subtracts it. Pieces of what remains are listed and drawn: green where
the map has density no atom accounts for, red where the model has nothing behind it. Each piece is
given with its volume, its peak height and the residues beside it, and is flagged when it sits at a
glycosylation sequon. By default the tool finds the resolution at which the model best matches the
map instead of relying on the value you enter, because a wrong resolution leaves halos round every
atom. It cannot say what unexplained density is: a glycan, a ligand, an ion, a misplaced side chain,
an unmodelled loop and a partner subunit all look alike here.

**Q-scores.** *Score density support* now also reports the Q-score of every atom: how well it sits
at the centre of its own density peak, from map values sampled out to 2 Å and correlated against a
Gaussian of width 0.6 Å, after Pintilie and colleagues. It is shown per residue beside the
correlation, with the value expected for a well-fitted model at the resolution entered.

**Saving the map around a model.** *Save map around model (MRC)* writes the region of the map within a
margin of the model as an MRC2014 file on standard x, y, z axes, with its position in the header, so that
ChimeraX, Coot and other programs place it on the model. Values are copied from the map's own grid points,
nothing is interpolated, and a map whose axes are stored permuted is written in standard order. Tick *Set
density beyond the margin from every atom to zero* for the density around the model alone, which is what a
figure usually needs.

**Fitting.** *Fit model into map* moves the model as a rigid body to where the map density at its
atoms is highest, first on a smoothed map and then on the map itself. It refines a placement that
is roughly right and cannot find one from nowhere. For a predicted model with its PAE file loaded,
*Fit each domain separately* takes the domains from the predicted aligned error, fits each on its
own, and reports the peptide bonds between them, since a domain fitted alone can pull away from
its neighbour. The loaded coordinates change only when you press *Apply*; *Save* writes the fitted
model as PDB.

Everything is processed locally. No coordinates are transmitted anywhere, which is what
makes the tool usable on unpublished or patient-derived structures.

---

## 2 · Contacts

Panel 2 lists the interaction types and lets you switch each on or off. Panel 2's button,
**Map contacts**, runs the detection.

Assignment is from heavy-atom geometry. Hydrogen bonds require a donor–acceptor separation
between 2.2 and 3.5 Å with both antecedent angles above 90°. Salt bridges require charged
nitrogen or oxygen within 4.0 Å. Hydrophobic contacts require apolar carbon or sulfur
side-chain atoms within 4.5 Å, excluding sequence neighbours. Aromatic stacking requires
ring centroids within 5.5 Å with an interplanar angle below 30° for parallel or above 50°
for edge-to-face. Cation–π requires a cationic centroid within 6.0 Å approaching over the
ring face. Metal coordination is any N, O or S within 2.9 Å of a metal. Covalent bonds are
assigned below the sum of covalent radii plus 0.45 Å and then classified as peptide,
disulfide, isopeptide, phosphodiester or other. Steric clashes are van der Waals overlaps
beyond 0.5 Å, excluding pairs that also satisfy a polar criterion — a short polar approach
is a hydrogen bond, not a clash.

Nucleic acids are treated alongside protein. Base donors and acceptors are defined so that
Watson–Crick and Hoogsteen pairing is reported as hydrogen bonding, and purine and
pyrimidine rings take part in the stacking analysis.

The **scope** selector restricts the result to contacts between different chains, within
one chain, or all of them. The Contacts tab lists every contact; the Contact map tab draws
them as a matrix.

---

## 3 · Mutation

Panel 3 selects a chain, a residue and a replacement, and offers three levels of
relaxation: the rotamer search alone, the search followed by local minimisation, or that
plus release of neighbouring side chains within a shell you specify.

The backbone is held fixed throughout and the wild-type coordinates are never altered; the
mutant is built as a separate model. The side chain is constructed from ideal internal
coordinates, with a systematic χ scan refined in five-degree steps, scored by a soft-core
interaction energy against its environment. Where the parent residue is glycine, Cβ is
placed using an improper dihedral measured from a real residue elsewhere in the same
structure, and the resulting configuration is verified to be L by a signed-volume test.

The Before / after tab reports contacts gained and lost, separated into those at the
mutated site and knock-on changes elsewhere. The mutant can be exported as a PDB file.

The fixed backbone is the principal limitation: substitutions that would in reality be
accommodated by backbone movement are systematically overestimated in their disruption.

---

## 4 · Energetics

Panel 4 scores the wild type and the mutant with an empirical function comprising a 12–6
Lennard-Jones term, a Coulomb term with a distance-dependent dielectric of 4r, a
directional hydrogen-bond term, a solvation term from Shrake–Rupley surface area with
atomic solvation parameters, and a side-chain conformational entropy penalty that scales
with burial. Atom pairs separated by three bonds or fewer are excluded from the non-bonded
sums.

This score has no ensemble behind it and therefore no error bar. It is a triage
instrument. The sign, and the ranking within a series of substitutions at the same site,
are what it supports; the absolute value is not, because the function is not parameterised
against measured mutation thermodynamics.

If the structure has more than one chain, an interface interaction energy is reported for
each chain boundary, including the desolvation of the surface each side loses on association
and the buried area itself. The partners are held in their bound conformation and nothing is
sampled, so it ranks interfaces rather than measuring binding; for that, use the alchemical
binding route above.

---

## 5 · Sampled free energy

Panel 5 offers four routes, and the difference between them is the difference between a
number and a measurement.

**Ensemble averaging** runs Metropolis Monte Carlo in torsion space at a stated
temperature and averages the empirical score over the sampled conformations, adding the
side-chain conformational entropy implied by the rotamer populations that emerge.

**Alchemical transformation** is the serious calculation. The wild-type side chain is
decoupled while the mutant is coupled in along a soft-core λ path, with surrounding side
chains resampled at each window, ∂U/∂λ accumulated by central difference and the path
integrated by the trapezoidal rule. Bennett's acceptance ratio is computed on the same
samples as an independent estimator, and the agreement between the two is displayed
alongside the result — a disagreement is the most direct indication of inadequate
sampling. The whole transformation is repeated with the environment removed, and the
difference of the two legs is reported, so the quantity is a double free energy difference
rather than a single one.

**Trajectory averaging** rebuilds and rescores the mutation independently in each frame of
a loaded trajectory.

**Alchemical binding** asks a different question with the same machinery. Instead of taking
the reference leg with the environment stripped away, it repeats the transformation in the
isolated chain, so that everything intramolecular cancels and what remains is the effect of
the substitution on association. A structure with one chain refuses it. Expect more scatter
than for a folding calculation at equal effort, because two independently sampled legs are
subtracted and their noise compounds rather than cancels.

Set the number of independent repeats to more than one. Uncertainty is taken from the
scatter between repeats, not from block averaging within a single run, because the
within-run estimate is systematically optimistic: in a representative case it read 0.25
kcal/mol while independent repeats scattered by about 1 kcal/mol. A single repeat is
reported with an explicit warning that one run cannot measure its own reliability.

---

## 6 · Crosslink geometry

Whether two residues can be crosslinked is routinely decided on straight-line distance.
That is the wrong measurement, because a linker or a reaching side chain must travel around
the protein rather than through it. Panel 6 computes the solvent-accessible surface
distance by Dijkstra search on a voxel grid in which every point within a probe radius of
any atom is blocked.

Two analyses use it. **Screen every Gln–Lys pair** tests each glutamine against each lysine
for geometric competence to form an Nε-(γ-glutamyl)lysine bond: the surface path between
the reactive tips is compared against the distance the two side chains can close by
rotation about their Cβ atoms, with both required to be solvent exposed. **Check a
crosslink list** takes pairs you paste from a cross-linking mass spectrometry experiment
and reports the straight-line and surface distances side by side, so you can see which
restraints a Euclidean cutoff would have accepted incorrectly.

**Screen across the ensemble** asks a different question. Given a trajectory, a multi-model
file, or a series of structures representing an opening, it screens each conformation
separately and correlates competence against how extended the molecule is in that
conformation. A pair competent only in the extended conformations is what a force-gated
reaction would look like: cryptic in the compact state, exposed as the molecule is pulled
open. The range of the extension coordinate is reported alongside, because if your
conformations differ by only a couple of ångströms in span, the correlation is noise.

All of this is geometry. It establishes that a pair is not excluded by distance and
burial, and says nothing about whether factor XIIIa or any other transglutaminase would
accept the site, which depends on sequence context and local conformation in ways no
geometric test captures.

---

## 7 · Network and scanning

The contact map is treated as a graph whose edges are weighted by pairwise interaction
energy rather than by contact count. Panel 7 computes betweenness by Brandes' algorithm,
articulation points whose removal would divide the network, and the strongest
communication path between any two residues with its bottleneck identified.

**Null test** is what makes the centrality interpretable. The network is randomised by
double edge swap, which rewires the topology while holding every residue's degree exactly
fixed, and the observed edge weights are reassigned at random. Observed betweenness is then
compared against that distribution. A residue with many contacts will have high
betweenness by construction; what matters is whether it has more than the wiring alone
would give. On the bundled example this separates six residues that are surprisingly
central from thirty-six that are central only as their degree implies.

Significance is judged on the z-score with a Benjamini–Hochberg correction across
residues. The empirical permutation p-value is reported alongside but is not the basis of
the judgement, because it cannot fall below one over the number of randomisations plus
one — with twelve randomisations across sixty-five residues, no corrected empirical value
could ever reach 0.05. The normal approximation behind the z-score is the assumption to be
aware of: betweenness distributions are skewed, so a large z is evidence of an outlier
rather than an exact probability.

**Saturation scan** builds all nineteen substitutions at one site and ranks them.
**Alanine scan** does the same across an interface, ranking by the change in interface
energy. Both are triage: take the extremes to the alchemical calculation.

---

## 8 · Cavities and pockets

A cavity is found by flooding the solvent grid from its boundary; whatever the flood never
reaches is enclosed by definition. A pocket is found by counting, along seven directions,
how often a free point has protein on both sides, after the manner of LIGSITE. Both report
volume, the largest inscribed sphere, the centroid and the lining residues, and either can
be shown in the viewer as a sphere.

Volume and enclosure are geometric properties of the structure. Whether a pocket binds
anything is not, and is not predicted here.

---

## 9 · Crystal environment

An interface in a deposited file is not necessarily an interface in solution. This panel
reads the unit cell and the crystallographic symmetry operators, expands the lattice, and
reports which neighbours the molecule touches and which of its residues are packed against
them. A residue at a lattice contact is not free in solution, which is worth knowing
before reading anything into its exposure, its B-factor or its apparent flexibility.

It also asks whether each chain pair in the file is related by a crystallographic operator,
and whether the operator generating a stated biological assembly is itself crystallographic.
On 1A8O the answer is pointed: the deposited biological dimer is generated by a
crystallographic two-fold, so its interface is also a lattice contact. That does not make
it spurious — many genuine dimers sit on a crystallographic axis — but the symmetry alone
cannot distinguish the two, and the assignment rests on the depositors' judgement rather
than on anything in the file. A file with the placeholder unit cell used for models and
solution structures is recognised as carrying no crystal information, and refuses to invent
a lattice.

---

## 10 · Evidence

This is the panel the rest of the tool is built around. Density correlation, ensemble
occupancy, predicted aligned error and pLDDT are each mapped onto a common scale and
reduced to one support value per contact, with the **weakest** source governing — a contact
seen clearly in the density but present in a tenth of the frames is not a well supported
contact. Contacts with no independent evidence are left unscored rather than assumed sound,
and if you have supplied nothing the panel says plainly that every result in the session
rests on the coordinates alone.

The propagation is the point. Betweenness is computed twice, on the network as detected and
again with every edge scaled by the support of its contacts, and the difference is reported
per residue. A residue that falls sharply owes its standing to contacts the data do not
sustain. A cut point present in the first network but not the second means the claim that
the structure would fragment there is itself unsupported. The site you are working on
inherits its own qualification, so a ΔΔG or a scan result carries a statement about whether
the geometry it was computed on is actually visible.

To use it, first score density support against a map, or load a structure with several
models or a trajectory, or supply a predicted aligned error for a predicted model.

---

## 11 · Water-mediated bridges

Bridges are detected from water oxygens to polar protein atoms at 2.2–3.5 Å with an
antecedent angle above 90°, as single waters and as two-water chains. Pairs that also touch
directly are marked, since a bridge alongside a direct contact means something different
from a bridge standing in for one.

Across an ensemble, occupancy is defined by residue pair through any water rather than by
the identity of a particular water, which is the only definition that survives the waters
moving. With a single structure there is no way to separate a structural water from one
that happened to be ordered enough to model, and the tool says so.

---

## 12 · Elastic network

A Gaussian network model with a specified cutoff gives predicted fluctuations, slow modes
and hinge residues. The correlation between predicted fluctuations and the deposited
B-factors is reported first and prominently, because it is the test of whether the model
describes this structure at all; a value below about 0.35 is flagged, and the modes should
then be read as a hypothesis rather than a description. Hinges, taken from sign changes in
the slowest mode, are cross-referenced against the articulation points of the contact
network.

The model is isotropic, so it gives magnitudes and correlations of motion but not
directions.

---

## 13 · Conservation

Paste a FASTA or CLUSTAL alignment. Per-column conservation is one minus the Shannon
entropy normalised by its maximum, with Henikoff position-based weighting so that a
subfamily of near-identical sequences does not dominate. The chain is matched to the
alignment automatically and a poor match is flagged.

The output cross-tabulates conservation against contact strength and network centrality.
Residues scoring highly on all three are a far narrower set than any one criterion alone,
and that intersection is usually the interesting one.

---

## 17 · Electrostatics

The linearised Poisson–Boltzmann equation is solved by successive over-relaxation on a
dielectric map built from the structure, with Debye screening in the solvent and boundary
values taken from the Debye–Hückel potential of the charges rather than clamped to zero.
From it come the potential anywhere, the surface potential residue by residue, the net
charge and the dipole moment.

The pKa estimate has two terms. Desolvation is the cost of carrying the charge where the
dielectric is low, computed from effective Born radii obtained by direct integration and
measured against the same group in an isolated model compound. The background is the
group's interaction with every other charge, read from a Poisson solution in which that
group's own charges are switched off, so the screening is solved for rather than assumed.

The practical output is the list of groups whose protonation is probably not what the rest
of the tool assumed: the energy function fixes protonation and treats histidine as neutral,
and any mutation or free energy involving a contradicted group carries that as a known
error.

Interactions between titrating sites are not treated — each is computed against the others
held at standard protonation, which is the usual single-site approximation and not a
titration calculation. Charges are a heavy-atom point model, so groups whose charge is
delocalised, tyrosine above all, have their desolvation overstated.

---

## 18 · Glycans

Modelled sugars are detected, assembled into trees by connectivity, and linked to their
attachment residue as N-linked or O-linked. Sequons — N-X-S/T on consecutive residues with
X not proline — are found independently, so you can see which are occupied and which carry
nothing.

Modelled sugars cast their real volume with a margin for hydration and motion. Unoccupied
sequons optionally get an exclusion sphere along the asparagine side chain, standing in for
a glycan that is very likely present in the protein but absent from the model. That shadow
is then applied to results already computed: competent Gln–Lys pairs with a reactive tip
inside it are flagged and should be discounted, and inter-chain contacts falling under
carbohydrate are counted, since an interface substantially covered by sugar is unlikely to
be the one the molecule uses.

The sphere is a crude proxy. Real glycans are branched, flexible and anisotropic, and it
reproduces neither their shape nor their motion. It exists to stop a site being called
accessible when a sugar almost certainly covers it, and should not be read as a structure.

---

## 16 · Mechanics

Give two attachment points, or leave the boxes empty to pull on the termini of the first
chain. Each contact is placed along that axis and given a resistance from its type, scaled
by how far its own direction lies across the pull rather than along it — a contact loaded
in shear resists far more than the same contact loaded in peel. Every cut perpendicular to
the axis is then held together by whatever crosses it, and the weakest such cut is where the
structure would part first. Out of that come a rupture order, the load-bearing set at the
bottleneck, and a per-residue load ranking, a cluster of which is what is usually meant by a
mechanical clamp.

This is a ranking of exposure to a pull, not a force. There are no piconewtons here.
Contacts are treated as breaking independently when in reality they fail cooperatively, and
the structure is not allowed to relax as it is loaded. Changing the attachment points
changes the answer completely, which is the physics rather than a defect.

---

## 14 · Atomic force microscopy

The tool simulates what an AFM tip would trace over the molecule: it lays the structure, or
a density map above a contour, on the substrate, builds the envelope, and dilates it by a
tip of given radius closing into a cone. That is standard morphological tip convolution with
no free parameters beyond the tip itself. The height is preserved; the lateral size is
broadened.

Given a measured topograph as a text matrix, it searches orientations and reports the one
that best reproduces it, with measured, fitted and difference panels. The score combines
shape correlation with absolute height agreement, because a correlation alone is blind to
scale and would accept a rod stood on end as a match for a compact molecule.

It also counts how many orientations differing by more than thirty degrees fit about as
well, and says plainly when the image does not determine which face is up. That number
matters more than the correlation: a poor fit is the informative outcome, since many shapes
give similar topographs at this resolution, while disagreement in height or footprint is
strong evidence that something is wrong. Measured heights are systematically low because
the tip presses into the molecule, by an amount no calculation here can supply.


**Fitting an AFM image against a cryo-EM map.** With a map loaded, enter a contour level in *From map at*
and the orientation fit uses the map's envelope above that level instead of the model. The result is how the
density must lie on the surface to reproduce the image: which face of the cryo-EM density was facing the tip,
and, in the difference panel, where the measured surface and the density disagree. The contour level decides
the envelope's size and so its simulated height; choose it as you would for display, and expect measured
heights to sit somewhat below the simulated ones because the tip presses into the molecule. As with a model,
read how many orientations fit about as well as the best: in testing, the same image fitted with the atomic
model scored well in an orientation turned over relative to the true one.

**Preparing a measured image.** A raw topograph carries the scanner's tilt, an offset that differs from
one scan line to the next, and the odd spike, and all three pass straight into a fit unless removed.
On loading, spikes are replaced by the median of their neighbourhood; the background is fitted, as an
offset, a plane or a second-order surface, together with one offset per scan line, using only pixels
classed as substrate; and the substrate is set to zero at its median, not its lowest point, which one
spike below the surface would otherwise decide. The class is seeded from the lowest pixels of every
segment of every scan line, so that neither a tilt nor a line offset can leave part of the image
without substrate, and re-estimated until it settles. What was done is reported: spikes removed, tilt,
the spread of line offsets, and the substrate roughness before and after. With too little bare
substrate for a curved background, the order is lowered and the reason given. Say whether scan lines
run along rows or columns of the file, since that decides which offsets are corrected.

**How blunt was the tip?** *Bound the tip radius from the image* uses the fact that an image can never show
a feature sharper than the tip that traced it: around every summit the image falls by no more than the tip
rises, so the image puts an upper limit on the tip's radius. It is a bound, not an estimate. It is close to the
truth only when the surface has features sharper than the tip, and it assumes the tip is a sphere joined to a
cone of the half-angle set in the panel. Noise is allowed for, so more noise loosens the bound without breaking
it. If the tip radius set in the panel is blunter than the image allows, the tool says so, because simulations
and fits made with that tip cannot reproduce what was measured.

**Flexible fitting.** After a rigid orientation fit, *Flexible fit along normal modes* deforms the model
along the softest motions of an anisotropic network, springs between Cα atoms, to better match the measured
image. Unlike the Gaussian network of panel 12, this model gives the direction of each motion, which a shape fit
needs. Its modes were checked against a full eigendecomposition of the same matrix and agree to eleven
significant figures. A penalty favours soft motions over stiff ones, and any deformation that changes a
consecutive Cα spacing by more than the strain limit is refused, because linear modes stretch the chain at large
amplitudes. Large proteins are coarse-grained to at most 400 beads. A height image is blind to some motions, so
different combinations of modes can explain the same image: the result is one deformation consistent with the
image, not a measurement of how the molecule moved. Test it against images not used in the fit before drawing a
conclusion. Save the deformed model as PDB with the button under the results.

**Comparing candidate structures.** Load several candidate structures and several images, and each
structure is fitted to each image with the same full orientation search a single fit uses. A coarser
search would be faster and wrong: a candidate whose true orientation is missed scores low and loses to
a wrong structure, and the ranking then looks confident. Structures are ranked by how many images they
win; an image on which the two best structures score within 0.05 of each other prefers neither and is
counted as a tie. A ranking says which candidate the images favour among those supplied. It cannot say
that any of them is right.

A long comparison can run for hours. Progress is saved in the browser as each fit finishes, so if the page
is closed, reloaded or stopped with *Stop after the current fit*, choosing the same files and settings and
running again resumes it without repeating anything; a changed setting starts afresh rather than mixing
results. The run continues when its tab is not in front, but the computer must stay awake: the page asks
the browser to keep the screen on, and shows a notification when it finishes. It cannot send email itself,
because nothing leaves your computer; *Write an email with the summary* opens your mail program with the
summary written in, and the CSV must be attached by hand. For hundreds or thousands of structures use the
command-line runner instead, which spreads the work over many processor cores and can email you itself:
see the validation package's README.
---

## 15 · Structural dendrogram

Select several structures. Each pair is aligned geometrically — superpose, reassign every
residue to its nearest partner within a shrinking cutoff, repeat, keeping the assignment
monotonic so it cannot cross itself — and the alignments give a distance matrix, from
either RMSD or one minus the TM-score. Trees are built by neighbour joining or UPGMA, with
bootstrap support from resampling the aligned positions, and export as SVG and Newick.

It is a dendrogram of structural similarity and not a phylogeny. Structural distance
saturates: two proteins that diverged long ago and two that diverged very much longer ago
may both sit at the same RMSD with the same fold. It also responds to conformational state,
crystallisation and resolution as readily as to divergence. Read the clustering; do not read
the branch lengths as time or the arrangement as descent.

---

## 19 · Design

Two scans ask where a feature could be placed rather than where one already is.

**Engineered disulfides** models cysteine at every position over a fine χ1 scan and tests
every pair within reach, scoring strain as the summed squared deviation from the ideal bond
— 2.05 Å between the sulfurs, 104° at each Cβ–S–S angle, ±87° for the dihedral.
Conformers clashing with the rest of the structure are discarded, and because both
positions are being replaced, neither one's existing side chain counts as an obstacle to
the other.

**Crosslink sites** models glutamine at one position and lysine at the other, tries both
assignments, and reports a site when some pair of clash-free conformers brings the lysine
NZ within bonding reach of the glutamine CD. Only solvent-exposed positions are considered.

Geometry is a necessary condition and not a sufficient one. A low-strain site is a
candidate to test, not a prediction.

---

## 20 · Binding hotspots

Ranks every interface residue by what replacing it costs the interface, scores the same
substitution against the whole structure, and removes the candidates that would waste a
mutant: residues in a disulfide, coordinating a metal, carrying an isopeptide, whose only
partner is a crystallographic mate, or lying under a modelled glycan. Poorly supported
contacts and uncommitted predicted aligned error register as cautions. Substitutions are
proposed to suit the residue rather than defaulting to alanine throughout: charge reversal
for a salt bridge, tryptophan to phenylalanine rather than to alanine, tyrosine to
phenylalanine to separate hydrogen bonding from stacking. The leading candidates can then be
measured by the alchemical binding calculation.

The structural cost uses the empirical score of the mutation panel, with solvent accessibility
taken over the residues within 15 Å of the site; it is the same rigid substitution as the
interface cost, not a relaxed or sampled one. (Before version 1.1.0 this column was empty and
every candidate carried a "could not be modelled" caution, because the check failed; the
interface ranking was not affected.)

Read the two costs together. A large interface cost with a small structural cost is a clean
experiment; both large may simply misfold, and a binding assay would then report a loss that
says nothing about the interface. Check your construct numbering before ordering anything,
and treat the list as a prioritisation of experiments rather than a substitute for them.

---

## 21 · Batch

Run one analysis across many structures at once: interaction summary, Gln–Lys competence,
cavities and pockets, chain interfaces, or water bridges. Each file is parsed and analysed
in isolation, the biological assembly is built where the file provides one, and the unit
used is recorded in its own column because it changes the answer. A file that cannot be
read is reported with its reason and the rest continue. Results export as CSV, and for the
crosslink screen every competent pair from every structure is pooled into a single ranked
table.

The single-structure session is untouched by a batch run.

---

## 22 · Reproducibility

**Save session manifest** records a hash of every input — structure, map, PAE, trajectory,
alignment, second structure — along with the settings and a digest of the results, using
the platform's cryptographic hashing where available and a clearly labelled fallback where
it is not.

**Verify a manifest** loads one back and checks it line by line: each input hash against
the file now loaded, the settings against those recorded, each result group against what
this session computed, and finally the overall digest. Missing and differing are reported
separately, because an input not loaded here is not a discrepancy but neither is it a
confirmation. Only lines marked as matching were actually verified.

Keep the manifest beside the figure it supports and anyone can establish later what
produced the numbers.

---

## Export

The Export panel produces a written PDF report carrying every section you have run, with its
charts drawn as vectors beside the tables; a machine-readable JSON report with every computed
section and its parameters; a contacts CSV; the mutant coordinates; and the charts
individually as SVG. The **Charts** tab assembles every plot the current session supports; each
saves individually.

If a download is refused, the text appears in the Export tab instead and can be copied. On
a page hosted with its own security policy — your own site, or a local file — downloads and
the fetch buttons work normally.

---

## Reporting a problem

If something fails, a red banner appears across the top of the work area with a **Copy
diagnostics** button. That records the browser, the screen size, whether WebGL is
available, what is loaded, and the full log. Send that text with a description of what you
were doing.
