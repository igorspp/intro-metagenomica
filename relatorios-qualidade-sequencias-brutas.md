# Relatórios de qualidade das sequências brutas

## Se conectando ao Puhti

Todas as análises de bioinformática serão realizadas no servidor [Puhti](https://puhti.csc.fi).  
As instruções de como se conectar ao Puhti usando o VS Code estão descritas nos slides [aqui](apanhado-geral-e-info-praticas.pdf).  

> **Lembrete:**  
> 
> Sempre que se conectar ao servidor, tenha certeza que está dentro da pasta certa usando o comando `pwd`. Se você não estiver no lugar certo, use o comando `cd` para ir para a sua pasta de trabalho (`/scratch/project_2006567/student3xx`). Se quiser, você pode usar a varíavel `$USER` ao invés de digitar o seu nome de usuário (por exemplo, `cd /scratch/project_2006567/$USER`).  

## Localização das sequências brutas

> **Sobre as amostras:**  
> 
> Como explicado [aqui](apanhado-geral-e-info-praticas.pdf), as amostras que utilizaremos são:  
> \- 2 amostras de estação de tratamento de esgoto (ETE): ERR1713356 e ERR2683233  
> \- 2 amostras de solo de tundra: ERR4998600 e ERR4998601  
> 
> Você pode ler mais sobre as amostras [aqui](https://www.nature.com/articles/s41467-019-08853-3) e [aqui](https://environmentalmicrobiome.biomedcentral.com/articles/10.1186/s40793-022-00424-2).  
> As sequências são públicas e foram baixadas do repositório [*Sequence Read Archive* (SRA)](https://www.ncbi.nlm.nih.gov/sra) do NCBI usando o programa [fasterq-dump](https://github.com/ncbi/sra-tools).  

As sequências brutas das amostras que utilizaremos estão localizadas em `/scratch/project_2006567/seq_brutas`.  
Vamos listar o conteúdo da pasta:  

```bash
ls -l ../seq_brutas
```

> **Perguntas:**  
> 
> \- O que é o comando `ls`?  
> \- Qual o propósito da bandeira `-l`?  
> \- Por que utilizamos `../`?  
> \- Sem utilizar `../`, de que outra maneira você poderia listar o conteúdo dessa pasta?

Se tudo estiver correto – e ninguém deletou nada por acidente – o comando acima deverá listar 8 arquivos.  
Para cada uma das 4 amostras, temos um arquivo com as sequências *forward* (`*_1.fastq`) e outro com as sequências *reverse* (`*_2.fastq`).  

## Obtendo relatórios de qualidade das sequências brutas
O processo de sequenciamento de DNA está sujeito a diversos tipos de problemas que podem introduzir erros e artefatos nas sequências.  
Por isso, análises de bioinformática geralmente começam com um controle de qualidade de sequências brutas.  
Aqui utilizaremos os programas [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc) e [MultiQC](https://multiqc.info) para obter relatórios de qualidade.  

> **NOTA SOBRE NÓS DE LOGIN E NÓS DE COMPUTAÇÃO:**  
>
> Quando nos conectamos a um servidor como o Puhti, nós somos direcionados a um **nó de login**.  
> Nós de login são limitados a tarefas simples como mover arquivos, editar códigos, etc.  
> Para rodarmos programas e tarefas que demandam mais recursos como maior memória RAM, número de núcleos (*cores*), e tempo, precisamos antes nos conectar a um **nó de computação**.  
> Em servidores que usam o sistema `SLURM`, podemos nos conectar a um nó de computação usando o programa `srun`.  

Vamos então nos conectar a um nó de computação, solicitando 4 núcleos e 2 GB de RAM por 30 minutos:  

```bash
srun --pty --account=project_2006567 --partition interactive --cpus-per-task=4 --mem 2000 --time 00:30:00 bash
```

Em poucos segundos após rodar o comando acima você estará conectado a um nó de computação.  
O tempo de espera varia de acordo com quantos usuários estão utilizando o sistema nesse momento, mas solicitações involvendo poucos recursos geralmente levam alguns segundos.   
Quando você estiver conectado a um nó de computação, você verá que o *prompt* mudará de algo como:  
> `[student389@puhti-login12 student389]$`  

para algo como:  

> `[student389@r18c01 student389]$`  

É possível também saber o nome do nó ao qual você está conectado com o seguinte comando:  

```bash
echo $SLURMD_NODENAME
```

> **NOTA SOBRE CONDA:**  
> 
> Nesse curso utilizaremos o programa `conda` para gerenciar as instalações dos programas.  
> Usando `conda`, é possível ter no seu sistema diferentes ambientes contendo diferentes programas de modo que as instalações não interfiram umas com as outras.  
> Você pode ler mais sobre `conda` [aqui](https://docs.conda.io/en/latest/).  
> Para listar os diferentes ambientes que foram criado para esse curso você pode rodar:  
> 
> ```bash
> conda env list 
> # ou
> conda info --envs
> ```

Vamos então ativar o ambiente `conda` onde os programas que iremos utilizar (`FastQC` e `MultiQC`) estão instalados:  

```bash
conda activate QC
```

Para vermos se realmente temos acesso aos programas, e de quebra ver qual a versão dos programas instalados, podemos rodar:  

```bash
fastqc --version
multiqc --version
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

> **Perguntas:**  
> 
> \- O que significa `../seq_brutas/*.fastq`? O que o asterisco (`*`) está fazendo ali?  
> \- Qual o propósito da bandeira `-o` e o argumento `fastqc`?  
> \- E `-t 4`?  Qual a relação do valor 4 aqui com o comando `srun` que rodamos anteriormente?

Para as duas últimas perguntas, você pode (**e deve**) acessar a ajuda do programa para entender que tipos de bandeiras e argumentos podemos incluir no comando e o que eles fazem:  

```bash
fastqc -h
# ou
fastqc --help
```

> **Dica #1:**  
> 
> Para visualizarmos textos longos no terminal sem termos que ficar subindo e descendo a tela podemos utilizar o comando `less`.  
> Por examplo, digamos que queremos visualizar o arquivo `ERR1713356_1.fastq` (que contem 40000000 de linhas):  
> 
> ```bash
> less ../seq_brutas/ERR1713356_1.fastq
> ```
> 
> Apertando a barra de espaço ou `ctrl+f` podemos ir para a próxima "página", e com `ctrl+b` voltamos uma "página".  
> Para sair, digitamos `q`.  

> **Dica #2:**  
> Utilizando tubulações (*pipes*), representadas por uma barra vertical (`|`), podemos canalizar o resultado (*output*) de um comando para ser a entrada (*input*) de outro.  
> Por exemplo, para acessarmos a ajuda do programa `FastQC` de uma maneira que seja mais fácil de navegar, podemos utilizar:  
> 
> ```bash
> fastqc --help | less
> ```


Quando o programa `FastQC` tiver acabado de rodar, liste o conteúdo da pasta `fastqc`:  

```bash
ls -1 fastqc
```

Se tudo deu certo, você deverá ver 16 arquivos: um arquivo `.html` e um `.zip` para cada arquivo que foi analisado pelo programa.  
Para juntarmos os relatórios de cada amostra em um arquivo comum mais fácil de navegar, utilizaremos o programa `MultiQC`:  

```bash
multiqc fastqc -o multiqc
```

Novamente, acesse a ajuda do programa para entender o que significa o comando que acabamos de rodar:  

```bash
multiqc --help | less
```

Agora que terminamos de rodar as análises podemos nos desconectar do nó de computação:

```bash
exit
```

> **Dica:**  
> 
> Para ter certeza que você saiu do nó de computação e está de volta no nó de login, olhe o *prompt*, que agora deve mostrar algo como `[student389@puhti-login12 student389]$`.  
> Além disso, o comando `echo $SLURMD_NODENAME` agora deverá retornar uma linha em branco.  

## Visualizando os relatórios de controle de qualidade

Agora utilizando o painel *Explorer* do VS Code (`View -> Explorer`), clique com o botão direito na pasta `multiqc` e depois em `Download...` para baixá-la para o seu computador.  
No seu computador, vá para onde salvou a pasta e abra o arquivo `multiqc_report.html` no seu navegador favorito.  
Investigue o relatório e veja que tipos de problemas foram detectados nas sequências.  

> **Dica:** 
> 
> Para cada painel (por exemplo, *Sequence Quality Histograms* ou *Per Sequence Quality Scores*) você pode clicar no botão `?Help` à direita do título para entender mais o que está sendo representado pelo gráfico.  

Quando todos tiverem terminado as análises, nós iremos olhar novamente o relatório juntos e discutir se algo precisa ser feito para tentar corrigir possíveis problemas com as sequências.  
