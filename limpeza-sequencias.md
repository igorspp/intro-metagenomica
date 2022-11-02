# Limpeza de sequências

Como observamos na atividade anterior, uma porcentagem significativa das sequências brutas contém alguns problemas como presença de adaptadores.  
Antes de prosseguirmos, é necessário então realizar uma limpeza das sequências brutas.  
Para isso, utilizaremos o programa [Cutadapt](https://cutadapt.readthedocs.io/en/stable/).  

> **NOTA SOBRE O USO DE *FOR LOOPS*:**  
> 
> Ao contrário do `FastQC`, que analisa múltiplos arquivos em um só comando, `Cutadapt` trabalha com um par de sequências *forward* e *reverse* de cada vez.  
> No nosso caso, então seria necessário rodar o `Cutadapt` 4 vezes, ou seja, uma vez para cada amostra.  
> Existem diversas formas de fazer isso, a mais simples talvez seria criando 4 cópias do mesmo comando e, em cada uma delas, incluir os nomes dos arquivos de uma das amostras.  
> Em um exemplo simplificado:  
> 
> ```bash
> ################## NÃO RODAR ##################
> cutadapt ERR1713356_1.fastq ERR1713356_2.fastq
> cutadapt ERR2683233_1.fastq ERR2683233_2.fastq
> cutadapt ERR4998600_1.fastq ERR4998600_2.fastq
> cutadapt ERR4998601_1.fastq ERR4998601_2.fastq
> ################## NÃO RODAR ##################
> ```  
> 
> Isto até que não seria um problema para as nossas 4 amostras, mas análises de metagenômica geralmente involvem dezenas/centenas/milhares de amostras.  
> Nesses casos, o modo acima não é muito eficiente pois 1) seria chato e monótono para preparar e rodar, 2) ficaria sujeito a erros de digitação, e 3) não é fácil de transferir para outro set de amostras.  
> 
> Uma das maneiras que podemos utilizar para rodar o mesmo comando múltiplas vezes involve o uso de um `for loop`.  
> Em um exemplo simplificado:
> 
> ```bash
> ######################### NÃO RODAR #########################
> for sample in ERR1713356 ERR2683233 ERR4998600 ERR4998601; do
>   cutadapt ${sample}_1.fastq ${sample}_2.fastq
> done
> ######################### NÃO RODAR #########################
> ```
> 
> **Quebrando em pedaços:**  
> \- Na linha #1, definimos um nome de variável que queremos utilizar. Aqui eu escolhi `sample`, mas poderia muito bem ter utilizado algo simples como `i`, ou algo mais complicado e informativo como `amostra_metagenomica`.  
> \- Ainda na linha #1, incluímos uma lista de items que serão passsados para o comando, nesse caso, os nomes das nossas amostras (`ERR1713356 ERR2683233 ERR4998600 ERR4998601`).  
> \- Na linha #2, definimos o comando/programa que será rodado (`cutadapt`), seguido pela variável que definimos na linha #1 (`${sample}`), e o restante do nome dos arquivos (`_1.fastq` e `_2.fastq` ).  
> \- E finalmente, a linha #3 sinaliza o fim do loop.  
> 
> Ao executar o loop acima, em cada uma das 4 iterações, a variável `${sample}` será substituída pelo nome de uma das 4 amostras que definimos na primeira linha; em efeito, gerando automaticamente o nome dos arquivos que serão analisados.  

## Limpando as sequências brutas

Agora vamos rodar o programa `Cutadapt` para realizar a limpeza das sequências brutas.  

Primeiro, nos conectamos a um nó de computação usando `srun`, ativamos o ambiente `conda` onde `Cutadapt` está instalado, e criamos uma pasta onde as sequências limpas serão armazenadas:  

```bash
srun --pty --account=project_2006567 --partition=interactive --cpus-per-task=4 --mem=2000 --time=02:00:00 bash
conda activate cutadapt
mkdir seq_limpas
```

Antes de rodar o programa efetivamente, vamos analizar o comando:

```bash
################## NÃO RODAR ###################
cutadapt ../seq_brutas/${sample}_1.fastq \
         ../seq_brutas/${sample}_2.fastq \
         -o seq_limpas/${sample}_1.fastq.gz \
         -p seq_limpas/${sample}_2.fastq.gz \
         -a CTGTCTCTTATACACATCTCCGAGCCCACGAGAC \
         -A CTGTCTCTTATACACATCTGACGCTGCCGACGA \
         -q 20 \
         -m 50 \
         -j 4 > seq_limpas/${sample}.log.txt
################## NÃO RODAR ###################
```

