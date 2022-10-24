# Prática de linha de comando

A maioria das nossas atividades será feita através da linha de comando do sistema Unix.  
Portanto, é recomendável ter pelo menos uma noção básica de como se orientar usando a linha de comando.  
Dedicaremos agora cerca de uma hora para (re-)aprender algumas noções básicas.  

**Usuários de Windows:** abra [este emulador de terminal](https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) em uma nova janela.  
**Usuários de MacOS/Linux:** inicie o aplicativo Terminal.  

> **NOTAS:** 
> 
> Coisas dentro de uma caixa como esta...
> 
> ```bash
> mkdir exercicio_unix
> cd exercicio_unix
> ```
> ...representa comandos que você deve digitar.  
> Cada linha representa um comando.  
> Comandos devem ser digitados em uma única linha, um de cada vez.  
> Após cada comando, tecle Enter para executar.  
> 
> Linhas que começam com o símbolo do jogo da velha (*hashtag*)...
> 
> ```bash
> # Isto é um comentário e será ignorado pelo sistema
> ```
> 
> ...representam comentários embutidos no código para dar instruções ao usuário, fazer observações sobre o código, etc.  
> Todo o texto em uma linha que comece com `#` é ignorada e não é executada.  
> 
> Ao longo desse curso nós usaremos comandos diferentes que utilizam sintaxes diferentes.  
> Comandos diferentes esperam diferentes tipos de bandeiras e argumentos.  
> Algumas vezes a ordem importa, outras não.  
> Alguns parâmetros possuem um valor padrão e não precisam ser declarados pelo usuário, a menos que queira modificar o valor padrão.  
> Se você não tiver certeza como rodar um comando, a melhor maneira de aprender mais é dando uma olhada no manual com o comando `man`.  
> Por exemplo, se você quiser ver o manual do comando `mkdir`, você pode executar:
> 
> ```bash
> man mkdir
> 
> # Aperte a barra de espaço ou ctrl+f para avançar o manual
> # Aperte ctr+b para voltar no manual
> # Aperte "q" para sair
> ```

## Criando e navegando pastas

Primeiro vamos ver onde estamos localizados dentro do sistema:

```bash
pwd
```

Existe algum arquivo aqui? Vamos listar o conteúdo da pasta onde estamos:

```bash
ls
```

Vamos agora criar uma nova pasta chamada `exercicio_unix`.  
Além do comando (`mkdir`), estamos passando agora um argumento que, neste caso, é o nome da pasta que queremos criar:

```bash
mkdir exercicio_unix
```

Alguma coisa mudou? Como listar o conteúdo da pasta novamente?

<details>
<summary>
DICA (CLIQUE PARA EXPANDIR)
</summary>

> ls

</details>  

Agora vamos entrar na pasta `exercicio_unix`:

```bash
cd exercicio_unix
```

Funcionou? Como saber onde estamos agora?

<details>
<summary>
DICA
</summary>

> pwd

</details>  

## Criando um arquivo

Vamos criar um novo arquivo chamado `meu_arquivo.txt` lançando o editor de texto `nano`:  

```bash
nano meu_arquivo.txt
```

Agora dentro da tela do `nano`:  

1. Escreva algo
2. Aperte `ctrl+x` para sair
3. Para salvar as modificações, escreva `yes` e depois tecle `Enter`
4. Confirme o nome do arquivo e tecle `Enter`

Agora liste o conteúdo da pasta.  
Você consegue ver o arquivo que acabamos de criar?  

## Copiando, renomeando, movendo e excluindo arquivos

Primeiro vamos criar uma nova pasta chamada `minha_pasta`.  
Você se lembra como fazer isso?

<details>
<summary>
DICA
</summary>

> mkdir minha_pasta

</details>  

E agora vamos fazer uma cópia de `meu_arquivo.txt`.  
Aqui, o comando `cp` precisa de dois argumentos, e a ordem desses argumentos é importante.  
O primeiro é o nome do arquivo que queremos copiar e o segundo é o nome do novo arquivo:  

