# T1-2020 - Pipeline de Detecção de Variantes Germinativas

# gitbash

* Para Windows - Terminal
	https://github.com/git-for-windows/git/releases/download/v2.28.0.windows.1/PortableGit-2.28.0-32-bit.7z.exe
	
* Para Macbook: 
	Buscar no Spotlight (buscar por Terminal - dois cliques)

```bash
ssh -o "ServerAliveInterval 120" -p SUA_PORTA convidado@3.236.18.138
```

# Primeiros Comandos

```bash
# ls para listar arquivos e diretorios
ls

# pwd para informar o diretorio atual
pwd

# cd para entrar e sair de diretorios 
cd resultados/

# e voltar para casa
cd ../

# aqui é a sua casa (seu home) 
cd
# ~ é a sua casa
cd ~

# criar o diretorio no home (caso ele ainda nao exista)
mkdir ~/bioinfo/
mkdir ~/bioinfo/resultados
mkdir ~/bioinfo/reference
mkdir ~/bioinfo/data
mkdir ~/bioinfo/data/fastq

```

Fonte: [Comandos Básicos do Terminal Linux](http://swcarpentry.github.io/shell-novice/)

# Diretório de Resultados

```bash
# volta para casa
cd

# mkdir para criar um diretorio
mkdir ~/bioinfo/resultados

# cd para entrar no diretorio resultados
cd ~/bioinfo/resultados/

# mkdir para criar o diretorio das amostras
mkdir NA12878_S1 NA12878_S2 NA12878_S3
```


# Copiar Arquivos .FASTQ

```bash
# copiando os arquivos fastq para o diretorio ~/bioinfo/data/fastq
cp /data/iSeqAmpliSeqRun/Data/Intensities/BaseCalls/iSeq-AmpliSeq/NA12878_S1_R*_001.fastq.gz ~/bioinfo/data/fastq
cp /data/iSeqAmpliSeqRun/Data/Intensities/BaseCalls/iSeq-AmpliSeq/NA12878_S2_R*_001.fastq.gz ~/bioinfo/data/fastq
cp /data/iSeqAmpliSeqRun/Data/Intensities/BaseCalls/iSeq-AmpliSeq/NA12878_S3_R*_001.fastq.gz ~/bioinfo/data/fastq

# voltar para o home
cd
```


# Instalação

No terminal do Linux, vamos instalar alguns pacotes: (o resto vem instalado).

``` bash
# install dabases annovar
# NOTA: entreno no site do ANNOVAR com seu e-mail e salve o arquivo no diretorio: ~/bionfo/app/

# entrar no diretorio
mkdir ~/bioinfo/app/
cd ~/bioinfo/app/

# preencher formulario para receber link de download
https://doc-openbio.readthedocs.io/projects/annovar/en/latest/user-guide/download/

# fazer download e descompactar 
wget -c http://www.openbioinformatics.org/annovar/download/0wgxR2rIVP/annovar.latest.tar.gz
tar -xzf annovar.latest.tar.gz

# alternativamente copiar do diretório compartilhado /data
cd ~/bioinfo/app/
cp -r /data/bioinfo/app/annovar/ .

cd ~/bioinfo/app/annovar/humandb/
# baixar as bases: clinvar e exac03
perl ~/bioinfo/app/annovar/annotate_variation.pl -buildver hg19 -downdb -webfrom annovar clinvar_20200316  ~/bioinfo/app/annovar/humandb/
perl ~/bioinfo/app/annovar/annotate_variation.pl -buildver hg19 -downdb -webfrom annovar exac03 ~/bioinfo/app/annovar/humandb/
```


# Bioinformática: Pipelines e Comandos


# Olá Ambiente de Bioinformática
***Linux:*** O Linux é um sistema operacional, assim como o Windows da Microsoft e o Mac OS da Apple. Ele foi criado pelo finlandês Linus Torvalds, e o nome é a mistura do nome do criador com Unix, um antigo sistema operacional da empresa de mesmo nome. [O que é Linux?](https://www.techtudo.com.br/artigos/noticia/2011/12/o-que-e-linux.html).

***Pipelines:*** Primeiro, o pipeline não é um termo de bioinformática, é na verdade um termo de ciência da computação, envolve encadeamento de processos / threads / funções etc. Em resumo, o resultado de um processo é a entrada de um novo processo na sequência. [A review of bioinformatic pipeline frameworks](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5429012/)

# Qual ambiente vamos utilizar?
Vamos utilizar uma instância na Amazon Cloud com Sistema Operacional Linux Ubuntu. Os programas para que possamos criar nosso pipeline de análises de sequenciamento de nova geração já foram instalados e serão apenas listados nesse documento. O objetivo é melhorar a experiência de usuários iniciantes em "digitar comandos em um terminal linux" e ser um roteiro para que todos possam se localizar.

# Estrutura de Diretórios
Estrutura de diretórios: sequências, programas e arquivos de referência.

* ~/bioinfo
	* ~/bioinfo/app
	* ~/bioinfo/data
		* ~/bioinfo/data/fastq
	* ~/bioinfo/reference
  * ~/bioinfo/resultados


# Programas Instalados
Listamos os programas previamente instalados em nosso ambiente para executar o pipeline de chamada de varinates:

* annovar [Download](http://annovar.openbioinformatics.org/)
* bwa [Download](http://bio-bwa.sourceforge.net/)
* FastQC [Download](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
* freebayes [Download](https://github.com/ekg/freebayes)
* samtools [Download](http://samtools.sourceforge.net/)
* gatk [Download](https://github.com/broadinstitute/gatk/releases)
* picard-tools [Download](https://broadinstitute.github.io/picard/)




# Download das Referências
Nesta estapa vamos utilizar dois cromossomos humanos.

Acessar o site: [Sequence and Annotation Downloads](http://hgdownload.cse.ucsc.edu/downloads.html)


```bash

cd ~/bioinfo/reference

# wget para fazer download do chr13
wget -c http://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr13.fa.gz

# wget para fazer download do chr17 
wget -c http://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr17.fa.gz

# cat para concatenar os arquivo do cromossomo 13 e 17 em hg19.fa
zcat chr13.fa.gz chr17.fa.gz > hg19.fa

# rm para deletar os arquivos chr13.fa e chr17.fa
rm chr13.fa.gz chr17.fa.gz
```

# BWA: index reference
Agora, é preciso indexar o arquivo FASTA da referência. Todos os programas de alinhmaneto tem esta etapa que otimiza o processo de alinhamento das nossas sequências na referência.

***NOTA:*** A etapa de index é feita apenas uma vez para cada arquivo de referência:

```bash
# bwa index para gerar o index da referencia hg19.fa

# entrar no diretorio reference
cd ~/bioinfo/reference

# indexar o arquivo hg19.fa
bwa index hg19.fa 
```

## Samtools: index reference 

***NOTA:*** A etapa de index é feita apenas uma vez para cada arquivo de referência:

```
cd ~/bioinfo/reference

samtools faidx hg19.fa 
```

## Picard: dicionário das sequências FASTA para ferramentas GATK

***NOTA:*** A etapa de criação de disionário das sequências é feita apenas uma vez para cada arquivo de referência:

```
cd ~/bioinfo/reference

picard-tools CreateSequenceDictionary \
REFERENCE=hg19.fa \
OUTPUT=hg19.dict
```

# FastQC: Relatório de Controle de Qualidade
Gerar relatório de controle de qualidade com FastQC (Tempo ~10s):

```bash
# fastqc para gerar relatorio de qualidade dos arquivo FASTQ
# opcao (-o ./) diz para salvar o resultado no mesmo arquivo em que o comando esta sendo rodado.

# cd para voltar para a casa
cd

# rodar fastqc e salvar o resultado de cada amostra em seu diretorio
fastqc -o ~/bioinfo/resultados/NA12878_S1/ ~/bioinfo/data/fastq/NA12878_S1_R1_001.fastq.gz ~/bioinfo/data/fastq/NA12878_S1_R2_001.fastq.gz
```

FastQC NA12878_S1 resultado


~10 min


# BWA-mem: maximal exact matches
Alinha sequencias de tamanho 70bp-1Mbp com o algoritmo BWA-MEM. Em resumo o algoritmo trabalha com "alinhamento por sementes" com maximal exact matches (MEMs) e então estendendo sementes com o algoritmo Smith-Waterman (SW). Link. Tempo (~60s):

```bash
# cd para voltar para casa
cd

# rodar bwa para alinhar as sequencias contra o genoma de referencia
bwa mem -R '@RG\tID:NA12878_S1\tSM:NA12878_S1\tLB:Illumina\tPL:iSeq'  ~/bioinfo/reference/hg19.fa ~/bioinfo/data/fastq/NA12878_S1_R1_001.fastq.gz ~/bioinfo/data/fastq/NA12878_S1_R2_001.fastq.gz > ~/bioinfo/resultados/NA12878_S1/NA12878_S1.sam
```

BWA-mem NA12878_S1 resultado


# samtools: fixmate, sort e index

## samtools fixmate

Preencha coordenadas de posicionamento, posicione FLAGs relacionadas a partir a alinhamentos classificados por nome. Tempo (~5s):

```bash
samtools fixmate ~/bioinfo/resultados/NA12878_S1/NA12878_S1.sam ~/bioinfo/resultados/NA12878_S1/NA12878_S1.bam
```

SAMTOOLS fixmate NA12878_S1 resultado


Tempo: ~1min

## samtools sort

O ***samtools sort*** vai ordenar de nome para ordem de coordenadas. Tempo (5s):

```bash
samtools sort -O bam -o ~/bioinfo/resultados/NA12878_S1/NA12878_S1_sort.bam -T /tmp/ ~/bioinfo/resultados/NA12878_S1/NA12878_S1.bam 
```

SAMTOOLS sort NA12878_S1 resultado


Tempo: ~1min

## samtools index

O ***samtools index*** cria um index (.BAI) do arquivo binário (.BAM):

```bash
samtools index ~/bioinfo/resultados/NA12878_S1/NA12878_S1_sort.bam 
```

SAMTOOLS index NA12878_S1 resultado


Tempo: ~1min

## HaplotypeCaller: chamador de variantes do GATK;
O HaplotypeCaller é capaz de chamar SNPs e indels simultaneamente por meio de montagem de novo local de haplótipos em uma região ativa. Em outras palavras, sempre que o programa encontra uma região que mostra sinais de variação, ele descarta as informações de mapeamento existentes e remonta completamente as leituras naquela região. Isso permite que o HaplotypeCaller seja mais preciso ao chamar regiões que são tradicionalmente difíceis de chamar, por exemplo, quando contêm diferentes tipos de variantes próximas umas das outras. Isso também torna o HaplotypeCaller muito melhor em chamar indels do que chamadores baseados em posição como UnifiedGenotyper.
```
time gatk HaplotypeCaller -R ~/bioinfo/reference/hg19.fa \
-I ~/bioinfo/resultados/NA12878_S1/NA12878_S1_sort.bam \
-O ~/bioinfo/resultados/NA12878_S1/NA12878_S1.haplotypecaller.vcf
```

HaplotypeCaller call variant NA12878_S1 resultado

# freebayes: chamador de variantes
O FreeBayes é um detector variante genético Bayesiano projetado para encontrar pequenos polimorfismos, especificamente SNPs (polimorfismos de nucleotídeo único), indels (inserções e deleções), MNPs (polimorfismos de múltiplos nucleotídeos) e eventos complexos (eventos compostos de inserção e substituição) menores que os comprimento de um alinhamento de seqüenciamento de leitura curta. Link. Tempo (~6min):

```bash
freebayes -f ~/bioinfo/reference/hg19.fa -F 0.01 -C 1 --pooled-continuous ~/bioinfo/resultados/NA12878_S1/NA12878_S1_sort.bam > ~/bioinfo/resultados/NA12878_S1/NA12878_S1.freebayes.vcf
```

### Parâmetros


```bash
 -C --min-alternate-count N
                   Require at least this count of observations supporting
                   an alternate allele within a single individual in order
                   to evaluate the position.  default: 2

 --pooled-continuous
                   Output all alleles which pass input filters, regardles of
                   genotyping outcome or model.
```


FREEBAYES call variant NA12878_S1 resultado


Tempo: ~10min

# annovar: anotador de variantes
ANNOVAR éma ferramenta eficiente para anotar funcionalmente variantes genéticas detectadas a partir de diversos genomas (incluindo o genoma humano hg18, hg19, hg38, bem como mouse, verme, mosca, levedura e muitos outros). [Link]. Aqui vamos converter .VCF para .avinput. Tempo (~5s):


```bash                   
perl ~/bioinfo/app/annovar/convert2annovar.pl -format vcf4 ~/bioinfo/resultados/NA12878_S1/NA12878_S1.haplotypecaller.vcf > ~/bioinfo/resultados/NA12878_S1/NA12878_S1.avinput
``` 

***output: mensagens na tela***
 
```bash
A12878_S1.haplotypecaller.vcf > ~/bioinfo/resultados/NA12878_S1/NA12878_S1.avinput
NOTICE: Finished reading 1939 lines from VCF file
NOTICE: A total of 1912 locus in VCF file passed QC threshold, representing 1694 SNPs (1118 transitions and 576 transversions) and 219 indels/substitutions
NOTICE: Finished writing 1694 SNP genotypes (1118 transitions and 576 transversions) and 219 indels/substitutions for 1 sample
WARNING: 1 invalid alternative alleles found in input file
```

**Arquivo .avinput**


```bash
# comando head para listar as 10 primeiras linhas do arquivo NA12878_S1.avinput
head ~/bioinfo/resultados/NA12878_S1/NA12878_S1.avinput

chr13   19088055        19088055        C       G       hom     37.32   2
chr13   19088085        19088085        G       C       hom     78.32   2
chr13   19088106        19088106        A       C       hom     78.32   2
chr13   19088113        19088113        C       T       hom     78.32   2
chr13   19088147        19088147        C       A       hom     78.32   2
chr13   19088155        19088155        G       A       hom     78.32   2
chr13   19088156        19088156        C       A       hom     78.32   2
chr13   19088171        19088171        A       G       hom     78.32   2
chr13   19106555        19106555        A       T       hom     661.06  15
chr13   19106558        19106558        G       T       hom     661.06  15
```
 
ANNOVAR convert2annovar NA12878_S1 resultado


Anotar as variantes chamadas utilizando algumas bases de dados públicas: Tempo (~5s).

```
perl ~/bioinfo/app/annovar/table_annovar.pl ~/bioinfo/resultados/NA12878_S1/NA12878_S1.avinput ~/bioinfo/app/annovar/humandb/ -buildver hg19 -out ~/bioinfo/resultados/NA12878_S1/NA12878_S1 -remove -protocol refGene,exac03,clinvar_20200316 -operation g,f,f -nastring .
```

***output:***

```bash
NOTICE: the --polish argument is set ON automatically (use --nopolish to change this behavior)
-----------------------------------------------------------------
NOTICE: Processing operation=g protocol=refGene

NOTICE: Running with system command <annotate_variation.pl -geneanno -buildver hg19 -dbtype refGene -outfile /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.refGene -exonsort -nofirstcodondel /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.avinput /home/convidado/bioinfo/app/annovar/humandb/>
NOTICE: Output files are written to /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.refGene.variant_function, /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.refGene.exonic_variant_function
NOTICE: Reading gene annotation from /home/convidado/bioinfo/app/annovar/humandb/hg19_refGene.txt ... Done with 72567 transcripts (including 17617 without coding sequence annotation) for 28263 unique genes
NOTICE: Processing next batch with 1914 unique variants in 1914 input lines
NOTICE: Reading FASTA sequences from /home/convidado/bioinfo/app/annovar/humandb/hg19_refGeneMrna.fa ... Done with 35 sequences
WARNING: A total of 448 sequences will be ignored due to lack of correct ORF annotation

NOTICE: Running with system command <coding_change.pl  /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.refGene.exonic_variant_function.orig /home/convidado/bioinfo/app/annovar/humandb//hg19_refGene.txt /home/convidado/bioinfo/app/annovar/humandb//hg19_refGeneMrna.fa -alltranscript -out /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.refGene.fa -newevf /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.refGene.exonic_variant_function>
-----------------------------------------------------------------
NOTICE: Processing operation=f protocol=exac03
NOTICE: Finished reading 8 column headers for '-dbtype exac03'

NOTICE: Running system command <annotate_variation.pl -filter -dbtype exac03 -buildver hg19 -outfile /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1 /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.avinput /home/convidado/bioinfo/app/annovar/humandb/ -otherinfo>
NOTICE: Output file with variants matching filtering criteria is written to /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.hg19_exac03_dropped, and output file with other variants is written to /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.hg19_exac03_filtered
NOTICE: Processing next batch with 1914 unique variants in 1914 input lines
NOTICE: Database index loaded. Total number of bins is 749886 and the number of bins to be scanned is 26
NOTICE: Scanning filter database /home/convidado/bioinfo/app/annovar/humandb/hg19_exac03.txt...Done
-----------------------------------------------------------------
NOTICE: Processing operation=f protocol=clinvar_20200316
NOTICE: Finished reading 5 column headers for '-dbtype clinvar_20200316'

NOTICE: Running system command <annotate_variation.pl -filter -dbtype clinvar_20200316 -buildver hg19 -outfile /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1 /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.avinput /home/convidado/bioinfo/app/annovar/humandb/ -otherinfo>
NOTICE: the --dbtype clinvar_20200316 is assumed to be in generic ANNOVAR database format
NOTICE: Output file with variants matching filtering criteria is written to /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.hg19_clinvar_20200316_dropped, and output file with other variants is written to /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.hg19_clinvar_20200316_filtered
NOTICE: Processing next batch with 1914 unique variants in 1914 input lines
NOTICE: Database index loaded. Total number of bins is 72394 and the number of bins to be scanned is 16
NOTICE: Scanning filter database /home/convidado/bioinfo/app/annovar/humandb/hg19_clinvar_20200316.txt...Done
-----------------------------------------------------------------
NOTICE: Multianno output file is written to /home/convidado/bioinfo/resultados/NA12878_S1/NA12878_S1.hg19_multianno.txt
```

**Arquivo .hg19_multiano.txt**

```bash
# filtro com o comando grep para buscar apenas o cabeçalho e variantes do tipo exonic
grep "^Chr\|exonic" ~/bioinfo/resultados/NA12878_S1/NA12878_S1.hg19_multianno.txt  | head

Chr     Start   End     Ref     Alt     Func.refGene    Gene.refGene    GeneDetail.refGene      ExonicFunc.refGene      AAChange.refGene      ExAC_ALL        ExAC_AFR        ExAC_AMR        ExAC_EAS        ExAC_FIN        ExAC_NFE        ExAC_OTH     ExAC_SAS CLNALLELEID     CLNDN   CLNDISDB        CLNREVSTAT      CLNSIG
chr13   21893979        21893979        -       AATTTTTTGATGAAACAAGACGACTTTGT   ncRNA_exonic    GRK6P1  .       .       .    ..       .       .       .       .       .       .       .       .       .       .       .
chr13   28601353        28601353        G       A       exonic  FLT3    .       synonymous SNV  FLT3:NM_004119:exon17:c.C2079T:p.Y693Y        .       .       .       .       .       .       .       .       .       .       .       .       .
chr13   28601358        28601358        -       TGT     exonic  FLT3    .       nonframeshift insertion FLT3:NM_004119:exon17:c.2073_2074insACA:p.F691_E692insT       .       .       .       .       .       .       .       .       .       .       .    ..
chr13   28601359        28601359        -       ATGACCAGGGTGGGCCCT      exonic  FLT3    .       nonframeshift insertion FLT3:NM_004119:exon17:c.2072_2073insAGGGCCCACCCTGGTCAT:p.F691delinsLGPTLVI    .       .       .       .       .       .       .    ..       .       .       .       .
chr13   28601364        28601364        T       G       exonic  FLT3    .       nonsynonymous SNV       FLT3:NM_004119:exon17:c.A2068C:p.I690L        .       .       .       .       .       .       .       .       .       .       .       .       .
chr13   28886227        28886227        A       G       exonic  FLT1    .       nonsynonymous SNV       FLT1:NM_002019:exon26:c.T3395C:p.I1132T       .       .       .       .       .       .       .       .       .       .       .       .       .
chr13   28886231        28886231        -       GT      exonic  FLT1    .       frameshift insertion    FLT1:NM_002019:exon26:c.3390_3391insAC:p.Q1131Tfs*25  .       .       .       .       .       .       .       .       .       .       .       .    .
chr13   28886232        28886232        -       CC      exonic  FLT1    .       stopgain        FLT1:NM_002019:exon26:c.3389_3390insGG:p.Y1130*       .       .       .       .       .       .       .       .       .       .       .       .       .
chr13   28886235        28886235        -       AGAAGCAAAACA    exonic  FLT1    .       nonframeshift insertion FLT1:NM_002019:exon26:c.3386_3387insTGTTTTGCTTCT:p.I1129_Y1130insVLLL .       .       .       .       .       .       .       .       .    .
```

Utilize o commando `less -SN resultados/NA12878_S1/NA12878_S1.hg19_multianno.txt` para visualizar o arquivo de anotação. Para sair do comando less pressione a tecla `q`.

