# Análise das montagens de metagenomas

Agora que as nossas montagens estão prontas, podemos nos deparar com algumas perguntas:  

* Quantos contigs foram produzidos em cada montagem?
* Temos bastante contigs longos, ou maioritariamente contigs curtos?
* Qual o tamanho total das montagens? 
* Quão representativo do todo são as montagens; em outras palavras, qual a porcentagem das nossas comunidades que foram efetivamente montadas?

Para responder as 3 primeiras perguntas, podemos utilizar um programa chamado [MetaQUAST](https://quast.sourceforge.net/metaquast).  
Como sempre, primeiro nos conectamos a um nó de computação e ativamos o ambiente `conda` e então rodamos o programa:  

```bash
srun --pty --account=project_2006567 --partition=interactive --cpus-per-task=4 --mem=2000 --time=06:00:00 bash

conda activate quast

metaquast.py montagens/*/final.contigs.fa \
             -o montagens_CQ \
             -t 4 \
             --contig-thresholds 0,1000,2000,2500,5000,10000,25000,50000 \
             --max-ref-number 0
```

O `MetaQUAST` deve demorar alguns poucos minutos para terminar.  

Para responder a quarta pergunta acima, podemos usar a seguinte abordagem:  
\- Para cada amostra, pegamos as sequências originais (limpas) e as mapeamos à montagem.  
\- A proporção das sequências originais que puderam ser efetivamente mapeadas à montagem é conhecida como taxa de mapeamento (*mapping rate*).  
\- A taxa de mapeamento é então utilizada como um indicativo da proporção das comunidade original que está de fato representada na montagem.

Como quase tudo em bioinformática, existem diversos programas desenvolvidos para o mapeamento de sequências.   
Aqui utilizaremos um programa chamado [bowtie2](https://github.com/BenLangmead/bowtie2), novamente utilizando um `for loop`:  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  bowtie2-build montagens/${sample}/final.contigs.fa montagens_CQ/${sample}.index

  bowtie2 -1 seq_limpas/${sample}_1.fastq.gz \
          -2 seq_limpas/${sample}_2.fastq.gz \
          -S montagens_CQ/${sample}.sam \
          -x montagens_CQ/${sample}.index \
          --threads 4 \
          --no-unal 2> montagens_CQ/${sample}.bowtie.log.txt
  
done

# Podemos deletar alguns arquivos grandes que não precisamos
rm montagens_CQ/*.sam montagens_CQ/*.bt2
```

A etapa acima deve demorar cerca de meia hora para terminar.  
Desconecte do nó de computação com o comando `exit` e copie a pasta `montagens_CQ` para o seu computador e abra o arquivo `report.html` no seu navegador favorito.  
Vamos tentar responder as 3 primeira perguntas feitas lá no começo, e adiciono mais uma:

> \- Quantos contigs foram produzidos em cada montagem?  
> \- Temos bastante contigs longos, ou maioritariamente contigs curtos?  
> \- Qual o tamanho total das montagens?   
> \- Qual a melhor montagem? Na verdade, faz algum sentido uma comparação assim?

Finalmente, olhando os arquivos `.bowtie.log.txt`, podemos responder a última pergunta:

> \- Quão representativo do todo são as montagens; em outras palavras, qual a porcentagem das nossas comunidades que foram efetivamente montadas?  
