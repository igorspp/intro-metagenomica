# Exemplos para análise de MAGs

Após obter os MAGs, podemos seguir adiante as nossas análises.  
Que tipo de análises faremos, é claro, depende da nossa pergunta inicial.  
Abaixo temos alguns exemplos de programas que podemos rodar para obter mais informações sobre os MAGs.  

Vamos nos conectar a um nó de computação:

```
srun --pty --account=project_2006567 --partition=interactive --cpus-per-task=8 --mem=10000 --time=06:00:00 --gres=nvme:10 bash
```

Agora vamos coletar todos os MAGs que obtivemos e colocá-los todos em uma só pasta:

```bash
mkdir todos_MAGs

cp anvio_*/summary/bin_by_bin/*MAG*/*MAG*contigs.fa todos_MAGs
```

## GTDB-Tk

Uma coisa que normalmente queremos saber é a qual grupos taxonômicos nossos MAGs pertencem.  
Embora o `anvi'o` nos dá uma ideia preliminar, podemos usar uma plataforma mais dedicada para atribuição taxonômica de MAGs.  
Aqui usaremos o programa [GTDB-Tk](https://github.com/Ecogenomics/GTDBTk), uma ferramenta para inferir taxonomia para MAGs com base no banco de dados [GTDB](https://gtdb.ecogenomic.org).  

Para esse programa, iremos submeter nossas análises ao sistema `sbatch`.  
Para isso precisamos de um script; copie o seguinte arquivo que criei para vocês:  

```bash
cp ../gtdbtk-sbatch.sh .
```

Abra o arquivo e veja que tipo de informação o script contém.  
E finalmente submeta o script:

```bash
sbatch gtdbtk-sbatch.sh

# Anote o número produzido acima  e use o programa sacct para ver o progresso, por exemplo:
sacct -j 14137383
```

## dRep

Frequentemente alguns dos nossos MAGs podem ser bem próximos uns aos outros, especialmente quando estamos trabalhando com mais de uma montagem.  
Nesse caso pode ser interessante remover a redundância e trabalhar com apenas um MAG de cada grupo de MAGs semelhantes.  
Para isso podemos usar um programa chamado [dDep](https://drep.readthedocs.io/en/latest/):  

```bash
conda activate drep

dRep compare drep -g todos_MAGs/*.fa -p 8
```

Baixe a pasta `drep` para o seu computador e investigue os resultados.  
Olhe principalmente os arquivos `.pdf` dentro da pasta `drep/figures`.  
Quantos MAGs você tem originalmente? Eles representam quantos grupos de MAGs não-redundantes (únicos)?

## CheckM2

[CheckM2](https://github.com/chklovski/CheckM2) é um programa para avaliar a qualidade de genomas/MAGs.  
Apesar do `anvi'o` nos fornecer medidas de completude e contaminação/redundância, `CheckM` é amplamente utilizado e pode nos ajudar a entender melhor a qualidade dos nossos MAGs:  

```bash
conda activate checkm2

checkm2 predict -i todos_MAGs/ -o checkm2 -x .fa -t 8
```

## Anotando MAGs

Eu considero a anotação de MAGs de longe a tarefa mais complicada em um trabalho de metagenômica.  
Existem diversos tipos de programas e bancos de dados desenvolvidos para isso, mas qual usar depende muito da pergunta que você quer responder.  
Aqui nós usaremos um programa do `anvi'o` para anotar todos os genes encontrados em relação ao banco de dados [KEGG KOfam](https://www.genome.jp/tools/kofamkoala/):  

```bash
conda activate anvio7

for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-run-kegg-kofams -c anvio_${sample}/CONTIGS.db -T 8
done
```

Esse passo pode demorar algumas horas para finalizar.   
Quando terminado, as anotações serão armazenadas dentro de cada `CONTIGS.db`.  
Adiante veremos como recuperar essas e outras informações para análises posteriores.  

## Análise no R

Agora que obtemos algumas informações sobre os nossos MAGs, vamos utilizar o programa/linguagem de programação [R](https://www.r-project.org) para as próximas análises.  

> **Dica:**  
> 
> Se você não possui um conhecimento básico de R, eu recomendo começar a aprender o quanto antes.  
> R é uma das linguagens de programação mais utilizadas em bioinformática (junto com Python), e será crucial para fazer sentido de todos esses dados que obtivemos com metagenômica.

Vamos coletar alguns arquivos para que possamos importar pro R:  

```bash
mkdir R

# Classificação taxonômica
cp gtdb/gtdbtk.bac120.summary.tsv R
cp gtdb/gtdbtk.ar53.summary.tsv R

# Resultado do CheckM2
cp checkm2/quality_report.tsv R

# Informações sobre os MAGs, produzida pelo comando anvi-summarize anteriormente
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  cp anvio_${sample}/summary/bins_summary.txt R/${sample}.bins.summary.txt
done

# Abundância dos MAGs, também produzida pelo comando anvi-summarize
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  cp anvio_${sample}/summary/bins_across_samples/mean_coverage.txt R/${sample}.coverage.txt
  cp anvio_${sample}/summary/bins_across_samples/detection.txt R/${sample}.detection.txt
done

# Anotação
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-export-functions -c anvio_${sample}/CONTIGS.db \
                        -o R/${sample}.functions.txt \
                        --annotation-sources KOfam
done

# Links entre ORFs e contigs
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-export-gene-calls -c anvio_${sample}/CONTIGS.db \
                         -o R/${sample}.gene.calls.txt \
                         --gene-caller prodigal \
                         --skip-sequence-reporting
done

# Links entre contigs e bins
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-export-collection -p anvio_${sample}/profiles_comb/PROFILE.db \
                         -C final \
                         -O ${sample} \
                         --include-unbinned
  
  mv ${sample}.txt R/${sample}.contigs.txt
  rm ${sample}-info.txt
done
```

Agora vamos importar tudo pro `R` e ver o que podemos descobrir sobre os MAGs.  

## Para praticar mais: tutoriais `anvi'o`

* [A simple read recruitment exercise](https://merenlab.org/tutorials/read-recruitment/) 
* [An anvi'o workflow for microbial pangenomics](https://merenlab.org/2016/11/08/pangenomics-v2/)
* [Vibrio jascida pangenome: a mini workshop](https://merenlab.org/tutorials/vibrio-jasicida-pangenome/)
* [An anvi'o workflow for phylogenomics](https://merenlab.org/2017/06/07/phylogenomics/)
* [Metabolic enrichment of high-fitness MAGs from a longitudinal FMT study](https://merenlab.org/tutorials/fmt-mag-metabolism/)
* [Studying microbial population genetics with anvi'o](https://merenlab.org/2015/07/20/analyzing-variability/)