> **Pergunta #4:**  
> 
> Olhando o manual do `Cutadapt` [online](https://cutadapt.readthedocs.io/en/stable/index.html) ou rodando `cutadapt --help`, responda:  
> \- O que significam as bandeiras `-o`, `-p`, `-a`, `-A`, `-q`, `-m` e `-j`?  
> \- Por que os argumentos para `-o` e `-p` terminam em `.fastq.gz`?  
> \- Qual o propósito do redirecionamento (`> seq_limpas/${sample}.log.txt`)?  
> 
> Além disso, qual o propósito da barra invertida (`\`) no comando acima?


Agora que entendemos bem o comando, vamos rodar o `Cutadapt`.  
Mas como nada nunca é tão simples, temos que prestar atenção a um detalhe: as amostras vêm de projetos diferentes onde foram utilizados kits diferentes de preparação de bibliotecas que por sua vez têm sequências diferentes de adaptadores.  
Então na prática precisamos rodar o `Cutadapt` em duas levas diferentes, uma para as amostras de solo de tundra e outra para as amostras de ETE.  

Para as amostras de tundra:  

```bash
for sample in ERR4998600 ERR4998601; do
  cutadapt ../seq_brutas/${sample}_1.fastq \
           ../seq_brutas/${sample}_2.fastq \
           -o seq_limpas/${sample}_1.fastq.gz \
           -p seq_limpas/${sample}_2.fastq.gz \
           -a CTGTCTCTTATACACATCTCCGAGCCCACGAGAC \
           -A CTGTCTCTTATACACATCTGACGCTGCCGACGA \
           -q 20 \
           -m 50 \
           -j 4 > seq_limpas/${sample}.log.txt
done
```

E para as amostras de ETE:  

```bash
for sample in ERR1713356 ERR2683233; do
  cutadapt ../seq_brutas/${sample}_1.fastq \
           ../seq_brutas/${sample}_2.fastq \
           -o seq_limpas/${sample}_1.fastq.gz \
           -p seq_limpas/${sample}_2.fastq.gz \
           -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCA \
           -A AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT \
           -q 20 \
           -m 50 \
           -j 4 > seq_limpas/${sample}.log.txt
done
```

Note que as únicas coisas que diferem entre os dois loops são os argumentos passados para `-a` e `-A`.  
Por que?

> **Pergunta #5 (OPCIONAL):**  
> 
> Você conseguiria desenvolver um jeito de rodar `Cutadapt` para as 4 amostras utilizando um só `for loop`?  
> Ou seja, um `for loop` onde os argumentos passados para `-a` e `-A` dependem de qual é a amostra que está sendo analisada em cada iteração?  
> Talvez algo involvendo declarações condicionais `if/else` seria uma possibilidade? 

Quando `Cutadapt` terminar de rodar, vamos dar uma olhada nos arquivos `.log.txt` que foram criados.  

> **Pergunta #6:**  
> 
> Para cada uma das amostras, responda:  
> \- Quantos pares de sequências brutas tínhamos originalmente?  
> \- Quantas sequências continham adaptadores?  
> \- Quantos pares de sequências foram removidas porque eram muito curtas?  
> \- Quantos nucleotídeos foram removidos por terem baixa qualidade?  
> \- Qual a porcentagem de nucleotídeos que passou o controle de qualidade?

## Obtendo relatórios de qualidade das sequências limpas

Após a limpeza das sequências usando o `Cutadapt`, será que conseguimos nos livrar dos problemas que tínhamos observado nas sequências brutas?  
Para descobrir, vamos rodar `FastQC` e `MultiQC` novamente, mas agora para as sequências limpas:  

```bash
conda activate QC
mkdir fastqc_limpas
fastqc seq_limpas/*.fastq.gz -o fastqc_limpas -t 4
multiqc fastqc_limpas -o multiqc_limpas
```

Quando tudo tiver terminado, lembre-se de desconectar do nó de computação com o comando `exit`.  

Agora baixe a pasta `multiqc_limpas` para o seu computador como feito anteriormente, abra o arquivo `multiqc_report.html` no seu navegador favorito, e compare com o relatório obtido anteriormente para as sequências brutas.  

> **Pergunta #7:** 
>
> \- Quais problemas foram resolvidos?  
> \- E quais não foram?  
> \- Apareceram novos problemas que não existiam nas sequências limpas?  
> \- Podemos continuar nossas análises ou devemos repetir o processo de limpeza? De repente utilizando parâmetros diferentes ou até mesmo  um outro programa?
