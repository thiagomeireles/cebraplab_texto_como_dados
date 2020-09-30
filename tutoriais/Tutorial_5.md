# Tutorial 5 - Análise de texto no R - pacote _stringr_

### Webscrapping para capturar material para o tutorial

Nossa primeira tarefa será obter um conjunto de textos com o qual trabalharemos. Classicamente, tutoriais de R sobre strings e mineração de texto utilizam "corpus" (já veremos o que é isso) de literatura moderna.

Para tornar nosso exemplo mais interessante, vamos recuperar as notícias do DataFolha que utilizamos no Tutorial 4. A primeira ferramenta que veremos neste tutorial é o pacote _stringr_. Além de carregarmos o pacote _strigr_, também utilizaremos os pacotes _rvest_ e _dplyr_:

```{r}
library(dplyr)
library(rvest)
library(stringr)
```

Feito isso, vamos raspar os dados das notícias relacionadas às eleições no site do DataFolha.

```{r}
url_base <- "http://search.folha.uol.com.br/search?q=eleicoes&site=datafolha&skin=datafolha&results_count=669&search_time=1%2C067&url=http%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Deleicoes%26site%3Ddatafolha%26skin%3Ddatafolha&sr="

dados_pesquisa <- tibble()

for (i in 1:27){
  
  print(i)
  
  i <- (i - 1) * 25 + 1
  
  url_pesquisa <- paste(url_base, i, sep = "")
  
  pagina <- read_html(url_pesquisa)
  
  nodes_titulos <- html_nodes(pagina, xpath = "//h2/a")
  
  titulos <- html_text(nodes_titulos)
  links <- html_attr(nodes_titulos, name = "href")
  
  tabela_titulos <- tibble(titulos, links)
  
  dados_pesquisa <- bind_rows(dados_pesquisa, tabela_titulos)
}

dados_noticias <- tibble()

for (link in dados_pesquisa$links){
  
  print(link)
  
  pagina <- try(read_html(link))
  
  if (!(class(pagina) %in% "try-error")){
    node_titulo <- html_nodes(pagina, xpath = "//h1[@class = 'main_color main_title']")
    titulo <- html_text(node_titulo)

    node_datahora <- html_nodes(pagina, xpath = "//time")
    datahora <- html_text(node_datahora)
    
    node_texto <- html_nodes(pagina, xpath = "//article[@class = 'news']/p")
    texto <- html_text(node_texto)
    texto <- paste(texto, collapse = " ")
    
    tabela_noticia <- tibble(titulo, datahora, link, texto)
    
    dados_noticias <- bind_rows(dados_noticias, tabela_noticia)
  } 
  
}
```

Os resultados têm dois problemas: temos o "_\n_" que não foi identificado pelo _encoding_ e no final de todos os textos das notícias foi capturada a opção de "Baixa a pesquisa completa".

Aproveitemos para ver duas funções novas, ambas do pacote _stringr_. Várias delas, como veremos, são semelhantes a funções de outros pacotes com as quais já trabalhamos. Há, porém, algumas vantagens ao utilizá-las: bugs e comportamentos inesperados corrigidos, uso do operador "pipe", nomes intuitivos e sequência de argumentos intuitivos.

_str\_replace\_all_ substitui no texto um padrão por outro, respectivamente na sequência de argumentos. Seu uso é semelhante à função _gsub_, mas os argumentos estão em ordem intuitiva. Por exemplo, estamos substituindo espaço por nada nos url:

```{r}
noticias <- dados_noticias$texto

noticias <- str_replace_all(noticias, "\n", "")
noticias <- str_replace_all(noticias, "Baixa a pesquisa completa", "")
```


### Funcionalidades do _stringr_

Qual é o tamanho de cada discurso? Vamos aplicar _str\_length_ para descobrir. Seu uso é semelhante ao da função _nchar_:

```{r}
len_noticias <- str_length(noticias)
len_noticias
```

Vamos agora observar quais são os noticias que mencionam "Lula" e "Bolsonaro". Para tanto, usamos _str\_detect_

```{r}
str_detect(noticias, "Lula")
str_detect(noticias, "Bolsonaro")
```

Poderíamos usar o vetor lógico resultante para gerar um subconjunto dos noticias, apenas com aqueles nos quais as palavras "Lula" e "Bolsonaro" são mencionadas. Mais simples, porém, é utilizar a função _str\_subset_, que funciona tal qual _str\_detect_, mas resulta num subconjunto em lugar de um vetor lógico:

