## SNP analysis workflow *(RiceVarMap version)*
#### Part1. Prepare the coordination & search for SNPs
*Some important notes in this process:*

* Choose the version of coordination according to your database, in this case, RiceVarMap adopts MSU6.1(=MSU6.0), so we have to switch the genome version to MSU6.1 rather than MSU7.0
* For some special miRNA precursors (such as osa-MIR444 family & osa-MIR810a), their genome intervals are divided into 2 parts, search each part separately

**Steps & Scripts involved:**

* ```Search_within_region.py``` to search for SNPs within precursor intervals
	* Params: a coordination file with columns as *chr_id, start, end, miRNA name*
	* Returns: a list, including *SNP name, SNP position, reference allele, major allele, minor allele & major allele freq*
* Search for SNPs within precursors and mature regions separately for convenience and store them in your excel file

***
#### Part2. miRNA Haplotype* analysis
**Some terms to be illustrated**

* **miRNA haplotype**
	* Adopt SNP as biological marker, for each miRNA precursor, SNPs distributed within its genome region form the miRNA haplotype (in ascending order of genome coordination)
	* e.g. osa-MIR443's miRNA haplotype: sf0330014542, sf0330014549, sf033001458, sf0330014600
* **haplotype pattern**
	* For each miRNA precursor, every locus of SNP is occupied with a nucleotide acid, so haplotype pattern means a specific sequence of nucleotide; and because every SNP possess 2 alleles(commonly, but not always), theoretically there are ```2^len(miRNA haplotype)```haplotype patterns for each haplotype
	* e.g. one haplotype pattern of osa-MIR443: CGGA
	* Special haplotype patterns:
		* Reference pattern: all loci are possessed by allele in reference genome
		* Non-reference pattern: all loci are possessed by allele different from the on in reference genome
* **trinary pattern**
	* This is a newly coined term, in which reference allele is replaced by 0, non-reference allele is replaced by 1 and 'N' is replaced by 2(Note that because the sequencing of rice genome got a miss-calling at the specific SNP position, an 'N' will occur)
	* e.g. reference pattern of osa-MIR443: CGGA <===> 0000; while AATT <===> 1111

**Steps of analysis:**

* step1: Classify SNPs into their corresponding precursor intervals in ascending order **(This is the so-called miRNA haplotype)**
* step2: Obtain reference pattern and non-reference pattern of each miRNAs (as reference)
* step3: For each precursor along with its haplotype, grasp the haplotype pattern* and the corresponding cultivars
* step4: Transform the haplotype pattern into trinary pattern* (To compare each haplotype pattern visually with 0-1-2 digits)
* step5: For each haplotype pattern, mutate the original RNA sequence with specific SNPs

**Scripts involved:**

* ```snp_signature.py```
	* params: reads in a table containing SNPs within miRNAs, table headers are miRNA name(1), snp_id(2), ref_allele(3), nonRef_allele(4)
	* returns:
		* -p: prints out reference pattern and non-reference pattern of each miRNA precursor
		* -s: prints out the haplotype of each miRNA precursor
* ```Genotype_with_SNPid.py```
	* params: input miRNA name along with its haplotype
	* return: grasp CSV file of SNPs against cultivars and classify the haplotype patterns of each precursor
	* work flow:
		* crawl the website of RiceVarMap and grasp the CSV file
		* for each haplotype, classify its patterns and corresponding cultivars
* ```Trinary_pattern.py```
	* params: Input file format: First column is reference pattern, second column is haplotype
	* return: the trinary pattern in order
* ```mutate_RNA_seq.py```
	* params (input file)
		* miRinfo file columns are: "precursor, sequence, strand +/-, chr_id, start, end"
		* hap_tab file columns are: "precursor, trinary pattern"
		* snp_info file columns are: "SNP ID, chr_id, position, Ref_allele, NonRef_allele"
		* haplotype file: "miRNA: SNP1, SNP2, SNP3..."
	* return: a table containing miRNA name, haplotype, original RNA seq, mutated RNA seq

*Some notes in this part*

* For those precursors, which contains introns, they shall be specially treated when mutate their RNA sequence
* Pay special attention to the order of SNPs within each precursor, which will largely affect the result of the downstream steps (scripts shall take this into account, including ```snp_signature.py & Genotype_with_SNPid.py```)
	
***