```bash
cp meu_arquivo.txt novo_arquivo.txt
```

Liste o conteúdo da pasta.  
Você vê o novo arquivo lá?

Agora digamos que queremos copiar um arquivo e colocá-lo dentro de uma pasta.  
Neste caso, damos o nome da pasta como segundo argumento para `cp`:

```bash
cp meu_arquivo.txt minha_pasta
```

Liste o conteúdo de `minha_pasta`.  
O `meu_arquivo.txt` está lá?

```bash
ls minha_pasta
```

Também podemos copiar o arquivo para outra pasta e dar um nome diferente:

```bash
cp meu_arquivo.txt minha_pasta/copia_de_meu_arquivo.txt
```

Liste o conteúdo de `minha_pasta` novamente.  
Quantos arquivos têm lá dentro agora?  

Em vez de copiar, podemos mover arquivos com o comando `mv`:

```bash
mv novo_arquivo.txt minha_pasta
```

Agora liste os conteúdos da pasta atual e também de `minha_pasta`.  
Para onde `novo_arquivo.txt` foi?  

Também podemos usar o comando `mv` para renomear arquivos:  

```bash
mv meu_arquivo.txt meu_novo_arquivo.txt
```

Agora liste os conteúdos da pasta novamente.  
O que aconteceu com `meu_arquivo.txt`?

Agora, digamos que queremos mover coisas de dentro de `minha_pasta` para o diretório atual.  
Qual o propósito do ponto (`.`) no comando abaixo?  
Vamos tentar:

```bash
mv minha_pasta/novo_arquivo.txt .
```
Agora liste os conteúdos da pasta atual e também de `minha_pasta`.  
O arquivo `novo_arquivo.txt` estava dentro do `minha_pasta` antes, onde ele está agora?  

A mesma operação pode ser feita de forma diferente.  
Qual o propósito dos dois pontos (`..`) no comando abaixo?  
Vamos tentar:  

```bash
# Primeiro entramos na pasta
cd minha_pasta

# Então movemos o arquivo um nível acima
mv meu_arquivo.txt ..

# E então voltamos um nível
cd ..
```

Agora liste os conteúdos da pasta atual e também de `minha_pasta`.  
O arquivo `meu_arquivo.txt` estava dentro do `minha_pasta` antes, onde ele está agora?  

Neste exercício acabamos criando diversas cópias do mesmo arquivo.  
Vamos dar uma limpada e deletar alguns arquivos:  

```bash
rm novo_arquivo.txt
```

Vamos listar o conteúdo da pasta.  
O que aconteceu com `novo_arquivo.txt`?  

Ao excluir arquivos, preste atenção no que você está fazendo:  
**Se você remover acidentalmente o arquivo errado, ele desaparecerá para sempre!**

E agora vamos deletar `minha_pasta`:

```bash
rm minha_pasta
```

Ops, algo deu errado não?  
Obtivemos uma mensagem de erro, o que ela significa?  

Para excluir uma pasta, temos que modificar o comando adicionando a bandeira (`-r`).  
Bandeiras são usadas para passar opções adicionais aos comandos:  

```bash
rm -r minha_pasta
```

Vamos listar o conteúdo da pasta.  
O que aconteceu com `minha_pasta`?

## Para aprender mais

O exercício acima abordou uma ínfima parte do que precisamos saber para navegar a linha de comando do Unix.  
Dependendo do seu nível de conhecimento, seria interessante praticar mais para poder fazer esse curso de uma maneira mas fluida.  

Abaixo listo alguns recursos que achei úteis para melhorar ou refrescar o conhecimento da linha de comando:

* [cmdchallenge.com](https://cmdchallenge.com)
* [astrobiomike.github.io/unix](https://astrobiomike.github.io/unix)
* [youtube.com/watch?v=IrDUcdpPmdI](https://www.youtube.com/watch?v=IrDUcdpPmdI)