```{r}
noticias_lula      <- str_subset(noticias, "Lula")
noticias_bolsonaro <- str_subset(noticias, "Bolsonaro")
```

Se quisessemos apenas a posição no vetor dos noticias que contêm "Lula", _str\_which_ faria o trabalho:

```{r}
str_which(noticias, "Lula")
str_which(noticias, "Bolsonaro")
```

Voltando ao vetor completo, quantas vezes "Lula" é mencionada em cada noticias? E Bolsonaro? Qual é o máximo de menções a "Lula" em um único discurso? E Bolsonaro

```{r}
str_count(noticias, "Lula")
max(str_count(noticias, "Lula"))
str_count(noticias, "Bolsonaro")
max(str_count(noticias, "Bolsonaro"))
```

Vamos fazer uma substituição nos noticias. No lugar de "Lula" colocaremos a expressão "Lula, guerreiro do povo brasileiro,". E no lugar de "Bolsonaro", "Bolsonaro, Brasil acima de tudo e Deus acima de todos," Podemos fazer a substituição com _str\_replace_ ou com _str\_replace\_all_. A diferença entre ambas é que _str\_replace_ substitui apenas a primeira ocorrênca encontrada, enquanto _str\_replace\_all_ substitui todas as ocorrências.

```{r}
str_replace(noticias_lula, "Lula", "Lula, guerreiro do povo brasileiro,")
str_replace_all(noticias_lula, "Lula", "Lula, guerreiro do povo brasileiro,")

str_replace(noticias_bolsonaro, "Bolsonaro", "Bolsonaro, Brasil acima de tudo e Deus acima de todos,")
str_replace_all(noticias_bolsonaro, "Bolsonaro", "Bolsonaro, Brasil acima de tudo e Deus acima de todos,")
```

Em vez de substituir, queremos conhecer a posição das ocorrências de "Lula" e de Bolsonaro. Com _str\_locate_ e _str\_locate\_all_, respectivamente para a primeira ocorrência e todas as ocorrências, obtemos a posição de começo e fim do padrão buscado:

```{r}
str_locate(noticias_lula, "Lula")
str_locate_all(noticias_lula, "Lula")

str_locate(noticias_bolsonaro, "Bolsonaro")
str_locate_all(noticias_bolsonaro, "Bolsonaro")
```

Finalmente, notemos que os noticias começam sempre mais ou menos da mesma forma. Vamos retirar os 100 primeiros caracteres de cada discurso para observá-los. Usamos a função _str\_sub_, semelhante à função _substr_, para extrair um padaço de uma string:

```{r}
str_sub(noticias, 1, 100)
```

As posições para extração de exerto podem ser variáveis. Por exemplo, vamos usar "len_noticias" que criamos acima para extrair os 50 últimos caracteres de cada discurso:

```{r}
str_sub(noticias, (len_noticias - 50), len_noticias)
```

Note que alguns noticias começam e terminam com espaços. Para nos livrarmos deles (apenas daqueles no começo e fim da string), utilizamos _str\_trim_:

```{r}
str_trim(noticias)
```

Infelizmente, não há tempo suficiente para entrarmos neste tutorial em um tema extremamante útil: expressões regulares. Expressões regulares, como podemos deduzir pelo nome, são expressões que nos permite localizar -- e, portanto, substituir, extrair, parear, etc -- sequências de caracteres com determinadas caraterísticas -- por exemplo, "quaisquer caracteres entre parênteses", ou "qualquer sequência entre espaços que comece com 3 letras e termine com 4 números" (placa de automóvel).

Você pode ler um pouco sobre expressões regulares no R [aqui](https://rstudio-pubs-static.s3.amazonaws.com/74603_76cd14d5983f47408fdf0b323550b846.html) em aula se terminar os três tutoriais de hoje. Com o uso de expressões regulares, outros dois pares de funções são bastante úteis _str\_extract_, _str\_extract\_all_, _str\_match_ e _str\_match\_all_.

## Nuvem de Palavras

Com a função _wordcloud_ do pacote de mesmo nome, podemos rapidamente visualizar as palavras discursadas tendo o tamanho como função da frequência (vamos limitar a 50 palavras):

```{r}
# install.packages(c("slam", "tm", "wordcloud")) 
# Retirem o # para instalar

library(wordcloud)
wordcloud(noticias, max.words = 50)
wordcloud(noticias_bolsonaro, max.words = 50)
wordcloud(noticias_lula, max.words = 50)
```

Não muito bonitas. Voltaremos a fazer nuvem de palavras depois de aprendermos outra maneiras de trabalharmos com texto como dado no R.
