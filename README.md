# CrashCourse-Phylogenomics

This hands-on course will introduce you to the awesomeness of phylogenomics.

You will learn the most fundamental steps in the phylogenomics pipeline, allowing you to go from a bunch of sequences from different species to a phylogenetic tree that represents how these species are related to each other in evolutionary terms. 

The phylogenomics pipeline can become very complex, many additional steps might be included (particularly at the stage of dataset assembly!) and some analyses can take weeks to complete. Pipelines are modular, meaning they can (and should) be improved, as well as modified to the particular question at hand. Here, we will cover the basics of a phylogenomic pipeline: identification of homologs, multiple sequence alignment, alignment trimming and phylogenetic inference with concatenation and coalescent methods.



## Objective and data


We will use a dataset [from this paper](https://academic.oup.com/sysbio/article/65/6/1057/2281640). The starting point is a subset of proteins obtained from genomes/transcriptomes for 23 species of vertebrates. We aim to reconstruct these species' phylogeny using concatenated and coalescent approaches. In practice, we are using a small subset of the genomes/transcriptomes of these species to speed up computations.

Let's start by cloning this repository. The starting data can be found in the `vertebrate_proteomes` folder.

<details>
  <summary>Need help?</summary>
  
```
git clone https://github.com/iirisarri/PEB_Phylogenomics.git

```
</details>


You will see 23 fasta files, each containing a set of proteins from a different species.



## Inferring ortholog groups


The first step is to identify homologs among all the proteins. We will use [OrthoFinder](https://github.com/davidemms/OrthoFinder) for this task. We will use default parameters, just providing the folder containing the proteome files.

```
orthofinder -f vertebrate_proteomes
```

Look for Orthofinder's results in your `vertebrate_proteomes` folder. A bunch of interesting information is contained there. The Orthogroups can be found in `Orthogroup_Sequences`. Each file corresponds to one orthogroup ("gene") containing one sequence per species.

Let's check what the orthogroups look like.

<details>
  <summary>Need help?</summary>
  
```
e.g. less -S OG0000006.fa
```
</details>


You will see that sequences are named `Genus_species_GENE_XXXX`. To concatenate genes at a later stage, sequences of the same taxa need to have uniform names. Thus, we can remove the gene names now. Can you simplify current names to being `Genus_species`?

<details>
  <summary>Need help?</summary>
  
```
for f in *fa; do awk -F"_" '/>/ {print $1"_"$(NF-2)}; !/>/ {print $0}' $f> out; mv out $f; done

# Explanation: loop through the files, modify the input file and save it to "out", overwrite the input with it. 
# Awk: using the "_" field separator, modify lines starting with ">" (sequence names) so that they will only 
# contain the second and second last elements separated by an underscore (>Genus_species).
# Print all other lines not starting with ">" (i.e. sequences) without modification.

# An easier version:
for f in *fa; do sed -E '/>/ s/_GENE.+//g' $f > out; mv out $f; done
# Explanation: in headers (lines starting with ">"), remove everything after "_GENE"
# Can you guess the difference with using awk? Look at Canis_lupus_familiaris
```
</details>

Don't forget to check the output: is your command doing what you want?


**NOTE ABOUT ORTHOLOGY**: Ensuring orthology is a difficult issue and often using a tool like Orthofinder might not be enough. Paralogy is a tricky business! Research has shown (e.g. [here](https://www.nature.com/articles/s41559-017-0126) [here](https://academic.oup.com/sysbio/article/71/1/105/6275704?login=false) or [here](https://academic.oup.com/mbe/article/36/6/1344/5418531)) that including paralogs can bias the phylogenetic relationship and molecular clock estimates, particularly when the phylogenetic signal is weak. Paralogs should always be removed before phylogenetic inference. But identifying them can be difficult and time-consuming. One could build single-gene trees and look for sequences producing extremely long branches or clustering outside of the remaining sequences. [Automatic pipelines](https://github.com/fethalen/phylopypruner) also exist.


## EXTRA: Pre-alignment and quality filtering

Often, transcriptomes and genomes have stretches of erroneous, non-homologous amino acids or nucleotides, produced by sequencing errors, assembly errors, or errors in genome annotation. But until recently, these type of errors had been mostly ignored because no automatic tool could deal with them.

We will use [PREQUAL](https://academic.oup.com/bioinformatics/article/34/22/3929/5026659?login=true), a software that takes sets of (homologous) unaligned sequences and identifies sequence stretches (amino acids or codons) sharing no evidence of (residue) homology, which are then masked in the output. Note that homology can be invoked at the level of sequences as well as of residues (amino acids or nucleotides). 

Download and Iinstall PREQUAL:
```
git clone https://github.com/simonwhelan/prequal
cd prequal
make
```

Running PREQUAL for each set orthogroup is easy:
```sh
for f in *fa; do prequal $f ; done
```
The filtered (masked) alignments are in .filtered whereas .prequal contains relevant information such as the number of residues filtered.


## Multiple sequence alignment


The next step is to infer multiple sequence alignments from orthogroups. Multiple sequence alignments allow us to *propose* which amino acids/ nucleotides are homologous. A simple yet accurate tool is [MAFFT](https://mafft.cbrc.jp/alignment/server/).

We will align gene files separately using a for loop:

```
for f in *fa; do mafft $f > $f.mafft; done
```


## Alignment trimming


Some gene regions (e.g., fast-evolving) are difficult to align and thus positional homology can be uncertain. It is unclear (i.e., problem-specific) whether trimming suspicious regions [improves](https://academic.oup.com/sysbio/article/56/4/564/1682121) or [worsens](https://academic.oup.com/sysbio/article/64/5/778/1685763) tree inference. However, gently trimming very incomplete positions (e.g. with >90% gaps) will speed up computation in the next steps without significant loss of phylogenetic information .

To trim alignment positions we can use [ClipKIT]([https://bmcevolbiol.biomedcentral.com/articles/10.1186/1471-2148-10-210](https://github.com/JLSteenwyk/ClipKIT)) but several other software are also available.

To remove alignment positions with > 90% gaps:

```
for f in *mafft; do clipkit $f -m gappy; done
```

While diving into phylogenomic pipelines, it is always advisable to check a few intermediate results to ensure we are doing what we should be doing. Multiple sequence alignments can be visualized in [SeaView](http://doua.prabi.fr/software/seaview) or [AliView](https://github.com/AliView/AliView). Also, one could have a quick look at alignments using command line tools (`less -S`).


## Concatenate alignment


To infer our phylogenomic tree we need to concatenate the trimmed single-gene alignments we generated. There are many tools that you can use for this step (e.g [concat_fasta.pl](https://github.com/santiagosnchez/concat_fasta) or [catsequences](https://github.com/ChrisCreevey/catsequences)). Here, we will use [FASconCAT](https://github.com/PatrickKueck/FASconCAT-G), which will read in all `\*.fas` `\*.phy` or `\*.nex` files in the working directory and concatenate them (in random order).

```
for f in ../*clipkit; do mv $f $f.fas; done
mkdir concatenation/
mv *clipkit.fas concatenation/

cd concatenation/
perl /software/FASconCAT-G_v1.04.pl -l -s
```

Is your concatenated file what you expected? It should contain 23 taxa and 21 genes. You might check the concatenation (`FcC_supermatrix.fas`) and the file containing the coordinates for gene boundaries (`FcC_supermatrix_partition.txt`). Looking good? Then your concatenated dataset is ready to rock!!



## Concatenation: Maximum likelihood


One of the most common approaches in phylogenomics is gene concatenation: the signal from multiple genes is "pooled" together with the aim of increasing resolution power. This method is best when among-gene discordance is low.

We will use [IQTREE](http://www.iqtree.org/), an efficient and accurate software for maximum likelihood analysis. Another great alternative is [RAxML](https://github.com/stamatak/standard-RAxML). The most simple analysis is to treat the concatenated dataset as a single homogeneous entity. We need to provide the number of threads to use (`-nt 1`) input alignment (`-s`), tell IQTREE to select the best-fit evolutionary model with BIC (`-m TEST -merit BIC -msub nuclear`) and ask for branch support measures such as non-parametric bootstrapping and approximate likelihood ratio test (`-bb 1000 -alrt 1000 -bnni`):

```
iqtree -s FcC_supermatrix.fas -m TEST -msub nuclear -bb 1000 -alrt 1000 -nt 1 -bnni -pre unpartitioned
```

A more sophisticated approach would be to perform a partitioned maximum likelihood analysis, where different genes (or other data partitions) are allowed to have different evolutionary models. This should provide a better fit to the data but will increase the number of parameters too. To launch this analysis we need to provide a file containing the coordinates of the partitions (`-p`) and we can ask IQTREE to select the best-fit models for each partition, in this case, according to AICc (more suitable for shorter alignments).

```
iqtree -s FcC_supermatrix.fas -p FcC_supermatrix_partition.txt -m TEST -msub nuclear -merit AICc -bb 1000 -alrt 1000 -nt 1 -bnni -pre partitioned
```

Congratulations!! If everything went well, you should get your maximum likelihood estimation of the vertebrate phylogeny (`.treefile`)! Looking into the file you will see a tree in parenthetical (newick) format. See below how to create a graphical representation of your tree.


## EXTRA: Bayesian Inference 
The next step would be to run a Bayesian analysis using Phylobayes, however, due to time constraints, we will provide you with the Bayesian topology. Compare it with the maximum likelihood one and check if you find any difference. 

This is the command we used for phylobayes

```sh
#Chain 1
pb_mpi  -d FcC_supermatrix.fas  -cat  -gtr  chain1

#Chain2
pb_mpi  -d  FcC_supermatrix.fas  -cat  -gtr  chain2

#To check for convergence
bpcomp -x burnin chain1  chain2

#To get the parameters
tracecomp -x burnin chain1  chain2
```


## Coalescence analysis


An alternative to concatenation is to use a multispecies coalescent approach. Unlike maximum likelihood, coalescent methods account for incomplete lineage sorting (ILS; an expected outcome of evolving populations). These methods are particularly useful when we expect high levels of ILS, e.g. when speciation events are rapid and leave little time for allele coalescence.

We will use [ASTRAL](https://github.com/smirarab/ASTRAL), a widely used tool that scales up well to phylogenomic datasets. It takes a set of gene trees as input and will generate the coalescent "species tree". ASTRAL assumes that gene trees are estimated without error.

Thus, before running ASTRAL, we will need to estimate individual gene trees. This can be easily done by calling IQTREE in a for loop:

```
for f in *clipkit; do iqtree -s $f -m TEST -msub nuclear -merit AICc -nt 1; done
```

After all gene trees are inferred, we should put them all into a single file:

```
cat *clipkit.treefile > my_gene_trees.tre
```

Now running ASTRAL is trivial, providing the input file with the gene trees and the desired output file name:

```
java -jar /srv/evop/resources/astral/ASTRAL/Astral/Astral.5.7.8.jar -i my_gene_trees.tre -o species_tree_ASTRAL.tre 2> out.log
```

Congratulations!! You just got your coalescent species tree!! Is it different from the concatenated maximum likelihood trees? 



## Tree visualization


Trees are just text files representing relationships with parentheses; did you see that already? But it is more practical to plot them as a graph, for which we can use tools such as [iTOL](https://itol.embl.de), [FigTree](https://github.com/rambaut/figtree/releases), [iroki](https://www.iroki.net/), or R (e.g. [ggtree](https://bioconductor.org/packages/release/bioc/html/ggtree.html), [phytools](http://www.phytools.org/); see provided R code).

Upload your trees to iTOL. Trees need to be rooted with an outgroup. Click in the branch of *Callorhinchus milii* and the select "Tree Structure/Reroot the tree here". Branch support values can be shown under the "Advanced" menu. The tree can be modified in many other ways, and finally, a graphical tree can be exported. Similar options are available in FigTree.

[Well done!](https://media.giphy.com/media/wux5AMYo8zHgc/giphy.gif)


## Software links

* Orthofinder (https://github.com/davidemms/OrthoFinder)
* PREQUAL (https://github.com/simonwhelan/prequal)
* MAFFT (https://mafft.cbrc.jp/alignment/software/source.html)
* MUSCLE v5 (https://github.com/rcedgar/muscle)
* TrimAL (https://vicfero.github.io/trimal/)
* FASTCONCAT (https://github.com/PatrickKueck/FASconCAT-G)
* IQTREE2 (http://www.iqtree.org/)
* Phylobayes (https://github.com/bayesiancook/phylobayes/tree/master)
* ASTRAL (https://github.com/smirarab/ASTRAL)
* FIGTree V1.4.4 (https://github.com/rambaut/figtree/releases) 
* TreeViewer (https://treeviewer.org/)
* iTOL (https://itol.embl.de/)
  
