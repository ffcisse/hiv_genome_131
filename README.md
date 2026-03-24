# HIV Genome Browser

This is the READme to recreate the HIV genome browser, focusing on accessory protein sequences across HIV subtypes A-C, and recombinant form AE. This READMe assumes the user already has apache2 and jbrowse-cli installed

#  1. Download system dependencies
Install wget, samtools, and tabix.

  1. wget is a tool for retrieving files over widely-used Internet protocols like HTTP and FTP.
  2. samtools and tabix are tools for processing and indexing genome and genome annotation files.

Linux
```
sudo apt install wget apache2
brew install samtools htslib
```
# 2. Verify apache2 server folder
For a normal linux installation, the folder should be /var/www or /var/www/html
Verify that one of these folders exists. We will now add JBrowse2 to the folder.

Take note of what the folder is, and use the command below to store it as a command-line variable. We can reference this variable in the rest of our code, to save on typing. You will need to re-run the export if you restart your terminal session!

**Be sure to replace the path with your actual true path!**
```
export APACHE_ROOT='/path/to/rootdir'
```
For a Linux instance:
```
export APACHE_ROOT='/var/www/html'
```
NOTE: APACHE_ROOT variable will erase if you restart your terminal.

# 3. Download and process HIV Genomic data (FASTA files) from NCBI database:
1. HIV Reference genome: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/864/765/GCF_000864765.1_ViralProj15476/GCF_000864765.1_ViralProj15476_genomic.fna.gz
2. Subtype A: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/104/735/GCA_003104735.1_ASM310473v1/GCA_003104735.1_ASM310473v1_genomic.fna.gz
3. Subtype B: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/102/975/GCA_003102975.1_ASM310297v1/GCA_003102975.1_ASM310297v1_genomic.fna.gz
4. Subtype C: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/098/035/GCA_003098035.1_ASM309803v1/GCA_003098035.1_ASM309803v1_genomic.fna.gz
5. Combinant Form CRF01_AE: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/104/795/GCA_003104795.1_ASM310479v1/GCA_003104795.1_ASM310479v1_genomic.fna.gz
  
Download reference HIV genomes for each subtype. Replace the link with corresponding genome after wget command
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/864/765/GCF_000864765.1_ViralProj15476/GCF_000864765.1_ViralProj15476_genomic.fna.gz
```
Unzip the gzipped reference genome, rename it, and index it. This will allow jbrowse to rapidly access any part of the reference just by coordinate.
```
gunzip GCF_000864765.1_ViralProj15476_genomic.fna.gz
mv GCF_000864765.1_ViralProj15476_genomic.fna HIVref.fa
samtools faidx HIVref.fa
```
Load genome into jbrowse
```
jbrowse add-assembly HIVref.fa --out $APACHE_ROOT/jbrowse2 --load copy
```
# 4. Download and process annotation tracks for each genome (GFF files) from NCBI database:
## Add genome assemblies ##
1. HIV Reference genome: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/864/765/GCF_000864765.1_ViralProj15476/GCF_000864765.1_ViralProj15476_genomic.gff.gz
2. Subtype A: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/104/735/GCA_003104735.1_ASM310473v1/GCA_003104735.1_ASM310473v1_genomic.gff.gz
3. Subtype B: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/102/975/GCA_003102975.1_ASM310297v1/GCA_003102975.1_ASM310297v1_genomic.gff.gz
4. Subtype C: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/098/035/GCA_003098035.1_ASM309803v1/GCA_003098035.1_ASM309803v1_genomic.gff.gz
5. Combinant Form CRF01_AE: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/104/795/GCA_003104795.1_ASM310479v1/GCA_003104795.1_ASM310479v1_genomic.gff.gz

## Adding genome annotation tracks ##
Commands for uploading HIV reference genome annotation track using wget from URL. If you have a predownloaded annotation gff file, skip to next step.
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/864/765/GCF_000864765.1_ViralProj15476/GCF_000864765.1_ViralProj15476_genomic.gff.gz
gunzip GCF_000864765.1_ViralProj15476_genomic.gff.gz
```

