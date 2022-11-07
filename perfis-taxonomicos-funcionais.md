# Obtenção de perfis taxonômicos e funcionais

Nós utilizaremos o programa [MEGAN](https://uni-tuebingen.de/en/fakultaeten/mathematisch-naturwissenschaftliche-fakultaet/fachbereiche/informatik/lehrstuehle/algorithms-in-bioinformatics/software/megan6/) para a obtenção de perfis taxonômicos e funcionais.  
Para podermos rodar as análises mais rapidamente, nós utilizaremos somente uma parte das sequências - 500,000 sequências por amostra para ser mais exato.  

Primeiro nos conectamos à um nó de computação, ativamos o ambiente `conda` e criamos uma nova pasta onde os resultados serão escritos:  

```bash
srun --pty --account=project_2006567 --partition=small --cpus-per-task=40 --mem=100000 --time=1-00:00:00 bash
conda activate megan
mkdir megan
```

Então utlizaremos o programa [seqtk](https://github.com/lh3/seqtk) para reduzir o tamanho das bibliotecas:  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  seqtk sample -s 100 seq_limpas/${sample}_1.fastq.gz 500000 > megan/${sample}.fastq
done
```

> **Pergunta #10:**  
> 
> \- Qual a função do argumento `-s 100` no comando acima e qual é a sua importância?  

Então utilizaremos o programa [DIAMOND](https://github.com/bbuchfink/diamond) para buscarmos as sequências mais similares no banco de dados `nr` do [NCBI](https://www.ncbi.nlm.nih.gov):

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  diamond blastx -q megan/${sample}.fastq \
                 -o megan/${sample}.daa \
                 -d /appl/data/bio/biodb/production/diamond/nr \
                 -f 100 \
                 -p 40 2> megan/${sample}.diamond.log.txt
done
```

> **Pergunta #11:**  
> 
> \- O que faz o sub-programa `blastx` que estamos rodando dentro do `DIAMOND`?    
> \- Qual a vantagem de utilizar o programa `DIAMOND` ao invés do bom e velho [BLAST](https://blast.ncbi.nlm.nih.gov) em análises de metagenomas?  

O passo acima deverá tomar um pouco mais de 3 horas para terminar...  
Enquanto espera, assista esse [vídeo](https://www.youtube.com/watch?v=2-4th7O0rOU) feito pelo criador do `MEGAN`.  

Quando as buscas tiverem terminado, rodaremos o comando `daa-meganizer` do programa `MEGAN`.  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  daa-meganizer -i megan/${sample}.daa \
                -mdb ../megan-map-Feb2022.db 2> megan/${sample}.megan.log.txt
done
```

> **Pergunta #12:**  
> 
> \- Qual a função do programa `daa-meganizer`?

Agora baixe os arquivos `.daa` para o seu computador.  
Talvez demore um tempo para baixá-los pois os arquivos são relativamente grandes:  

```bash
du -sh megan/*.daa
```

Enquanto espera, baixe a versão gráfica (GUI) do `MEGAN` e instale **no seu computador**:

* MacOS: [https://software-ab.informatik.uni-tuebingen.de/download/megan6/MEGAN_Community_macos_6_24_4.dmg](https://software-ab.informatik.uni-tuebingen.de/download/megan6/MEGAN_Community_macos_6_24_4.dmg)
* Linux: [https://software-ab.informatik.uni-tuebingen.de/download/megan6/MEGAN_Community_unix_6_24_4.sh](https://software-ab.informatik.uni-tuebingen.de/download/megan6/MEGAN_Community_unix_6_24_4.sh)
* Windows: [https://software-ab.informatik.uni-tuebingen.de/download/megan6/MEGAN_Community_windows-x64_6_24_4.exe](https://software-ab.informatik.uni-tuebingen.de/download/megan6/MEGAN_Community_windows-x64_6_24_4.exe)

> **Dica:** 
> 
> Se tiver alguma dúvida sobre a instalação, dê uma olhado no [manual](http://software-ab.cs.uni-tuebingen.de/download/megan6/manual.pdf) do programa `MEGAN`.  

Copie também o arquivo [info_amostras.txt](info_amostras.txt), que irá nos ajudar a saber qual amostra é qual.  

Abra então o programa `MEGAN` no seu computador e investigue a composição taxonômica e funcional das amostras e responda:  

> **Pergunta #13:**  
> 
> \- Existem diferenças entre as amostras?  
> \- As amostras que vem de um mesmo ambiente são mais similares entre si?  
> \- Quais säo os filos mais abundantes em cada grupo de amostra? E em relação ao metabolismo?
