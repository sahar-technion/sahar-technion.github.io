---
title: SMILES the language of Cheminformatics
description: SMILES are the lowest common denomenator used to explain the chemical world to computer. And like with any new journey the best way to star is to understand the language.
date: 2026-07-09 12:00:00 +0300
categories: [Retrosynthesis, Background]
image:
  path: /assets/images/SMILES_rings.png
math: true
---
## Intro
Just as mathematics relies on numbers and operators, every specialized language has its own symbols, syntax, and semantics to convey complex ideas. The structural design of a chemical language is crucial: it dictates what can be expressed, how efficiently it can be communicated, and where the primary analytical focus lies.
In case of chemistry and especially organic chemistry we have many recurrences of carbons and hydrogens, so to simplify the notation we could decide to just infer them like in the skeletal formulae. But since we still need to understand the full structure we choose to note the carbon atoms. 
Another dilema in simplification and focus could be functional groups. Since chemical reactions and properties are mostly based on functional groups it could be useful to have the language based on them with more basic building blocks as connectors, like in IUPAC naming. But since we are trying to allow discovery of new functionalities and relations between atoms, looking at the molecule as a combination of functional groups can be very limiting.
Ideally, we want every "word" written in a chemical language to have real-world physical meaning—representing a structurally stable, valid molecule. Unfortunately, this is a known limitation of SMILES strings. Despite this caveat, SMILES remains the gold standard within the cheminformatics community. The vast majority of modern libraries, toolkits, and machine learning models are fundamentally built around it, making it the most practical language to base your work upon rather than its modern successor, SELFIES.

## The Rules
### Atoms and symbols
Most common organic atoms can be written directly without brackets: 
`C`, `N`, `O`, `S`, `P`, `F`, `Cl`, `Br`, and `I`. Atoms outside that “organic subset,” atoms with charges, isotopes, explicit hydrogens, unusual valence, or atom maps usually go in brackets, like `[NH4+]`, `[13C]`, or `[O-]`.
Lowercase letters mean aromatic atoms, so `c`, `n`, `o`, `s`, `p` represent aromatic versions of those elements. That is why benzene is often written `c1ccccc1`, while cyclohexane is `C1CCCCC1`.
Bonds and branching
Single bonds are usually implied, while `=` means double, `#` means triple, and `:` can represent aromatic bonding. Branches are written in parentheses, so `CC(C)C` means isobutane and `CC(=O)O` means acetic acid.
Parentheses always attach the branch to the atom immediately before them. Nested branching is allowed, which makes SMILES concise even for heavily substituted frameworks.
### Rings
Rings are encoded by numbering the two atoms that are connected across the ring closure. For example, `C1CCCCC1` closes a six-membered saturated ring, 
and `c1ccccc1` closes an aromatic six-membered ring.
If a molecule has fused or bridged rings, the same atom can carry more than one ring-closure digit. That is how structures like decalin or polycyclic frameworks stay linear in notation even when the underlying graph is not.

<img src="{{ '/assets/images/SMILES_rings.png' | relative_url }}" alt="SMILES Ring Closure Diagrams" width="40%" data-proofer-ignore>
<!-- <img src="/assets/images/SMILES_rings.png" alt="SMILES Ring Closure Diagrams" width="40%" data-proofer-ignore> -->

### Stereochemistry
Tetrahedral chirality uses `@` and `@@` inside bracketed atom specifications. The exact meaning is defined relative to the order of neighbors in the SMILES string, so `N[C@@H](C)C(=O)O` encodes one enantiomer of alanine, while the opposite marker gives the other configuration.

<img src="/assets/images/SMILES_stereochemistry.png" alt="SMILES Stereochemistry Diagrams" width="30%" data-proofer-ignore>

For double bonds, geometric stereochemistry is written with slash notation: `C/C=C/C` and `C/C=C\C` distinguish the two alkene configurations. In practice, the slashes indicate the relative orientation of substituents around the C=C bond, so this is how SMILES captures E/Z information.

<img src="/assets/images/SMILES_EZ.png" alt="SMILES EZ Diagrams" width="30%" data-proofer-ignore>

### Reaction notation
Reaction SMILES extend the same language to chemical reactions by separating components with `>`. The usual format is `reactants`>`reagents`>`products`, with multiple species on each side separated by dots.
Atom mapping can also be added in bracketed atoms to track which atoms correspond across reactants and products. That makes reaction SMILES useful not only for storage, but also for cheminformatics tasks like matching, transformation analysis, and machine-readable synthesis data.

## Let's Practice
The best way to see if you actually understood a topic is by practicing, but instead of making it boring let's try using a game. 
Try to find the molecule that corresponds to the following SMILES strings. Good Luck!
<iframe
	src="https://saharsall-find-the-smiles.hf.space?__theme=light"
	frameborder="0"
	width="100%"
	height="800px"
	style="border: 1px solid #e5e7eb; border-radius: 8px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05);"
></iframe>

<iframe
	src="https://saharsall-smiles-like-me.hf.space?__theme=light"
	frameborder="0"
	width="100%"
	height="800px"
	style="border: 1px solid #e5e7eb; border-radius: 8px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05);"
></iframe>
