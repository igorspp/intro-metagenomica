# Montagem de metagenomas

Agora que nos asseguramos que as sequências são de boa qualidade, continuaremos com a montagem de metagenomas.  
Existem diversos programas que têm sido desenvolvidos para montagem de genomas e metagenomas.  
Aqui utilizaremos o programa [MEGAHIT](https://github.com/voutcn/megahit), pois geralmente resulta em boas montagens e possui uma pegada de memória baixa em relação a outros programas.  

Primeiro, devemos nos conectamos a um nó de computação.  
Note que agora solicitaremos consideravelmente mais recursos que nos passos anteriores:  

```bash
srun --pty --account=project_2006567 --partition=serial --cpus-per-task=40 --mem=10000 --time=12:00:00 bash
```

> **Dica:**  
> 
> Porque estamos solicitando mais recursos, e dependendo das outras tarefas que estão rodando no servidor no momento, talvez demore um pouco mais de tempo para conseguirmos acesso ao nó de computação.  
> Se enquanto espera você quer levantar, dar uma volta, pegar um café, ou apenas fazer outra coisa no computador, você pode adicionar ao comando `srun` a bandeira `--mail-user=seu_email@seu_provedor.com.br`.  
> Assim, você receberá um e-mail quando lhe for dado acesso ao nó de computação.  
> Mas preste atenção para o seu computador não entrar no modo hibernar/dormir e não saia da internet, senão você perderá a conexão com o servidor e terá que começar novamente. 

Quando estivermos então conectados ao nó de computação, devemos ativar o ambiente `conda` onde `MEGAHIT` está instalado e criar uma pasta onde as montagens serão armazenadas:  

```bash
conda activate megahit
mkdir montagens
```

Agora vamos rodar `MEGAHIT` para as 4 amostras, novamente utilizando um `for loop`:  

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

> **ATENÇÃO:** 
> 
> No modo interativo que estamos utilizando, se o seu computador entrar no modo hibernar/dormir ou perder acesso à internet você perderá a conexão com o servidor e o comando que estiver rodando será cancelado.  
> Uma maneira de evitar isso é utilizando o programa `screen`, que efetivamente abre uma nova sessão do terminal no *background* que continua rodando mesmo que você desligue o computador.    
> Se quiser tentar, dê uma olhada nas instruções [aqui](https://kb.iu.edu/d/acuy).  
> Ou se estiver afim de um desafio, porque não abandonar o modo **interativo** e rodar os programas no modo de **lote** (*batch*)?  
> Você pode ler mais sobre o modo de lote [aqui](https://docs.csc.fi/computing/running/creating-job-scripts-puhti/).  


Enquanto espera, dê uma olhada no manual do `MEGAHIT` [aqui](https://www.metagenomics.wiki/tools/assembly/megahit) e [aqui](https://github.com/voutcn/megahit/wiki) e veja se há algo que você poderia ter feito diferente.  
Por exemplo, quais são os valores padrão para `--k-min` e `--k-max` que foram utilizados?  
No caso das nossas amostras, seria melhor ter modificado esses valores?  

> **Lembrete:** 
> 
> Quando o `MEGAHIT` tiver terminado de rodar, não se esqueça de desconectar do nó de computação com o comando `exit`.
