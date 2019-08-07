F-statistics exercise notes
==

## Generating the dataset
We are going to use the dataset that you used yesterday, with Fernando, for the
PCA and admixture analyses. But before we proceed, we are going to convert the
files to the [EIGENSTRAT](https://github.com/DReichLab/EIG/tree/master/EIGENSTRAT)
format. To do this we are going to use a tool called `convertf`. `convertf` uses
a parameter file - let us call it plink2geno.par. This is given below.
```
MYDATA: [YOUR DATA DIRECTORY]
DATA: /science/groupdirs-nfs/SCIENCE-SNM-Archaeo/2019/Day4/data
genotypename:   DATA/AncientModern.bed
snpname:        DATA/AncientModern.bim
indivname:      DATA/AncientModern.pedind
outputformat:    EIGENSTRAT
genotypeoutname: MYDATA/AncientModern.geno
snpoutname:      MYDATA/AncientModern.snp
indivoutname:    MYDATA/AncientModern.ind
```
Now we can go back to the command line and run convertf.
```
$ convertf -p plink2geno.par
```
Finally, before we run any further analysis, we need to change the format of our ind file
so that other programs can parse it properly.
```
cut -f 2 -d ":" [YOUR DATA DIRECTORY]/AncientModern.ind | cut -f1-2 -d " " > SampleInfo.txt
cut -f 1 -d ":" [YOUR DATA DIRECTORY]/AncientModern.ind | tr -d " " > PopNames.txt
paste SampleInfo.txt PopNames.txt | tr -s " " "\t" > AncientModern.ind
```
## Outgroup F3 statistics
Let us now try to run some outgroup F3 statistics for the ancient populations you
encountered yesterday. For this, we need to use the program qp3Pop from the package
[AdmixTools](https://github.com/DReichLab/AdmixTools). This program requires 4 input files
1. Genotype file - AncientModern.geno
2. SNP file - AncientModern.snp
3. Individual file - AncientModern.ind
4. Poplist file - List of population combinations for which to compute F3.

We are going to create one population list file for each ancient population.
This file contains population triplets, with the first column being the population we want to test, the second column being all the popuations in the dataset, and finally the third column being the outgroup.

Let us first look at the European LNBA population. I have already created such a file for this population, and it is located at `/science/groupdirs-nfs/SCIENCE-SNM-Archaeo/2019/Day4/fstats`. The file for `Europe_LNBA` is called Europe_LNBA.poplist.  Copy this file to the
directory you are working in.
```
cp /science/groupdirs-nfs/SCIENCE-SNM-Archaeo/2019/Day4/fstats/Europe_LNBA.poplist .
```
Examine the contents of the file.

The last thing we need before we run the F3 stats is the parameter file for qp3Pop.
You can copy the file from the same location as above and it is called Europe_LNBA.f3.par.
Here are its contents.
```
genotypename:   [YOUR DATA DIRECTORY]/AncientModern.geno
snpname:        [YOUR DATA DIRECTORY]/AncientModern.snp
indivname:      [YOUR DATA DIRECTORY]/AncientModern.ind
popfilename:    [YOUR DATA DIRECTORY]/Europe_LNBA.poplist
```
Now what do the results show? Do they make sense in terms of the populations?
What conclusions can you draw from these results about the affinity of the European LNBA population?

Now, try doing the same thing with the Steppe EMBA population.

## Admixture F3 statistics
In the previous section, we used the outgroup F3 statistic, where we knew that the Ju'hoan were not admixed with any of the test populations. Now we are going to use the F3 statistic, to figure out if there in any admixture between the target population (column 3 of the poplist file) and the source populations (columns 1 and 2).

In this case, we are going to use the French as the population we want to test for admixture. Let us create a poplist file to test this. We are going to use a bash loop to do this.
```
for pop1 in $(sort [YOUR DATA DIRECTORY]/PopNames.txt| uniq | grep -v French); do
  for pop2 in $(sort [YOUR DATA DIRECTORY]/PopNames.txt| uniq | grep -v French); do
    if [[ $pop1 > $pop2 ]]; then
      continue
    fi
    if [[ $pop1 != $pop2 ]]; then
      echo "$pop1 $pop2 French"
    fi
  done
done > Europe_LNBA.admixF3.poplist
```