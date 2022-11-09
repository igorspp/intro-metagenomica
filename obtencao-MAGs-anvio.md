# Obtenção de MAGs usando anvi'o

Nós utilizaremos o programa [anvi'o](https://anvio.org) para combinar contigs em **genomas oriundos da montagem de metagenoma** (MAGs, *metagenome-assembled genomes*).  
Primeiro nos conectamos a um nó de computação, ativamos o ambiente `conda` e criamos novas pastas para os resultados:  

```bash
srun --pty --account=project_2006567 --partition=small --cpus-per-task=40 --mem=20000 --time=1-00:00:00 --gres=nvme:10 bash

conda activate anvio7

for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  mkdir anvio_${sample}
done
```

A primeira coisa que precisamos fazer rodar um programa para reformatar os arquivos `.fasta` da nossa montagem, pois `anvi'o` não gosta do jeito que os contigs foram nomeados.  
Vamos aproveitar o mesmo comando para remover contigs menores que 2000 bp.  
Contigs curtos possuem baixa informação taxonômica, poucos ou nenhum gene e tornam o processo de combinação de contigs computacionalmente mais complicado.  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
    anvi-script-reformat-fasta montagens/${sample}/final.contigs.fa \
                            -o anvio_${sample}/contigs.fa \
                            -r anvio_${sample}/contigs_reformat.txt \
                            -l 2000 \
                            --prefix ${sample} \
                            --simplify-names
done
```

No próximo comando criaremos um dos artefatos do sistema `anvi'o`, o `CONTIGS.db`.  
`CONTIGS.db` é um banco de dados de [SQL](https://pt.wikipedia.org/wiki/SQL), onde informações sobre os contigs são armazenados.  
Além de criar o `CONTIGS.db`, o comando abaixo também rodará o programa [Prodigal](https://github.com/hyattpd/Prodigal) para identificar genes dentro dos contigs e calculará as frequências de kmers (`k = 4`).  
Todas essas informações são então adicionadas ao `CONTIGS.db`.  
Para saber mais, veja [aqui](https://anvio.org/help/main/artifacts/contigs-db/).  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-gen-contigs-database -f anvio_${sample}/contigs.fa \
                            -o anvio_${sample}/CONTIGS.db \
                            -n ${sample} \
                            -T 40
done
```
> **Pergunta #14:**  
> 
> \- Por que calculamos frequências de kmers e para que são utilizadas?

No comando abaixo, `anvi'o` usará o programa [hmmer](http://hmmer.org), que por sua vez utiliza [modelos ocultos de Markov](https://pt.wikipedia.org/wiki/Modelo_oculto_de_Markov), para identificar certos genes de cópia única que serão utilizados para calcular quão completos e redundantes ("contaminados") nossos MAGs são.       
Informação sobre a presença destes genes de cópia única são então adicionadas ao `CONTIGS.db`.  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-run-hmms -c anvio_${sample}/CONTIGS.db \
                -T 40
done
```

> **Pergunta #15:**  
> 
> \- Como a presença de genes de cópia única (*single-copy genes*) pode nos informar a respeito da completude e contaminação de um genoma/MAG?

No próximo comando, `anvi'o` tentará atribuir uma taxonomia aos genes de cópia única que foram encontrados no passo anterior.  
Isso é feito utilizando como referência o [Genome Taxonomy Database (GTDB)](https://gtdb.ecogenomic.org).  
Novamente, esta informação é então adicionada ao `CONTIGS.db`.  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-run-scg-taxonomy -c anvio_${sample}/CONTIGS.db \
                        -T 40
done
```

O próximo passo é coletar informação sobre a abundância dos contigs através das amostras.  
Para isso, utilizaremos o programa `bowtie2` para mapear/alinhar as sequências filtradas em relação aos contigs, gerando um arquivo `.sam`, que é então convertido um arquivo binário `.bam` usando o programa [Samtools](https://github.com/samtools/samtools).  

Note que no comando temos um efeito [*Inception*](https://www.adorocinema.com/filmes/filme-143692/) com um `for loop` dentro de outro `for loop`.  
Basicamente, as sequências filtradas de cada uma das quatro amostras são mapeadas à cada uma das quatro montagens que foram obtidas.  
O comando abaixo levará cerca de meia hora:  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  mkdir anvio_${sample}/mapeamento

  bowtie2-build anvio_${sample}/contigs.fa \
                anvio_${sample}/mapeamento/contigs

  for reads in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
    bowtie2 -1 seq_limpas/${reads}_1.fastq.gz \
            -2 seq_limpas/${reads}_2.fastq.gz \
            -S anvio_${sample}/mapeamento/${reads}.sam \
            -x anvio_${sample}/mapeamento/contigs \
            --threads 40 \
            --no-unal &> anvio_${sample}/mapeamento/${reads}_log.txt

    samtools view -F 4 -bS anvio_${sample}/mapeamento/${reads}.sam | samtools sort > anvio_${sample}/mapeamento/${reads}.bam
    samtools index anvio_${sample}/mapeamento/${reads}.bam
  done
done
```

Enquanto espera, responda:

> **Pergunta #16:**  
> 
> \- Por que estamos fazendo isso e por que não mapear as sequências filtradas de cada amostra somente à montagem orignária da mesma amostra? Em outras palavras, que outro aspecto importante sobre os contigs isso nos revela?  
> \- **Dica:** veja a pergunta #14 e o slide #19 da aula [metagenomica-nivel-genomas.pdf](metagenomica-nivel-genomas.pdf).

As informações geradas no passo anterior são então adicionadas a um novo banco de dados `SQL` chamado `PROFILES.db`.  
Dentre elas, o que nos interessa mais é a cobertura de cada nucleotídeo em cada uma das amostras e a presença de polimorfismos de nucleotídeo único (SNPs, *single-nucleotide polymorphisms*).  
Para saber mais sobre o `PROFILE.db`, veja [aqui](https://anvio.org/help/main/artifacts/profile-db/).  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  mkdir anvio_${sample}/profiles

  for reads in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
    anvi-profile -i anvio_${sample}/mapeamento/${reads}.bam \
                 -o anvio_${sample}/profiles/${reads} \
                 -c anvio_${sample}/CONTIGS.db \
                 -T 40
  done
done
```

No comando abaixo, combinamos os quatro `PROFILE.db` obtidos para cada montagem em um só e criamos matrizes de distância entre os contigs baseados na frequência de kmers e cobertura diferencial:  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-merge anvio_${sample}/profiles/*/PROFILE.db \
             -o anvio_${sample}/profiles_comb \
             -c anvio_${sample}/CONTIGS.db \
             -S ${sample}
done
```

Quando tudo tiver terminado, saia do nó de computação:

```bash
exit
```

Como **exceção**, para os próximos passos nós usaremos o programa `anvi-interactive` **dentro de um nó de login**.  
Isso se deve à uma certa complicação técnica  que não importa muito no momento, relacionada à criação de um [túnel SSH](https://docs.aws.amazon.com/pt_br/emr/latest/ManagementGuide/emr-ssh-tunnel-local.html) dentro de um nó de computação.  

Como saímos do nó de computação, precisamos ativar o ambiente `conda` novamente:  

```bash
conda activate anvio7
```

Agora utilizaremos o programa `anvi-interactive` para combinar contigs em MAGs, como demonstrado anteriormente:

> **Lembrete:** 
> 
> \- Não esqueça de salvar a coleção de MAGs clicando em `Store bin collection`.  
> \- Se salvar a coleção com um nome diferente de `default`, você vai ter que modificar toda vez nos próximos comandos onde temos a bandeira `-C default`.  
> \- Quando terminar uma amostra, aperte `ctrl+c` para terminar o servidor e ir para a próxima amostra dentro do `for loop`. 

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-interactive -c anvio_${sample}/CONTIGS.db \
                   -p anvio_${sample}/profiles_comb/PROFILE.db
done
```

Quando tiver terminado com o primeiro round, vamos dar uma olhada nos bins e ver quais podem ser refinados.  
A partir de agora, vamos fazer uma amostra de cada vez:

```bash
# Definir qual amostra vai ser analizada
sample=ERR4998600

# Gerar relatório de bins
anvi-estimate-genome-completeness -c anvio_${sample}/CONTIGS.db \
                                  -p anvio_${sample}/profiles_comb/PROFILE.db \
                                  -C default

# Lista com os nomes dos bins que queremos refinar
bins='Bin_7 Bin_9 Bin_10 Bin_15 Bin_17 Bin_18 Bin_19 Bin_20 Bin_21 Bin_23 Bin_24 Bin_25 Bin_29 Bin_30 Bin_31'

# E então lançamos o progama anvi-refine para os bins acima:
for bin in $bins; do
  anvi-refine -c anvio_${sample}/CONTIGS.db \
              -p anvio_${sample}/profiles_comb/PROFILE.db \
              -C default \
              -b $bin
done
```

> **Lembrete:** 
> 
> \- Não esqueça de salvar os MAGs refinados clicando em `Store refined bins in the database`.  
> \- Quando terminar um bin, aperte `ctrl+c` para terminar o servidor e ir para o próximo bin dentro do `for loop`. 

Quando tiver terminado a primeira amostra, mude o valor da variavel `$sample` acima e faça o mesmo para as outras três amostras.  

Quando tiver terminado de refinar todos os MAGs que quiser de todas as quatro amostras, rodaremos dois programas dentro de um nó de computação:  

```bash
srun --pty --account=project_2006567 --partition=interactive --mem=10000 --time=06:00:00 bash

conda activate anvio7
```

O primeiro comando criará uma nova coleção e renomeará os nossos bins.  
Bins que estão dentro dos limites definidos em `--min-completion-for-MAG` e `--max-redundancy-for-MAG` serão chamados de MAGs.  
De acordo com o nível de estringência, podemos modificar esses valores para termos menos MAGs mas de melhor qualidade (limite alto para *completeness* e baixo para *redundancy*), ou mais MAGs de menor qualidade (limite baixo para *completeness* e alto para *redundancy*).  
Aqui, utilizaremos valores mais ou menos relaxados, i.e. 50% completude e 10% redundância.  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-rename-bins -c anvio_${sample}/CONTIGS.db \
                   -p anvio_${sample}/profiles_comb/PROFILE.db \
                   --report-file anvio_${sample}/MAGs_renomeados.txt \
                   --collection-to-read default \
                   --collection-to-write final \
                   --prefix "$USER"_"$sample" \
                   --call-MAGs \
                   --min-completion-for-MAG 50 \
                   --max-redundancy-for-MAG 10
done
```

No comando acima criamos uma nova coleção chamada `final`, e nossos bins foram renomeados baseado nos limites definidos.  

> **Dica:**  
> 
> Se você quiser você pode usar o comando `anvi-show-collections-and-bins` para listar as coleções que foram criadas e os nomes dos bins contidos em cada coleção. Para saber mais sobre o comando rode `anvi-show-collections-and-bins -h` ou abra [aqui](https://anvio.org/help/main/programs/anvi-show-collections-and-bins/).    

Agora rodaremos um último comando que irá criar diversos tipos de arquivos sobre os bins para serem utilizados nas análises seguintes.  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  anvi-summarize -c anvio_${sample}/CONTIGS.db \
                 -p anvio_${sample}/profiles_comb/PROFILE.db \
                 -o anvio_${sample}/summary \
                 -C final
done
```

Abra o arquivo `bins_summary.txt` localizado dentro de cada uma das quatro pastas `summary` e responda:  

> **Pergunta #17:**  
> 
> \- Quantos MAGs (i.e. > 50% completos e < 10% redundantes) foram obtidos no total?  
> \- Você espera que os MAGs sejam completamente diferentes entre si ou não?  
> \- E quanto aos MAGs obtidos pelos seus colegas nesse curso, eles serão diferentes aos seus?
