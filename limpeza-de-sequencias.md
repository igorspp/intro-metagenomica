# Limpeza de sequências

Como observamos no passo anterior, uma porcentagem significativa das sequências brutas contém alguns problemas como presença de adaptadores.  
Antes de prosseguirmos, precisamos então realizar uma limpeza.  
Para isso, utilizaremos o programa [Cutadapt](https://cutadapt.readthedocs.io/en/stable/).  

Ao contrário do `FastQC`, que aceita múltiplos arquivos de uma só vez, `Cutadapt` trabalha com um par de sequências *forward* e *reverse* de cada vez.  
No nosso caso, então precisaríamos rodar `Cutadapt` 4 vezes, ou seja, uma vez para cada amostra.  
Existem diversas formas de fazer isso, a mais simples talvez seria criando 4 cópias do mesmo comando e, em cada uma delas, incluir os nomes dos arquivos de uma das amostras.  

Por exemplo, digamos que queremos visualizar as primeiras 10 linhas de cada arquivo `.html` geradas pelo programa `FastQC` no passo anterior.  
Nesse caso, precisaríamos rodar o comando `head` 8 vezes:  

```bash
head -n 10 fastqc/ERR1713356_1_fastqc.html
head -n 10 fastqc/ERR1713356_2_fastqc.html
head -n 10 fastqc/ERR2683233_1_fastqc.html
head -n 10 fastqc/ERR2683233_2_fastqc.html
head -n 10 fastqc/ERR4998600_1_fastqc.html
head -n 10 fastqc/ERR4998600_2_fastqc.html
head -n 10 fastqc/ERR4998601_1_fastqc.html
head -n 10 fastqc/ERR4998601_2_fastqc.html
```

Isto até que não seria um problema para 8 arquivos, mas análises de metagenômica geralmente involvem dezenas/centenas/milhares de amostras.  
Nesses casos, o modo acima não é muito eficiente pois 1) seria chato e monótono para preparar e rodar, 2) ficaria sujeito a erros de digitação, e 3) não é fácil de transferir para outro set de amostras.  

Uma das maneiras que podemos utilizar para rodar o mesmo comando múltiplas vezes involve o uso de um `for loop`.  
No caso acima, um `for loop` para visualizarmos as primeiras 10 linhas de cada arquivo consistiria por exemplo:


```bash
for sample in ERR1713356 ERR2683233 ERR4998600 ERR4998601
do
  head -n 10 fastqc/${sample}_1_fastqc.html
  head -n 10 fastqc/${sample}_2_fastqc.html
done
```

* Na linha #1: definimos um nome de variável que queremos utilizar. Aqui eu escolhi `sample`, mas poderia muito bem ter utilizado algo simples como `i`, ou algo mais complicado e informativo como `amostra_metagenomica`.  
* Ainda na linha #1, incluímos uma lista de items que serão analizados, nesse caso, os nomes das nossas amostras (`ERR1713356 ERR2683233 ERR4998600 ERR4998601`).  
* A linha #2 sinaliza o fim da lista e começo do comando que será invocado.  
* Nas linha #3 e #4, definimos o comando/programa que será rodado (`head -n 10`), seguido pelo nome da pasta onde os arquivos estão (`fastqc/`), a variável que definimos na linha #1 (`${sample}`), e o restante do nome dos arquivos (`_1_fastqc.html` ou `_2_fastqc.html` ).  
* E finalmente, a linha #4 sinaliza o fim do loop.  

Ao executarmos o loop acima, em cada uma das 4 interações, a variável `${sample}` será substituída pelo nome de uma das 4 amostras que definimos na lista da primeira linha; em efeito, gerando automaticamente o nome dos arquivos que serão analisados.  

Uma maneira mais elegante e ainda mais automática de definir o mesmo loop seria:  

```bash
for file in $(ls fastqc/*.html)
do
  head -n 10 $file
done
```

No exemplo acima, ao invés de passarmos manualmente a lista de items a serem analisados, usamos o comando `ls` para listar os arquivos `.html` existentes dentro da pasta `fastqc`, que são então passados automaticamente para o loop.  

---

Agora que nós entendemos como utilizar um `for loop` para analisar múltiplos arquivos, vamos rodar o programa `Cutadapt` para realizar a limpeza das sequências brutas.  

Primeiro, nos conectamos a um nó de computação, ativamos o ambiente `conda` onde `Cutadapt` está instalado, e criamos uma pasta onde as sequências limpas serão armazenadas:  

```bash
sinteractive -A project_2006567 -c 4
conda activate cutadapt
mkdir seq_limpas
```

Antes de rodar o programa efetivamente, vamos analizar o comando que iremos rodar:

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

Olhando o manual do `Cutadapt` [online](https://cutadapt.readthedocs.io/en/stable/index.html) ou rodando `cutadapt -h`, tente responder:  

* O que significam as bandeiras `-o`, `-p`, `-a`, `-A`, `-q`, `-m` e `-j`?  
* Por que os argumentos para `-o` e `-p` agora terminam em `.fastq.gz`?
* Qual o propósito do redirecionamento (`> seq_limpas/${sample}.log.txt`)?  

Finalmente, podemos então rodar o `Cutadapt`.  
Mas como nada nunca é tão simples, temos uma coisa que precisamos prestar atenção:   
As amostras vêm de projetos diferentes que não utilizaram os mesmos kits de preparação de bibliotecas metagenômicas.  
E esses kits diferentes utilizam sequências diferentes de adaptadores.  
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

Note que as únicas coisas que diferem entre os dois loops são os argumentos passados para `-a` e `-A`.  Por que?

**EXERCÍCIO OPCIONAL:**  
Você conseguiria desenvolver um jeito de rodar `Cutadapt` para as 4 amostras utilizando um só `for loop`?  
Ou seja, um `for loop` onde os argumentos passados para `-a` e `-A` dependem de qual é a amostra que está sendo analisada em cada interação?  
Talvez algo involvendo declarações condicionais `if/else` seria uma possibilidade? 

---

Quando `Cutadapt` terminar de rodar, vamos dar uma olhada nos arquivos `.log.txt` que foram criados.  
Para cada uma das amostras, tente responder:  

* Quantos pares de sequências brutas tínhamos originalmente?  
* Quantas sequências continham adaptadores?  
* Quantos pares de sequências foram removidas porque eram muito curtas?  
* Quantos nucleotídeos foram removidos por cause de baixa qualidade?  
* Qual a porcentagem de nucleotídeos que passou o controle de qualidade?

Também podemos rodar `FastQC` e `MultiQC` para as sequências limpas.  
Assim poderemos ver se algo está diferente em relação às sequências brutas, por exemplo, se ainda temos adaptadores:  

```bash
conda activate QC
mkdir fastqc_limpas
fastqc seq_limpas/*.fastq.gz -o fastqc_limpas -t 4
multiqc fastqc_limpas -o multiqc_limpas
```

Quando tudo tiver terminado, lembre-se de desconectar do nó de computação com o comando `exit`.  

Agora baixe a pasta `multiqc_limpas` para o seu computador, abra o arquivo `multiqc_report.html` no seu navegador favorito, e compare com o relatório obtido anteriormente para as sequências brutas.  
A qualidade das sequências está melhor ou não?  
