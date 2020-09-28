# Tutorial 3

## Apresentação do problema

Agora que sabemos coletar de uma página da busca atributos e os conteúdos de uma "tag", precisamos repetir o procedimento para todas as páginas de resultado. O objetivo é aplicar os conceitos do Tutorial 1, mas com as novas funções que vimos ao longo do Tutorial 2. Para tanto, vamos usar um "for loop" do Tutorial 1 para ir de uma página a outra.

O primeiro passo é, mais uma vez, ter o nosso link da pesquisa que queremos coletar armazenado em um objeto. Aqui selecionaremos apenas os resultados da Edição Impressa para limitar o número de páginas que capturaremos para fins didáticos

```{r}
url_base <- "https://search.folha.uol.com.br/search?q=covid&site=jornal&ed=15%2F10%2F2015&periodo=todos&results_count=461&search_time=0%2C704&url=http%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Dcovid%26site%3Djornal%26periodo%3Dtodos&sr="
```

## Função Paste

Como é possível reparar, o número da página fica ao final do link, por isso podemos utilizar a função chamada _paste0_ ou "colar" ao invés da função _gsub_.

Na linguagem do R, escreveremos assim para o nosso caso:

```{r}
i <- 1
url_pesquisa <- paste(url_base, i, sep = "")
```

A "URL" é o endereço da página de busca, o "i" é o contador numérico do loop e o último argumento refere-se ao separador que igualamos a um par de aspas sem nada dentro, deixando claro que para funcionar corretamente nada, nem uma barra de espaço, deve ficar entre o endereço e o contador.

## Coletando o conteúdo e o atributo de todos os links

A lógica de coleta do atributo e do conteúdo de um node continua o mesma. A única diferença é que precisamos aplicar isso para todas as páginas. Agora que temos a URL construída, podemos montar um "for loop" que fará isso para nós.

Antes de prosseguir, vamos observar o URL da página de busca (poderíamos buscar termos chave, mas, neste caso, vamos pegar todas as notícias relacionadas a eleições). Na página 2 da busca vemos que o final é "sr=26". Na página 3 o final é "sr=51". Há um padrão: as buscas são realizadas de 25 em 25. De fato, a última página da busca é a 19. Para "passarmos" de página em página, portanto, temos que ter um "loop" que conte não mais de 1 até 361, mas na seguinte sequência numérica: {1, 26, 51, 76, ..., 426, 451}.

Precisamos, então, que "i" seja recalculado dentro do loop para coincidir com a numeração da primeira notícia de cada página. Parece difícil, mas é extremamente simples. Veja o loop abaixo, que imprime a sequência desejada multiplicando (i - 1) por 25 e somando 1 ao final:

Obs: a primeira página possui um link repetido que não tem o mesmo texto, por isso começaremos na segunda.

```{r}
for (i in 1:19){
  i <- (i - 1) * 25 + 1
  print(i)
}
```

O que precisamos agora é incluir nas "instruções do loop" o que foi discutido no tutorial 2. 

Em primeiro lugar, construímos o url de cada página do resultado da busca:

```{r}
url_pesquisa <- paste(url_base, i, sep = "")
```

A seguir, capturamos o código HTML da página após chamar o pacote *rvest*:

```{r}
library(rvest)
pagina <- read_html(url_pesquisa)
```

Escolhemos apenas os "nodes" que nos interessam:

```{r}
nodes_titulos <- html_nodes(pagina, xpath = "//ol/li/div/div/a/h2[@class = 'c-headline__title']")
nodes_links <- html_nodes(pagina, xpath = "//ol/li/div/div/a")
```

Extraímos os títulos e os links com as funções apropriadas:

```{r}
titulos <- html_text(nodes_titulos)
links <- html_attr(nodes_links, name = "href")
```

Fazemos os ajustes realizados no Tutorial 2 para eliminar os caracteres extras e manter somente valores únicos dos links:

```{r}
titulos <- gsub("\n                                                                                                                ", "", titulos)
links <- unique(links)
```

Combinamos os dois vetores em um data frame:

```{r}
tabela_titulos <- data.frame(titulos, links)
```

Falta "empilhar" o que produziremos em cada iteração do loop de uma forma que facilite a visualização. Criamos um objeto vazio antes do loop. 

Usaremos a função _bind\_rows_ (ou _rbind_ se estiver com problemas com o _dplyr_) para combinar data frames. A cada página agora, teremos 11 resultados em uma tabela com duas variáveis. O que queremos é a junção dele com os 25 resultados de cada uma das outas 18 páginas. Vamos também chamar a biblioteca _dplyr_ para usar sua função _bind\_rows_.

```{r}
library(dplyr)
dados_pesquisa <- data.frame()
dados_pesquisa <- bind_rows(dados_pesquisa, tabela_titulos)
```

Chegou o momento de colocar dentro loop tudo o que queremos que execute em cada uma das vezes que ele ocorrer. Ou seja, que imprima na tela a página que está executando, que a URL da página de resultados seja construída com a função paste, para todas elas o código HTML seja examinado, lido no R e transformado em objeto XML, colete todos os links e todos os títulos e que "empilhe". Lembrando que não podemos esquecer de definir a URL que estamos usando e criar um data frame vazio para colocar todos os links e títulos coletados antes de iniciar o loop.

Obs: a primeira página possui um link repetido que não tem o mesmo texto, por isso começaremos na segunda.

```{r}
url_base <- "https://search.folha.uol.com.br/search?q=covid&site=jornal&ed=15%2F10%2F2015&periodo=todos&results_count=461&search_time=0%2C704&url=http%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Dcovid%26site%3Djornal%26periodo%3Dtodos&sr="

dados_pesquisa <- data_frame()

for (i in 1:19){
  
  print(i)

  i <- (i - 1) * 25 + 1
  
  url_pesquisa <- paste(url_base, i, sep = "")
  
  pagina <- read_html(url_pesquisa)
  
  nodes_titulos <- html_nodes(pagina, xpath = "//ol/li/div/div/a/h2[@class = 'c-headline__title']")
  nodes_links <- html_nodes(pagina, xpath = "//ol/li/div/div/a")
  
  titulos <- html_text(nodes_titulos)
  titulos <- gsub("\n                                                                                                                ", "", titulos)

  links <- html_attr(nodes_links, name = "href")
  links <- unique(links)
  
  tabela_titulos <- data.frame(titulos, links)
  
  dados_pesquisa <- bind_rows(dados_pesquisa, tabela_titulos)

}
```

Pronto! Temos agora todos os títulos e links de todos os resultados do site da Folha de São Paulo nas edições impressas para a palavra "covid" em um único banco de dados.
