[//]: ==================================
# Manual of SSimp
[//]: ==================================

This command line software enables summary statistics imputation (SSimp) for GWAS summary statistics. 
	
## Minimal example
[//]: -------------------------------
The minimal requirements are: (1) GWAS summary statistics file with at least a SNP id, Z, reference alleles and risk alleles, and (2) the path to the reference panel. 

`../bin/ssimp --gwas ../data/gwas1.txt --ref ../ref/small.vcf.sample.vcf` will generate a `gwas1.imp.log` file and a `gwas1.imp.out` file.
	

## Arguments
[//]: -------------------------------
Arguments can be shortend, e.g. `--wind` instead of `--window.width`

`--gwas [no default]`, path to GWAS dataset, extension does not matter, nor does text separation.  Columns should be named. Missings have to be marked as `NA`. quicktest, snptest, metal and plink output files will be automatically recognised. Minimal columns are SNP-id, Z, reference allele and risk allele. More info see below.

`--ref [no default]` path to vcf file (same folder should contain the tbi file)

`--out [gwasfilename]` string, if not define will be the gwasfilename (without extension ) + .imp

`--log [gwasfilename]` string, if not define will be the gwasfilename (without extension ) + .log, if set to FALSE, then no log file is stored

`--impute.range [no default]` should have the form of `chr:pos.start-chr:pos.end`, if just chr:NA, then the whole chr is imputed

`--impute.snp [no default]` strings

`--impute.snps [NULL]` txt file (e.g. snps2impute.txt) with a set of SNPs to impute (no header): rsid or chr:pos.hg19 (no quotes)

`--lambda [2/sqrt(n)]` numeric value or string (`2/sqrt(n)`, `optimize`), n are the number of individuals in the reference panel. `optimize` not yet implemented.

`--impute.maf [0]` numeric value, lower limit for variants to be imputed: everything above and equal this threshold will be imputed

`--tag.maf [0]` numeric value, lower limit for tag SNPs: everything above and equal this threshold will be used as tag SNPs. 

`--window.width [1e6]` numeric value, core window length

`--flanking.width [250e3]` numeric value, flanking space left and right side of the core window
		
`--missingness [TRUE]` enable variable sample size approach. This is automatically set to `FALSE` if `N` is not provided or `N` is set to `NA`.

### Note	
[//]: -------
- If `impute.range` and `impute.snps` are not defined, then all variants in the refpanel are imputed that are not provided as tag SNPs.
- If the GWAS positions do not match the reference panel positionn, use use LiftOver as a command line tool: http://genome.ucsc.edu/cgi-bin/hgLiftOver
- The option `missingness` is automatically set to FALSE if `N` is not provided or `N` is set to `NA`.
- The minimal columns required are `SNP`, `A1`, `A2`, `Z`. If `Z` is not present, but `P` and `b` are, `Z` is calculated through `P` and `b`. Alternatively, if `b` and `SE` are present, then it is also possible to calculate `Z` via `b` and `SE`. 
- Odds ratios need to be provided as Z-statistics or log-transformed into effect sizes.

## Format of GWAS dataset
[//]: -------------------------------
Column names are automatically recognized using commonly used names. The `log` file will indicate which columns are used and recognized. Positions should match the positions in the reference panel (e.g. both hg19). It is recommended to provide the sample size (N), as incorporating missingness leads to a more accurate estimate. 

## Reference panel
[//]: -------------------------------


## Imputation
[//]: -------------------------------
For more details on the summary statistics imputation method, please see our-paper (2017). 

`insert here a brief recap of the method`

- If SNP-ID are present in the GWAS, then pos copied from reference panel. 
- If SNP-ID are not present, then the combination of Chr:Pos:A1:A2 are taken as identifier
- Because either SNP-ID or Chr:Pos:A1:A2 are used as identifier, it is also possible to impute indels.
- Z statistics are imputed, along with `N.imp` (an estimate for the sample size) and `r2.pred` (adjusted imputation quality).
- To speed up computation, we use a sliding window approach (`--window.width` and `--flanking.width`). SNPs to be imputed are assigned to one window. The window number is reported in the `.imp.out` file. 

 
## Output
[//]: -------------------------------

### log file
[//]: -------
The `.imp.log` file provides a summary of the imputation done and the sanity check results. 

### out file
[//]: -------
The `.imp.out` file has the following columns:

- `SNP` SNP-ID
- `Chr` Chromosome (only 1 to 22 right now)
- `Pos` Position (hg19, or same as in reference panel)
- `Z.imp` Imputed Z statistics
- `N.imp` Estimation of N (only if missingness was set to `TRUE`)
- `r2.pred` Imputation quality (adjusted as in ...)
- `A1` Reference allele
- `A2` Effect allele
- `window.nbr` in which window imputed
- `lambda` lambda used to impute

if `Z.imp NA` and `r2.pred 0` means that there was not tag SNP.