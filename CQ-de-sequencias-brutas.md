# Controle de qualidade de sequências brutas

As sequências brutas das amostras que utilizaremos estão localizadas em uma pasta na raíz do diretório do projeto (`/scratch/project_2006567/seq_brutas`).  

Primeiro vamos checar; onde estamos?  

```bash
pwd
```

Tenha certeza que você está trabalhando dentro da sua pasta.  
Por exemplo, o resultado do comando `pwd` para o usuário *student389* deve ser `/scratch/project_2006567/student389`.  
Se você não estiver no lugar certo, use o comando `cd` para ir para a sua pasta de trabalho.  
**Dica:** você pode usar a varíavel `$USER` ao invés de digitar o seu nome de usuário:  

```bash
cd /scratch/project_2006567/$USER
```

Agora vamos listar o conteúdo da raiz do diretório do projeto:  

```bash
ls -l ../
```

Perguntas:  
* O que é o comando `ls`?  
* Qual o propósito da bandeira `-l`?  
* O que significa `../`?  

O resultado do comando acima deve listar – se você estiver no lugar certo, claro – o conteúdo da raíz do diretório do projeto:  
* Pasta `miniconda3`: é onde estão instalados os programas que vamos utilizar, gerenciados via [conda](https://conda.io)  
* Pasta `seq_brutas`: é onde estão as sequências brutas das amostras que utilizaremos no curso  
* Pastas `student3xx`: é onde cada participante irá trabalhar  

Agora vamos listar o conteúdo da pasta `seq_brutas`:  

```bash
ls -l ../seq_brutas
```

Se tudo estiver correto – e ninguém deletou nada por acidente – o comando acima deverá listar 8 arquivos.  
Para cada uma das 4 amostras, temos um arquivo com as sequências *forward* (`*_1.fastq`) e outro com as sequências *reverse* (`*_2.fastq`).  
As amostras que utilizaremos são:  

* 2 amostras de estação de tratamento de esgoto (ETE): ERR1713356 e ERR2683233  
* 2 amostras de solo de tundra: ERR4998600 e ERR4998601  

Você pode ler mais sobre essas amostras [aqui](https://www.nature.com/articles/s41467-019-08853-3) e [aqui](https://environmentalmicrobiome.biomedcentral.com/articles/10.1186/s40793-022-00424-2).  
Essas sequências são públicas e foram baixadas do repositório [*Sequence Read Archive* (SRA)](https://www.ncbi.nlm.nih.gov/sra) do NCBI.  

---

Agora que nós entendemos de onde vem e onde estão as sequências brutas, nós realizaremos um controle de qualidade (CQ).  
Para isso, utilizaremos os programas [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc) e [MultiQC](https://multiqc.info).  
Ambos foram instalados dentro de um ambiente `conda` chamado simplesmente de `QC`.  

Quando logamos no sistema Puhti, somos direcionados a um **nó de login**.  
Nós de login são limitados a tarefas simples como mover arquivos, editar códigos, etc.  
Para rodarmos programas e tarefas que demandam mais RAM, precisamos antes nos conectarmos a um **nó de computação**:  

```bash
sinteractive -A project_2006567 -c 4
```

Explicação:  
* `sinteractive`: comando utilizado no Puhti para se conectar facilmente a um nó de computação interativo  
* `-A`: *account* (conta); projeto que será "cobrado" pelo uso dos recursos, nesse caso, o projeto do curso  
* `-c`: *cores* (núcleos); aqui estamos solicitando 4 núcleos de processamento para podemos rodar 4 processos simultaneamente  

Em poucos segundos depois de rodar o comando acima você estará conectado a um nó de computação.  
O tempo varia de acordo com quantos usuários estão utilizando o sistema nesse momento, mas para solicitações involvendo poucos recursos (e.g. pouca RAM e número de núcleos), geralmente demora alguns segundos.  
Quando você estiver conectado a um nó de computação, você verá que o *prompt* mudará de algo como `[student389@puhti-login12 student389]$` para algo como `[student389@r18c01 student389]$`.  
É possível também saber o nome do nó ao qual você está conectado com o comando `echo $SLURMD_NODENAME`.  

Depois de conectados a um nó de computação, precisamos ativar o ambiente `conda` onde os programas que iremos utilizar estão instalados:  

```bash
conda activate QC
```

Agora precisamos criar uma pasta onde os resultados da análise feita pelo programa `FastQC` serão armazenados:  

```bash
mkdir fastqc
```

E então finalmente podemos rodar o programa:  

```bash
fastqc ../seq_brutas/*.fastq -o fastqc -t 4
```

Se tudo der certo, após alguns poucos minutos a análise estará concluída.  

Perguntas:  
* O que significa `../seq_brutas/*.fastq`? O que o asterisco (`*`) está fazendo ali?  
* Qual o propósito da bandeira `-o` e o argumento `fastqc`?  
* E `-t 4`?  

Para as duas últimas perguntas, você pode (**e deve**) acessar a ajuda do programa para entender que tipos de bandeiras e argumentos podemos incluir no comando e o que eles fazem:  

```bash
fastqc -h
```

Agora liste o conteúdo da pasta `fastqc`:  

```bash
ls fastqc
```

Se tudo deu certo, você deverá ver 16 arquivos: um arquivo `.html` e um `.zip` para cada arquivo que foi analisado pelo programa.  
Para juntarmos os relatórios de cada amostra em um arquivo comum mais fácil de navegar, iremos utilizar o programa `MultiQC`:  

```bash
multiqc fastqc -o multiqc
```

Novamente, acesse a ajuda do programa para entender o que significa o comando que acabamos de rodar:  

```bash
multiqc -h
```

Após terminar de rodar as análises podemos nos desconectar do nó de computação:

```bash
exit
```

Para ter certeza que você saiu do nó de computação e está de volta no nó de login, olhe o *prompt*, que agora deve mostrar algo como `[student389@puhti-login12 student389]$`.  
O comando `echo $SLURMD_NODENAME` agora deverá retornar uma linha em branco.  

Agora baixe a pasta `multiqc` para o seu computador e abra o arquivo `multiqc_report.html` no seu navegador favorito.  
Investigue o relatório e veja se tudo está bem com as sequências ou não.  
Para cada painel (por exemplo, *Sequence Quality Histograms* ou *Per Sequence Quality Scores*) você pode clicar no botão `?Help` à direita do título para entender mais o que está sendo representado.  

Quando todos tiverem terminado suas análises, nós iremos olhar novamente o relatório juntos e discutir se algo precisa ser feito.  
