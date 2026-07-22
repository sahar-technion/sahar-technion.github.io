---
title: Algorithmic Atom to Atom Mapping
description: If we want to understand and predict reactions we first need to understand what is happening within the reactions, or in other words which atom goes where? Sounds simple? Well if you're a computer not so much.
date: 2026-07-20 12:00:00 +0300
categories: [Retrosynthesis, Background]
math: true
---
## Intro
Great! We now know how to give the computer a reaction to look at, but how can the machine learn what is a likely reaction and what isn't? First we need to realize that in every reaction most of the molecule\s don't change, which is why we have named reactions and generalized reaction types, instead of just infinite different unrelated reactions. If we could get a model to learn these different "templates" it could then identify the template most fitting to the reactants and predict the products, or think of different reactants that could lead to a specific product. To generate these templates, we need to identify what part of the molecule is static and what changes, and to do that we first need to map every atom in the **products** to an atom in the reactants, so when we generate the template we do it correctly. For example:

<img src="/assets/images/esterification_mapping_example.png" alt="Esterification atom mapping" width="460%" data-proofer-ignore>

Since models need large datasets to train on, we don't want to manually insert each of these mappings. Instead let's try to generate an algorithm to do it for us.

## The Steps
### SMILES to Graphs

Unlike SMILES that are easy to read and generate but have less of a spacial meaning, when we try to look at whats happening in specific given reactions - it's better to look at structural features. 
To do so, we want to look at our molecules as graphs (Where a node represents an atom and an edge represents a bond) and see how these graphs change between the reactants and the products. 
*Also, since we are looking at a given reaction we don't need to worry about graph->molecule feasabilty since we aren't inventing anything new, so we don't care if our "language" contains impossible molecules as we do in other tasks.

<img src="/assets/images/SMILES_to_graph.png" alt="Step by step conversion of SMILES format to graph" width="60%" data-proofer-ignore>

#### The Implementation
```
1. Parse the SMILES:
   - create one node for each atom in the SMILES 
   		- We will ignore Hydrogens to simplify the computational complexity but in theory we can account 
		  them too by calculating valency.
   - Tag each node with element, aromatic/non‑aromatic, etc.
2. Add bonds:
   - For each each atom in SMILES connect it (with an edge) to the atom before it ignoring brackets.
   - For numerical indices add an edge between nodes with the same index (for rings)
   - Tag each edge with bond order (single, double, aromatic…).

* in practice RDKIT has a function for this
```

### Mapping the Maximal Common Scaffold (MCS)

As we previously noted, in most reactions - a large part of each molecule stays exactly the same. Because of this, it would be useful to map the biggest identical piece between the products and the reactants (it doesn't have to be one big connected piece, but rather the match that contains the most atoms) since we assume that would be the case for correct mapping. To improve computation efficiency we will take too shortcuts that are actually incorrect for solving this in reality:
* We will accept upto one change in neighbors and bonds in total (since this can very well happen in a reaction it will usually be the correct mapping).
* Once we find a biggest piece (a piece with the highest number of nodes) we won't look for a better assignment - this will reduce many "branches" of checks down the road.

#### The Implementation
```
1. Generate candidate matches:
   - Start from pairs of atoms with same element (C-C, O-O, etc.).
   - Extend matches by adding neighbouring atoms if
     bond types and neighbors still match (upto one difference).
2. Keep only consistent matches:
   - If two candidates contradict (e.g., same atom matched to
     two different atoms), discard the smaller/less consistent one.
3. Choose the largest consistent match:
   - Count atoms in each candidate.
   - Keep the candidate with the most atoms as the common scaffold.
```


### Mapping the Leftovers

After we mapped the main structure, we might still have atoms that either appear only in one side (products or reactants) or that had multiple changes and therefore weren't mapped in the Common Scaffold phase. Here we would like to test out all of the remaining options of mapping leftover atoms from the same type. This is viable since we assume we covered most of the atoms in the previous phase. But how do we choose the right option between all the mappings? We build a scoring function - and the combination with the lowest score (least required changes) should be the mapping that makes the most sense from a chemists perspective. Things we can penalize for:
* Changes in bond order
* Changes in neighbors (which can actually count as changes in bond order)
* Number of leftover atoms - maybe it's better to have a leftover atom than an atom that doesn't make sense because of all the changes it implies...
* Changes in stereochemistry
* Change in charges
* Changes in local environment - Is it part of a ring or a chain? Aromatic or aliphatic? In each of the rection sides
*to simplify things we will just do the first three.

#### The Implementation
```
Input:
1. For each unmapped reactant atom R and each unmapped product atom P of the same element try to match them up
2. Compute a total cost for the tested mapping (based on the rules we chose)
3. Save the lowest cost so far and the mapping that correspnds to it
```

## Let's See it in action
The best way to see if you actually understood a topic is by practicing, but instead of making it boring let's try using an interactive implementation. 
You can try to see how the example reaction is mapped using our algorithm, try your own reaction (you should start by previewing the molecules to check that your input is correct) or try any of the reactions below and try to understand why the algorithm is or isn't working for them.

<iframe
  src="https://algorithmic-atomtoatom-mapping.streamlit.app/?embed=true&__theme=light"
  frameborder="0"
  width="100%"
  height="800px"
  style="border: 1px solid #e5e7eb; border-radius: 8px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05);"
></iframe>

---

---

* Mannich Reaction: O=C1CCCCC1.CNC.CO >> O=C1C(CN(C)(C))CCCC1
<details>
  <summary>Does it work?</summary>
  
  Nope🥲, $C_{10}$ in the reactants should be mapped to $C_3$ in the products but instead it is mapped to $C_6$. 
  There are two issues in this case:
  1. We are only looking at the first best mapping in MCS and since it's not the correct option we get a wrong result
  2. Even if we would consider other options, the cost of the correct mapping would be the same. We need to give better scoring to mapping that conserve symmetries.
</details>

---

* Clayden Condensation: c1ccccc1C=O.CC(=O)C >> c1ccccc1C=CC(=O)C
<details>
  <summary>Does it work?</summary>
  
  Yes! 🥳
</details>

---

* Dies Alder: C=CC=COC.C=CC(=O)OC >> COC1C(C(=O)OC)CCC=C1
<details>
  <summary>Does it work?</summary>
  
  Nope🥲, the mapping of the ring atoms to `C=CC=COC` is off by one since the conjugation of the double bonds in the reaction causes multiple changes.
  Because of this the best MCS isn't the right choice in this case (even if our implementation wouldn't accept even a single difference between the products and the reactants).
</details>
