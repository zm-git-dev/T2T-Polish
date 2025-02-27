# Merqury

Merqury was run three times using `Illumina` `HiFi` and `hybrid` 21-mer databases during evaluation. We recommend using the `hybrid` k-mer db for evaluating CHM13 assemblies.

## Dependencies
* [Meryl v1.3](https://github.com/marbl/meryl)
* [Merqury](https://github.com/marbl/merqury)

Pre-built CHM13 databases are downloadable from here:
* [IlluminaPCRfree.k21.meryl](https://s3-us-west-2.amazonaws.com/human-pangenomics/T2T/CHM13/assemblies/qc/IlluminaPCRfree.k21.meryl.tar.gz): 21-mers from Illumina PCR-free library
* [hifi20k.k21.meryl](https://s3-us-west-2.amazonaws.com/human-pangenomics/T2T/CHM13/assemblies/qc/hifi20kb.k21.meryl.tar.gz): 21-mers from HiFi 20 kb library
* [hybrid.k21.meryl](https://s3-us-west-2.amazonaws.com/human-pangenomics/T2T/CHM13/assemblies/qc/hybrid.k21.meryl.tar.gz): 21-mers from hybrid Illumina and HiFi data

Extract with `tar -xzf`, download Merqury and Meryl. No installation is required to run Merqury and a binary release is available for [Meryl](https://github.com/marbl/meryl).

## Quick start

```
$tools/merqury/merqury.sh $read.meryl asm.fasta out-prefix
```
`asm_only.bed` file contains k-mers in the assembly not found in the given `$read.meryl`.

Use bedtools to merge and provide for obtaining [low coverage associated with consensus base error](https://github.com/arangrhie/T2T-Polish/tree/master/coverage#prerequisites-1).

## Generating k-mer databases

### 1. Illumina and/or HiFi

In general, any k-mer databases can be obtained with
```
meryl count k=21 reads.fastq output reads.meryl
```

### 2. Hybrid

While evaluating T2T-CHM13v0.9, we noticed sequencing biases affecting k-mers when estimating base accuracy (QV). Therefore, we built a hybrid database to exclude low frequency erroneous k-mers and include reliable k-mers found either in Illumina or HiFi. For more details, refer to [McCartney et al, 2021](https://doi.org/10.1101/2021.07.02.450803).

The hybrid k-mer database was generated by first excluding k-mers occurring only once in each database:
```
meryl greater-than 1 IlluminaPCRfree.k21.meryl output illm.gt1.meryl
meryl greater-than 1 hifi20k.k21.meryl output hifi.gt1.meryl
```

Next, matching the diploid (2-copy) peak to 35x:
```
meryl divide-round 3 illm.gt1.meryl output illm.gt1.div3.meryl
meryl increase 4 hifi.gt1.meryl output hifi.gt1.add4.meryl
```

Finally, union the two dbs and set the frequency to the maximum observed in the two datatypes:
```
meryl union-max illm.gt1.div3.meryl hifi.gt1.add4.meryl output hybrid.meryl
```
