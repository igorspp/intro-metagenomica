# Montagem de metagenomas

Agora que nos asseguramos que as sequências são de boa qualidade, continuaremos com a montagem de metagenomas.  
Existem diversos programas de montagem de (meta)genomas.  
Aqui utilizaremos o programa [MEGAHIT](https://github.com/voutcn/megahit), pois geralmente resulta em boas montagens e possui uma pegada de memória baixa em relação a outros programas.  

Primeiro, devemos nos conectamos a um nó de computação, mas como necessitamos de mais recursos usaremos um modo diferente ao anterior:  

```bash
srun --ntasks=1 --time=12:00:00 --mem=10G --account=project_2006567 --partition=small --cpus-per-task=40 --pty /bin/bash
```

Depois, ativamos o ambiente `conda` onde `MEGAHIT` está instalado, e criamos uma pasta onde as montagens serão armazenadas:  

```bash
conda activate megahit
mkdir montagens
```

Agora rodamos `MEGAHIT` para as 4 amostras, novamente utilizando um `for loop`:  

```bash
for sample in ERR4998600 ERR4998601 ERR1713356 ERR2683233; do
  megahit -1 seq_limpas/${sample}_1.fastq.gz \
          -2 seq_limpas/${sample}_2.fastq.gz \
          -o montagens/${sample} \
          -t 40 \
          --min-contig-len 1000
done
```

Em torno de 2h30min teremos nossa montagens prontas.  
Enquanto espera, dê uma olhada no manual do `MEGAHIT` [aqui](https://www.metagenomics.wiki/tools/assembly/megahit) e [aqui](https://github.com/voutcn/megahit/wiki) e veja se há algo que você poderia ter feito diferente.  
Por exemplo, quais são os valores padrão para `--k-min` e `--k-max` que foram utilizados?  
No caso das nossas amostras, seria melhor ter modificado esses valores?  
