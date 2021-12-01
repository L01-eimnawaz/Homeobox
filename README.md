# final-project-L01-eimnawaz

# 1. Lab 5: Finding Homologs with Blast

> First, we begin by starting the insatance and enetring into the directory we desire. In lab 5, this waas done in the cloned lab 5 directory. Here, it will reflect me putting in all of my files in my repository for the final project.

    chmod 0400 Downloads/MannuTay13.pem

    ssh -i Downloads/MannuTay13.pem ec2-user@44.196.160.255

    cd ~/labs

    git clone https://github.com/Bio312/final-project-L01-eimnawaz

    cd final-project-L01-eimnawaz

    cd ~/labs/final-project-L01-eimnawaz


> The steps taken in lab 5 could not be recreated on my own weeks later because the protesome folder needed to set up teh blast database could not be found when I went back to redo the lab. As a result, I simply copied and pasted the commands from when the lab was first done and brought the files over too. The next few labs will be doing the same.

    gunzip proteomes/*.gz

    cat  proteomes/* > allprotein.fas

    makeblastdb -in allprotein.fas -parse_seqids -dbtype prot
    
> The previous three steps consisted of the commands needed to set up the BLAST database from the compressed proteosomes given to us. These proteosomes were compressed and concatenated, which then allowed them to be formed into a database from which we could run our query sequences to identify homologs. 

    ncbi-acc-download -F fasta -m protein XP_001634249.2

    blastp -db allprotein.fas -query XP_001634249.2.fa -outfmt 0 -max_hsps 1 -out homeobox.blastp.typical.out
    
    less homeobox.blastp.typical.out
    
> I downloaded the homeobox protein Hox-A6 (Nematostella vectensis) and performed a blast search on that query protein, looking at the output with the command less.

    blastp -db allprotein.fas -query XP_001634249.2.fa -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle" -max_hsps 1 -out homeobox.blastp.detail.out
    
> The blast seaarch we performed was the standard, but we then wanted to perfromed a more detailed search with the flag -outfmt.
   
    awk '{if ($6<0.000000000000000000001)print $1 }' homeobox.blastp.detail.out > homeobox.blastp.detail.filtered.out

    wc -l homeobox.blastp.detail.filtered.out
    
> With the more detailed search performed, took those putative homologs and filtered out the results even further. To do this, even filtered out for evalues less than 0.000000000000000000001. When we counted the results, we ended up with 51 proteins.
    

    seqkit grep --pattern-file homeobox.blastp.detail.filtered.out allprotein.fas > homeobox.blastp.detail.filtered.fas

    muscle -in homeobox.blastp.detail.filtered.fas -out homeobox.blastp.detail.filtered.aligned.fas

    t_coffee -other_pg seq_reformat -in homeobox.blastp.detail.filtered.aligned.fas -output sim
    
> The last few steps consist of alligning the gene family sequence, as well as performing a global multiple sequence alignment in muscle, later using t_coffee to find that the average percent identity between the query sequence and all other sequences was 30.2%.


# 2. Lab 6: Phylogeny using IQ-TREE

>We begin Lab 6 by starting our instance and going into the lab 6 directory, similar to what we did for Lab 5. We then use the next few commands to install Newick Utilities into github, which we will be using trhoughout the lab.

        cd ~/tools

        git clone git://github.com/tjunier/newick_utils.git
    cd newick_utils/

        autoreconf -fi

        ./configure

        make

        sudo make install

         nw_display -h

>Next, we make a phylogenetic tree from the last lab's sequencing data using IQ-TREE

    cp ../lab5-L01-eimnawaz/homeobox.blastp.detail.filtered.aligned.fas .
    
> The sed command was used to remove any spaces within annonations in our sequences. This was done beacsue those spaces, if left alone, would have been lost in teh next few steps. However, they are still important, which is why this command repalces the spaces with underscores.

    sed "s/ /_/g" homeobox.blastp.detail.filtered.aligned.fas > homeobox.blastp.detail.filtered.aligned_.fas
    
> The next command, containing iqtree, was used to find the maximum liklihood tree estimate. I used the command less to view the output, becasue althhough it wasn't necessary, it helped to see the tree as it was during that step. It was here that I saw that the best-fit model according to BIC was JTT+F+R5

    iqtree -s homeobox.blastp.detail.filtered.aligned_.fas -nt 2

    less homeobox.blastp.detail.filtered.aligned_.fas.iqtree
    
> The tree that was obtained was just the standard. However, in order to better view the data we generated, we will need to root the tree. The type of rooting I choose was midpoiny, which was done with the command using gotree and drawn with the command after it.

    gotree reroot midpoint -i homeobox.blastp.detail.filtered.aligned_.fas.treefile -o homeobox.blastp.detail.filtered.aligned_.fas.midpoint.treefile

    nw_display -s  homeobox.blastp.detail.filtered.aligned_.fas.midpoint.treefile -w 1000 -b 'opacity:0' >  homeobox.blastp.detail.filtered.aligned_.fas.midpoint.treefile.svg
    
> The midpoint-rooted tree obtained, I pushed it into my github repository.

    git add homeobox.blastp.detail.filtered.aligned_.fas.midpoint.treefile.svg

# 3. Lab 7: Reconciling a Gene and Species Tree

>Lab 7 begins with starting our instance and going into the correct directory, as always. After that, we instal Numpy, which is needed to run Python for today's lab, and cargo, which is a rust pack manager.

    sudo yum install numpy
    
    sudo easy_install -U ete3

    curl https://sh.rustup.rs -sSf | sh
    
    1
    
> With that done, we restart the shell and begin to install thridkind, which is a program that generates graphic tree reconciliations.

    bash

    argo install thirdkind

    thirdkind
    
> We first copy the gene tree we generated in lab 6 for our gene family and bring it into this lab's repository.

    cp ~/labs/lab6-L01-eimnawaz/homeobox.blastp.detail.filtered.aligned_.fas .
    
> We use the follwoing command to perform the reconciliation via Notung. 

    java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s speciesTreeBilateriaCnidaria.tre -g homeobox --reconcile --speciestag prefix --savepng --events
    
>Python was then used to generate a RecPhyloXML object, which we used to view the gene-within-species tree.

    python2.7 ~/tools/recPhyloXML/python/NOTUNGtoRecPhyloXML.py -g homeobox.blastp.detail.filtered.aligned_.fas.midpoint.treefile.reconciled --include.species

> Thirdkind was then used to create a gene-reconciliation-with species tree reconciliation.

    thirdkind -f homeobox.blastp.detail.filtered.aligned_.fas.midpoint.treefile.reconciled.xml -o  homeobox.blastp.detail.filtered.aligned_.fas.midpoint.treefile.genes.tre.reconciled.svg

> For this segemnt we re-ran IQ-TREE, this time adding ultrafast bootstrap support to the optimal tree.

    iqtree -s homeobox.blastp.detail.filtered.aligned_.fas -bb 1000 -nt 2 -m JTT+F+R5 -t homeobox.genes.tre -pre homeobox.genes.ufboot

    gotree reroot midpoint -i homeobox.genes.ufboot.treefile -o homeobox.genes.midpoint.ufboot.treefile
    
> Lastly, we re-rooted the tree to create a midpoint-rooted tree, similar to what we did in lab 6. All these files were then pushed into the github repository.   

# 4. Lab 8: Domain Prediction using Interproscan

> We begin this lab by intalling the DataMash utility and running the following commands. After that is done, we can start our instance and go to our Lab 8 directory.

    cd ~/tools
    wget http://ftp.gnu.org/gnu/datamash/datamash-1.3.tar.gz
    tar -xzf datamash-1.3.tar.gz  
    cd datamash-1.3
    ./configure
    make
    make check
    sudo make install 
    
> We first create a directory in which all of our generated files will go to.    

    mkdir ~/labs/lab8-L01-eimnawaz/myfamily
    
> Then we run change directory and run iprscan5, which will send our sequences to the inteproscan servers to be analyzed.

    cd ~/labs/lab8-L01-eimnawaz/myfamily
    iprscan5   --email eiman.nawaz@stonybrook.edu  --multifasta --useSeqId --sequence   homeobox.blastp.detail.filtered.fas

> The cat command was used to to gather the output of iprscan, which were the domains identified for each protein, and concatenate them all into a single file. Grep was then used to filter through the domains, keeping only those defined by the Pfam database.

    cat ~/labs/lab8-L01-eimnawaz/myfamily/*.tsv.tsv > ~/labs/lab8-L01-eimnawaz/homeobox.domains.all.tsv

    grep Pfam ~/labs/lab8-L01-eimnawaz/homeobox.domains.all.tsv >  ~/labs/lab8-L01-eimnawaz/homeobox.domains.pfam.tsv
    
> We used the following command to change the output we received, re-arranging the interproscan output.  

    awk 'BEGIN{FS="\t"} {print $1"\t"$3"\t"$7"@"$8"@"$5}' ~/labs/lab8-L01-eimnawaz/homeobox.domains.pfam.tsv | datamash -sW --group=1,2 collapse 3 | sed 's/,/\t/g' | sed 's/@/,/g' > ~/labs/lab8-L01-eimnawaz/homeobox.domains.pfam.evol.tsv

> The final result was then pushed into our github repository, concluding tthe data-gathering portion of writing this paper

