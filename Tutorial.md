#	MGC Intro to Genomics Course

In this course you will demonstrating some of the basic concepts of genome analysis. The requirements for this course are as follows:  

1. **Ubuntu** Ubuntu is a Linux based operating system like Windows or MacOS. Running Ubuntu within a virtual machine is like having a computer on your computer. We will be using Ubuntu for all of our analysis. [Here](https://learn.microsoft.com/en-us/windows/wsl/install) is a guide for installing a subsystem for Windows. If you are on a MacOS you will need a different way to install an Ubuntu subsystem. 
2. **A [GitHub account](https://github.com/)** GitHub is a cloud file storage service like Google Docs or OneDrive. By uploading files to GitHub we can access them from anywhere and even track changes to files. Each database is known as a repository. 

## The Command Line Interface and Installing Software

Unlike a computer you may be used to, our Ubuntu subsystem uses command line to execute functions. Instead of clicking buttons on a screen to issue commands, you will need to manually type them in. These commands are given using Bash, a command line language similar to R and Python. Before you start entering commands we need to enable copy and paste for the virtual machine. To do this go to the machine's Settings > Advanced > and use the dropdown menus for Shared Clipboard and Drag and Drop to bidirectional.

Next you will need to open the terminal. Click the circle in the bottom left corner of the screen to show apps and click Terminal.  


First we will need to install the program git, this will be necessary for accessing the files used in this tutorial. 

To install git use the following command

```
sudo apt install git
```

Next you will need to sign into GitHub.

```
git config --global user.name Yourgithub_username
git config --global user.email Youremail@example.com
```

Now that we have all of our software lets go over some basic Bash commands. First you are going to need this repository for this workshop. 

```
git clone https://github.com/alcarr785/MGC_Genotyping_Workshop
cd MGC_Genotyping
```
where the command `clone` copies the entire repository for you to use. Next, `cd` is used to change directory into the repository. If we want to go up a directory we can use the command `cd ..`. Next we want to view the files in here.

The `ls` command prints all the files and directories in this directory. 
```
ls
```

We have 3 files here named Oys.ped Oys.map Failed.txt file. We will go over what the .ped and .map extensions are in the next section but for now lets see what's in the FailedQC.txt file.

```
cat FailedQC.txt
```

The 'cat' command allows us to read the contents of files. This file contains some of the samples that were marked as failing qc by Axiom with family in the first column and individual in the second, separated by a tab.   
*Note: This file will not be included in the data returned by CAT, you will have to manually make such a file using the report from CAT of failed samples. One way of doing this is by using [nano](https://linuxize.com/post/how-to-use-nano-text-editor/).*

There are many commands in Bash, some of which you will learn through this course. A cheat sheet for some of the more important ones can be found [here](https://github.com/RehanSaeed/Bash-Cheat-Sheet). The number of commands may seem overwhelming at first but as with all things you will get better at it through practice. 

All of our data analysis will be done in PLINK, a program that can be used for manipulation and analysis of genomic data.

Unlike git, PLINK cannot be installed directly from the command line. Instead it will have to be downloaded directly from the website using the `wget` command.

```
wget https://s3.amazonaws.com/plink1-assets/plink_linux_x86_64_20231211.zip
```

To unzip it we need to first install a program that will unzip PLINK and then unzip it into a new folder.

```
sudo apt install unzip
sudo unzip plink_linux_x86_64_20231211.zip -d plink
```

Now we have to make PLINK accessible in command line, this will allow us to run the program in the virtual machine regardless of what folder we are in. 

```
cd plink
sudo cp plink /usr/local/bin
sudo chmod 755 /usr/local/bin/plink
```

To test if it works we first need to get out of the plink folder and type plink into the command line.
```
cd ..
plink
```

## PLINK
CAT will give a .ped file and a .map file both of which are necessary for running PLINK. They will also provide a few other files, but they will not be necessary for this workshop. 

### PLINK Files
The .ped file contains one row for every sample genotyped. The first two columns are the family/population and individual respectively. The next four are related to parent ids, sex and phenotype but are not relevant for this workshop. After this there are 2 columns for each SNP in the dataset, one for each allele. The .map file has one row for each SNP with the Chromosome, SNP identifier, position in centimorgans (not relevant) and position on the chromosome for each SNP. 

For example if we had genotyped 30 individuals at 500 SNPs, we would have a .ped file with 1006 columns and 30 rows and a .map file with 500 rows. In this dataset we have 45 individuals sequenced at 65893 SNPs. We can verify this by counting the number of rows in each file using the following commands.

```
wc -l Oys.ped
wc -l Oys.map
```

Here 'wc' is a command that returns the word count in a file while '-l' is a modifier that returns the number of lines in a file. 

### The PLINK syntax
Now that you understand what PLINK files are, it is time to learn how to actually use the software. Because we are working in a command line interface, we have to manually type all of our commands into the machine. Here is an example of a PLINK call:
```
plink --file Raw --recode --out RawNew
``` 
In this example `plink` calls the program, think of this like double clicking an application to open it. Next the flag `--file` tells PLINK which filename to open, do not enter the file extension here, it will automatically look for a .ped and .map file. Next, `--recode` tells the program to create a new file after all other operations are done and `--out` tells the program what to name this new file. By default `--recode` creates a .ped and .map file. Each flag (`--function`) executes a different function. A comprehensive list of functions and how to use them can be found [here](https://zzz.bwh.harvard.edu/plink/data.shtml) and we will be using some of them shortly. 

### Filtering
While genotyping is a powerful and useful technology, sequencing errors and non-useful SNPs can affect the results of downstream analysis that need to be taken care of. 

**Note: Before filtering it is important to consider what type of questions you will be asking and how different filtering methods will affect the answers to your questions. You may not need to use all these filters, and you may even have to rely on ones not covered here. It all depends on your research project.**

To keep everything organized, lets make a folder for filtering, this can be done using `mkdir`

```
mkdir Filtering
cd Filtering
```

The first step in filtering will be to remove the individuals identified by CAT as not sequencing properly, these can be found in the spreadsheet provided by CAT.

```
plink --file ../Oys --chr-set 94 --remove ../FailedQC.txt --chr 1-10 --recode --out Oys.1.qc
```

The `--chr-set 94` flag is used to handle an error with PLINK not recognizing the mitochondrial chromosome labeled 94. Originally PLINK was made for human data and gets confused by this. Next `--remove FailedQC.txt` takes a file with a of individuals to remove, in this case FailedQC.txt. Finally, `--chr 1-10` is used so that we are only using chromosomal DNA not mitochondrial DNA on chromosome 94. 
 
Next we will remove SNPs that are missing in more than 5% of individuals and individuals that are missing more than 5% of SNPs. This is done with the `--geno` and `--mind` flags respectively. This step is important because this missing information can lead to unreliable results. The threshold values for missingness can vary based on the species and the study you are doing. 

```
plink --file Oys.1.qc --geno 0.05 --mind 0.05 --recode --out Oys.2.miss
```


We also want to remove SNPs with a minor allele frequency less than 0.05 as these are often the result of a sequencing error using the `--maf` flag. It is possible to do all of the previous filtering parameters in one step, they are simply put in different steps for the purpose of this demonstration.

```
plink --file Oys.2.miss --maf 0.05 --recode --out Oys.3.maf
```

The final step in filtering is a bit more complicated, we will be removing SNPs that are linked. [Linkage disequilibrium](https://www.nature.com/articles/nrg2361) is the result of two or more loci being associated with each other, often because of proximity to each other on the genome which can affect the results of studies that assume alleles are independent of each other.  
To address this we need to prune our dataset, meaning that if two SNPs are linked, one of them will be removed. Unlike the other commands this needs two separate commands to work the first is `--indep-pairwise` which takes three arguments: the size of the section of a genome it should count, how much to shift the window over after each count and the r<sup>2</sup> threshold which measures how correlated two SNPs are. The parameters we will be using are 50,5 and 0.5 respectively. This flag tells PLINK to produce two files, a `.prune.in` file and a `.prune.out` which contain a list of SNPs that aren't linked and a list of SNPs that are linked respectively. 

```
plink --file Oys.3.maf --indep-pairwise 50 5 0.5 --recode --out Oys
```

Because we want independent SNPs we are only keeping those that aren't linked. This can be done using the `--extract` function and the `prune.in` file created from the function above. 

```
plink --file Oys.3.maf --extract Oys.prune.in --recode --out Oys.final
```

Now we have a filtered dataset that we can work with. While we can keep it in a PLINK format and use other PLINK functions to analyze the data, we could also convert it to a different file format that can be read by other programs. Here we are going to create a vcf file using `--recode`. 

```
plink --file Oys.final --recode vcf --out Oys.final
```


## Using Git

Congrats you now have  beautiful and fully filtered PLINK and vcf files. Only one small problem, they are only on your virtual machine. If only there was some sort of hub where you could git your data from. Oh wait!

Git is a great way of accessing data from different devices, sharing data with colleagues, tracking changes to documents or making reproducible workshops for your lab mates to follow. 

On your web browser sign into [GitHub](github.com).
Next find the green button on the left hand side of the webpage that says New and click create a new repository, then follow the steps on the next page and click create repository. You will also need to create a [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens), 

Now back in the virtual machine you are going to clone this new repository. To to this copy the link to it and enter it into the `git clone` command. 

```
git clone (Link to your repository)
cd REPOSITORY
``` 


Now you can move the final vcf file to the git repository using the `cp` command. As you may have guessed `cp` copies files and takes the argument of the file to copy and the location to copy it to. You may also notice that I put an asterisk at the end of the file. This asterisk tells bash to target any files that match this string. In this case it is filenames that start with "Oys.final.".

```
cp ../MGC_Genotyping_Workshop/Filtering/Oys.final.* .
```

Now we have all of our final documents in our new repository. Next we need to upload it. This requires three commands. The first one `git add *` creates a queue of files to upload. Next `git commit` will be for recording files and a message, this is just a way of tracking what changes you made to the repository for now put something general like "Uploaded files". Finally `git push origin main` will actually upload the files. 

```
git add *
git commit
git push origin main 
```

Next it will ask you for your username and password. For password you will need to paste your personal access token generated earlier. 

Now if you go online you should see all your files. From here you can download them to your desktop, but there is a [Git Desktop](https://desktop.github.com/download/) app which is essentially Git but with a user interface. 

I hope this was a great way for you to get started with bioinformatics. If you need anymore resources or want to expand your knowledge the internet is a great resource, here are just a few websites and creators that helped me.

* [Genomics Boot Camp](https://www.youtube.com/watch?v=FnKizCPz85U&list=PLdf-U83sN48P30mGSpudOsi9CyzRYCO21): A playlist of some youtube videos about introductory genomic analysis
* [Speciation Genomics](https://speciationgenomics.github.io/): A guide to analyzing genomic data in VCF format.
* [NACE 2024 Oyster Genomics Workshop](https://marineevoecolab.github.io/NACE_MAS_Genomics_Workshop/): Another how-to guide to genomic analysis using PLINK and R. 
