### Initial setup

* Make sure you have an update to date gcc compiler in your path 


### Set up directories

```
mkdir gene_test && cd gene_test
base_dir=${PWD}
mkdir bin
mkdir deps && cd deps
deps=${PWD}
```


### Install Kent's source tree (required for Bio::DB::BigFile installation)


```
## first might need to load (my)SQL, openSSL 

wget https://github.com/ucscGenomeBrowser/kent/archive/v335_base.tar.gz
tar xzf v335_base.tar.gz

export KENT_SRC=$PWD/kent-335_base/src
export MACHTYPE=$(uname -m)
export CFLAGS="-fPIC"
export MYSQLINC=`mysql_config --include | sed -e 's/^-I//g'`
export MYSQLLIBS=`mysql_config --libs`

cd $KENT_SRC/lib
echo 'CFLAGS="-fPIC"' > ../inc/localEnvironment.mk

make clean && make
cd ../jkOwnLib
make clean && make

```

### Install Bio::DB::BigFile dependency

```
mkdir -p $HOME/cpanm
export PERL5LIB=$PERL5LIB:$HOME/cpanm/lib/perl5
cpan install Bio::DB::BigFile

## at this point in the installation, cpan should prompt you to paste in the link for Kent's source tree that we just installed. If it doesn't, something has probably gone wrong!
```


### Download loftee which annotates loss of function variants

```
cd ${deps}
git clone -b grch38 https://github.com/konradjk/loftee
```



### Download primateAI
```
cd ${deps}

# Download pre-computed scores from https://github.com/Illumina/PrimateAI (note you need to sign up for a free account for this)

module load htslib

mkdir primateAI && cd primateAI
gunzip -cf PrimateAI_scores_v0.2_hg38.tsv.gz | sed '12s/.*/#&/' | sed '/^$/d' | awk 'NR<12{print $0;next}{print $0 | "sort -k1,1 -k 2,2n -V"}' | bgzip > PrimateAI_scores_v0.2_GRCh38_sorted.tsv.bgz
tabix -s 1 -b 2 -e 2 PrimateAI_scores_v0.2_GRCh38_sorted.tsv.bgz
```


### Move dependencies into the correct folder 

### Convert bgen to vcf with no samples



```
module load bcftools
module load htslib

qctool=path_to_qctool
bgen=path_to_bgen_file

## this just gets the first sample from the bgen. 
## For some reason the VCF needs to have a single sample when you pass it to bcftools

${qctool} \
       -g chr22.bgen -os chr22.bgen.sample >
sed '1,2d' chr22.bgen.sample | head -n1 > keep_sample.txt


### convert the bgen to a single sample vcf
### split this by chr
${qctool} \
       -g ${bgen} \
       -og single_sample.vcf.gz \
       -s ${sample} \
       -excl-samples keep_sample.txt \
       -force

## bgzip the vcf
zcat single_sample.vcf.gz | bgzip -c > single_sample.vcf.bgz
tabix single_sample.vcf.bgz

## strip the samples from the vcf, we don't need these any more
bcftools view -G single_sample.vcf.bgz -Ob > no_sample.vcf.bgz
```

### Run VEP

```
loftee=/well/ckb/users/aey472/projects/vep_annotation/Plugins/loftee

export PERL5LIB="/well/ckb/users/aey472/projects/vep_annotation/Plugins/loftee":$PERL5LIB
export PERL5LIB="/users/ckb/aey472/cpanm/lib/perl5/x86_64-linux-thread-multi":$PERL5LIB
export PERL5LIB="/users/ckb/aey472/.vep/Plugins/cadd":$PERL5LIB
export PERL5LIB=$PERL5LIB:$HOME/cpanm/lib/perl5

module purge all
module load VEP/103.1-GCC-10.2.0
module load OpenSSL/1.1
module load SAMtools/1.12-GCC-10.2.0
module load HTSlib/1.11-GCC-10.2.0

vep \
        --input_file ${input_vcf} \
        --dir_cache /gpfs3/well/ckb/users/aey472/projects/vep_annotation/data/cache \
        --assembly GRCh38 \
        -o ../output/imputed_v2.0_b38_chr${SGE_TASK_ID}_annot.loftee.0Kb.vcf \
        --force_overwrite \
        --vcf \
        --everything \
        --offline \
        --distance 0 \
        --plugin LoF,loftee_path:${loftee}/,human_ancestor_fa:/well/ckb/users/aey472/projects/vep_annotation/data/human_ancestor.fa \
        --cache
```


