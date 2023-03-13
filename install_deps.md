### Set up directories

```
mkdir gene_test && cd gene_test
mkdir deps && cd deps
```


### Install Kent's source tree


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

mkdir -p $HOME/cpanm
export PERL5LIB=$PERL5LIB:$HOME/cpanm/lib/perl5
cpanm -l $HOME/cpanm Bio::DB::BigFile
```

### Install cpanm to install bio big wig



### Download loftee 


### Move dependencies into the correct folder 

### Convert bgen to vcf with no samples


```
module load bcftools
module load htslib

qctool=
bgen=
bcftools=


${qctool} \
       -g ${bgen} \
       -og ../data/${outname}.vcf.gz \
       -s ${sample} \
       -excl-samples 23_gsid_keep1.sample \
       -force

zcat ../data/${outname}.vcf.gz | bgzip -c > ../data/${outname}.bgz
tabix ../data/${outname}.bgz
/well/ckb/users/aey472/program_files/bcftools/bcftools view -G ../data/${outname}.bgz -Ob > ../data/${outname}_nosamples.bgz
```

### Run VEP