Use jbrowse to sort the annotations: 
  1. jbrowse sort-gff sorts the GFF3 by refName (first column) and start position (fourth column), while preserving header lines at the top of the file (which start with “#”). 
  2. Compress the GFF with bgzip (block gzip, which zips files into little blocks for rapid access), and index with tabix.
  3. The tabix command outputs a file named HIVref.gff.gz.tbi in the same directory, and we then refer to “HIVref.gff.gz” as a “tabix indexed GFF3 file”.
  4. Change 'HIVref' title to HIV1_SubtypeA, B, C, CRF_AE each time you upload an annotation track

```
jbrowse sort-gff GCF_000864765.1_ViralProj15476_genomic.gff > HIV_ref.gff
bgzip HIV_ref.gff
tabix HIVref.gff.gz
```

Load annotation track into jbrowse.

```
jbrowse add-track HIVref.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames HIVref
```
Repeat this for all desired annotation tracks for each subtype, replacing the corresponding link, title.gff, and assemblyNames for different subtypes.

# 5. Configure Plugins
Configure the 3D protein viewer and MSA plugin by editing your config.json file. This file is typically found in your $APACHE_ROOT/jbrowse2 or /var/www/html/jbrowse2 folder:
1. Copy and paste into your config.json file above assemblies using vim or nano editors on terminal
```
"plugins": [
  {
    "name": "Protein3d",
    "url": "https://unpkg.com/jbrowse-plugin-protein3d/dist/jbrowse-plugin-protein3d.umd.production.min.js"
  },
  {
    "name": "MsaView",
    "url": "https://unpkg.com/jbrowse-plugin-msaview/dist/jbrowse-plugin-msaview.umd.production.min.js"
  }
  ]
```
Your config.json file should look something like this: <img width="914" alt="Screenshot 2024-12-09 at 12 28 35 PM" src="https://github.com/user-attachments/assets/7b4e195d-c798-48ab-ab22-b692a84465b9">

JBrowse has multiple other plugin configurations you can find here: https://jbrowse.org/jb2/plugin_store/

# 6. Navigating Multiple Sequence Alignment Viewer Plugin
## Genome Alignment ##
1. Download the multiple sequence alignment FASTA file to be uploaded to the Msaview plugin. Using the NCBI Virus Database,  look up the specific NCBI accession numbers for different HIV1 Subtypes. https://www.ncbi.nlm.nih.gov/labs/virus/vssi/#/virus?SeqType_s=Nucleotide&VirusLineage_ss=Human%20immunodeficiency%20virus%201,%20taxid:11676&utm_source=data-hub&ids=U51188%20AF067155%20K03455%20U54771%20NC_001802%20
* HIV1 RefSeq genome: NC_001802.1 
* Subtype A: U51188.1
* Subtype B: K03455.1
* Subtype C: AF067155.1
* Combinant Form CRF01_AE: U54771.1

2. Download these files 
* Click Align in the top right corner to generate an MSA FASTA file and download. 
* Click Build Phylogenetic Tree to generate a newick file and download.

In JBrowse:
1. Click 'Add' in top left corner
2. Click Multiple Sequence Alignment Viewer
3. Choose your MSA file and newick files to display

## Protein alignment ##
1. Go to https://www.hiv.lanl.gov/content/sequence/NEWALIGN/align.html
2. To view the MSA FASTA file for each protein (Env, Gag, Nef, Pol, Rev, Vif, Vpr, Vpu) select a protein in the drop-down section. 
3. Specify parameters accordingly to align protein sequences from subtype M and CRFs <img width="1391" alt="Screenshot 2024-12-09 at 3 59 29 PM" src="https://github.com/user-attachments/assets/a0c9d279-14cf-421b-ad5f-45538ab055d0">

4. Make sure that the format is in FASTA and the organism is HIV-1/SIVcpz, and then select "Get Alignment".
5. Download the file
6. Display file in MSA Viewer

# 7. Navigating 3D Protein Viewer Plugin

1. When annotation track is open, hover over a protein, and right click. Click "Launch protein view"
2. When Protein view is up, click open file manually. Click PDB ID bubble and input the following PDB ID's provided (one at a time). Click Launch 3-D Protein Structure View:
   * 8FYJ - Env BG505 SOSIP-HT2 bound to two CD4 
   * 6MEO - gp120 bound to CD4 and CCR5
   * 6HAK- Reverse Transcriptase bound to dsRNA
   * 6URI- Nef bound to CD4 and AP2
   * 7U0F- Rev bound to tubulin ring
   * 6CYT- Tat bound to AFF4, P-TEFb, and TAR loop
   * 8FVJ- Vif bound to APOBEC3H, CBF-beta, ELOB, ELOC, and CUL5 dimer
   * 6XQJ: Vpr bound to hHR23A (NMR)
   * 4P6Z: Vpu bound to BST2 and AP1
3. Click the wrench icon to open settings.
4. Under download structure tab, reinput the PDB ID you first input. Click apply, which should take you to the State Tree Page
5. Click the Assembly 1 tab, then apply action, and finally 3D representation. Click apply.
6. Repeat steps for other proteins.

# 7. Uploading new annotation track from another genome browser
Utilize HIV LANL Genome Browser https://www.hiv.lanl.gov/components/sequence/HIV/featuredb/search/feature_search.comp ](https://www.hiv.lanl.gov/content/sequence/jbrowse/?loc=HXB2%3A1..9719&tracks=DNA%2CHXB2_gene_map&data=hivdata%2FEpitope-nucleic&highlight= to implement custom annotation tracks.

1. Click on HXB2-sub-protein-map track
2. Hover over track and 'Save track data' as GFF format
   <img width="363" alt="Screenshot 2024-12-09 at 4 19 43 PM" src="https://github.com/user-attachments/assets/a7364f91-0719-4288-b8e0-8c153fc99748">

3. Ensure seqIDs in gff file matches the headers for the HXB2_SubtypeB FASTA uploaded earlier in assemblies. If it doesn't match, modify GFF file to replace seqID column with correct FASTA header for the genome: K03455.1
4. Use cat command to view what the headers currently are.
```
  cat /path/to/HXB2_SubtypeB.fa
```
5. Run bash command to replace old_header with new header, K03455.1
   
```
   sed -i 's/^>insert_old_header/>K03455.1/' HXB2_SubtypeB.fa > HXB2_SubtypeB.fa
```
6. Replace current gff seqIDs to match this new header to ensure alignment with your FASTA file. Manually do this in text editor or use Chat-GPT AI to generate gff file with matching IDs.
7. Repeat section 4 to upload annotation track with a GFF file.

# 8. Add synteny tracks using Pairwise Alignment .paf files
1. Install minimap2, a platform that generates pairwise alignment given FASTA sequences
```
  sudo apt install minimap2
```
2. Generate paf
```
  minimap2 -x asm5 HIV_SubtypeA.fa CRF01_AE.fa >SubtypeA_vs_SubtypeAE.paf
```
3. Add track
```
  jbrowse add-track SubtypeA_vs_SubtypeAE.paf --category SyntenyTrack --assemblyNames HIV1_SubtypeA,CRF01_AE --load copy --out /var/www/html/jbrowse2
```
