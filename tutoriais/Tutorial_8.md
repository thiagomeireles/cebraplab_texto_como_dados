# Tutorial 8: O pacote *quanteda*

## Instalando os pacotes

Hoje trabalharemos a partir do pacote *quanteda*. Nas instalações hoje teremos algumas diferenças nas instalações de pacotes, uma vez que nem todos estão disponíveis como o *quanteda* no repositório [CRAN](https://CRAN.R-project.org/package=quanteda), o que vimos nos outros exemplos de pacotes.

```{r, eval = FALSE}
install.packages("quanteda")
```
Além das versões mais estáveis, é possível baixar versões de desenvolvedor dos pacotes, as [versões de GitHub](https://github.com/quanteda/quanteda). Recomendo que mantenham os pacotes do CRAN quando disponíveis, elas tendem a ser mais estáveis.

Além do pacote, seus desenvolvedores sugerem a instalação de alguns pacotes adicionais. Os dois primeiros também estão no CRAN.

O pacote [*readtext*](https://github.com/quanteda/readtext) é uma alternativa amigável para leitura de textos em R proveniente de quase qualquer formato. 

Já o [*spacyr*](https://github.com/quanteda/spacyr) é um pacote de PNL (Processamento Natural de Linguagem) que contem marcação de classe gramatical, reconhecimento de entidade e análise de dependência.

```{r, eval = FALSE}
install.packages(c("readtext", "spacyr"))
```

Já os outros dois são pacotes adicionais ao *quanteda* e não estão no repositório CRAN. Nesse caso, utilizamos o pacote [*devtools*](https://cran.r-project.org/web/packages/devtools/devtools.pdf) (pacote utilizado para desenvolvimento de pacotes). No entanto, não carregaremos todas suas funções, utilizaremos uma abordagem diferente aqui chamando diretamente a função especificando de qual pacote ela faz parte. No caso dos pacotes de hoje, eles estão em repositórios do GitHub.

O primeiro é o [*quanteda.corpora*](https://github.com/quanteda/quanteda.corpora) que fornece dados em texto para uso com o *quanteda*

```{r eval = FALSE}
devtools::install_github("quanteda/quanteda.corpora")
```

Já o segundo é o [*quanteda.dictionaries*](https://github.com/kbenoit/quanteda.dictionaries) que possui vários dicionários para utilizar com o *quanteda*. 

```{r eval = FALSE}
devtools::install_github("kbenoit/quanteda.dictionaries")
```

Espero que tenham percebido a lógica da instalação. Dizemos que queremos que o *devtools* utilize(::) a função *install_github*. Aqui é necessário especificar o usuário/desenvolvedor/repositório do pacote (kbenoit, usuário do Kenneth Benoit que é o diretor do projeto quanteda).

## Criando um corpus

```{r, message = FALSE}
library("quanteda")
```

### Opções de fontes de corpus

O *quanteda* ao utilizar a dependência *readtext* tem um pacote bastante eficiente para leitura (fácil) de textos. A principal função do pacote possui o nome do pacote `readtext()`. Com ela é possível importar arquivos do disco ou da internet e gerando um tipo de dataframe que pode ser usado para construção de um objeto corpus com a função `corpus()` do *quanteda*

A função `readtext()` aceita arquivos em texto (`.txt`), valores separados por vírgula (`.csv`), dados formatados em XML ou dados JSON genéricos ou de APIs específicas (sobre dados em JSON, não tratamos no curso mas existem diversos tutoriais disponíveis como [este](https://www.tutorialspoint.com/r/r_json_files.htm)).

Já a função `corpus()` trabalha com vetores character carregados anteriormente no ambiente de trabalho por outras ferramentas, com os objetos `VCorpus` que vimos no pacote *tm* e dataframes que contenham uma coluna de texto ou metadados.

### Criando um corpus a partir de um vetor

Vamos utilizar os dados do Datafolha que coletamos nos tutoriais anteriores. Assim, o código do próximo chunk carrega os pacotes, organiza e baixa os textos e ajusta a formatação.

```{r, message=FALSE, warning=FALSE}
library(dplyr)
library(rvest)
library(stringr)
library(tidyr)

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

# Aqui separamos o vetor datahora em dois com a função `separate()` do tidyr e convertemos a data em uma variável de tipo Data, extraindo o ano para filtrar os dados mais à frente
dados_noticias <- dados_noticias %>% 
  separate(datahora, c("data", "hora"), sep = " ") %>% 
  mutate(data = as.Date(data, "%d/%m/%Y"),
         ano = format(data, format = "%Y"))

noticias <- dados_noticias$texto

noticias <- str_replace_all(noticias, "\n", "")
noticias <- str_replace_all(noticias, "Baixa a pesquisa completa", "")
```

A forma mais simples de criação de um corpus é a que trataremos neste tutorial. Utilizaremos um vetor que já está na memória do R. Para isso, basta aplicarmos a função `corpus()`ao vetor de interesse.

```{r}
corpus_noticias <- corpus(noticias)
summary(corpus_noticias)
```

Veja como é interessante ao pedirmos o resumo do corpus com a função `summary()`. Temos as informações básicas do que cada texto contém.

Existe, também, a possibilidade de adicionarmos variáveis relacinadas a cada um dos documentos ao corpus, como tínhamos no dataframe de onde extraímos o vetor. Para isso, utilizamos a função `docvars()` do pacote *quanteda*. Acrescentaremos ao nosso corpus o título da notícia, a data e o ano da publicação.

```{r}
docvars(corpus_noticias, "titulo") <- dados_noticias$titulo
docvars(corpus_noticias, "data")   <- dados_noticias$data
docvars(corpus_noticias, "ano")    <- dados_noticias$ano
summary(corpus_noticias)
```


## Como um corpus do *quanteda* funciona?

### Princípios de um corpus

No último tutorial passamos rapidamente pela construção de corpus sem destacar o que eles são. Por definição, um corpus é uma "biblioteca" do documento original que foi convertido para um formato mais simples, codificado em UTF-8 (lembram dos encodings?) e armazenado com seus metadados no nível do corpus e do documento (as notícias no nosso caso). O que aplicamos com a função `docvars()` é justamente a aplicação dos metadados no nível do documento. Essas variáveis nos dão informações que descrevem o atributo de cada documento.

A vantagem da utilização de corpus é que eles são projetados para ser mais ou conjunto mais ou menos estático de textos com relação ao seu processamento e à sua análise. Mas o que isso quer dizer? De uma forma mais simples, os textos dos corpus não são projetados para sofrerem alterações internas, como as etapas de limpeza, remoção de pontuação ou de *stopwords*. Na verdade, utilizamos o corpus para extrair textos como parte do processamento e os atribuindo a novos objetos. A ideia aqui é que o corpus se mantenha como uma referência original para outros tipos de análise, como as que os radicais e a pontuação sejam necessárias. Com isso, podemos partir diretamente de um mesmo corpus.

Assim, é importante que saibamos extrair o texto de um corpus, não? Para isso utilizamos simplesmente a função `texts()`, como no exemplo abaixo em que pegaremos a segunda notícia do nosso corpus.

```{r}
texts(corpus_noticias)[2]
```

Podemos, também, utilizar a função `summary()`para resumir as informações dos textos de um corpus, inclusive definindo quantos desses textos queremos entender. No exemplo abaixo, pediremos que resuma apenas 8.

```{r}
summary(corpus_noticias, n = 8)
```

Você deve ter observado que na primeira vez que aplicamos o `summary()`para todo o corpus, ele apresentou somente os 100 primeiros resultados. Esse é o padrão aplicado à função. Se quisermos que ele o faça para todos os textos, especificamos em `n` isso com a função `lenght()`. Assim, podemos salvar um novo objeto com o resumo de todo o corpus e, por exemplo, fazer  rapidamente um gráfico descritivo com essas informações.


```{r, fig.width = 8}
tokeninfo <- summary(corpus_noticias, n = length(corpus_noticias))

library(ggplot2)

tokeninfo %>% 
  group_by(ano) %>% 
  summarise(tokens_ano = sum(Tokens),
            noticias_ano = n()) %>% 
  ungroup() %>% 
  mutate(tokens_notica = tokens_ano/noticias_ano) %>% 
  ggplot(aes(x = ano, y = tokens_notica, group = 1)) +
  geom_line() +
  geom_point() +
  xlab("Ano") + ylab("Tokens por notícia") +
  theme_bw()
```

Explicando rapidamente os procedimentos com o *dplyr* aplicados antes do gráfico: agrupamos as notícias por ano com a função `group_by()`; resumimos as informações do total de tokens com `sum()` e do número de notícias com `n()` dentro da função `summarise()`; desagrupamos com o `ungroup()` (isso é altamente recomendável para continuar manipulando os dados depois de um `group_by()`); e, por fim, criamos uma variável com o número de tokens por notícia com o `mutate()`. Vocês perceberam que não criamos um novo objeto para isso e iniciamos o `ggplot()` após um *pipe*, o que indica que o argumento *data* da função reconhecerá automaticamente toda manipulação que fizemos anteriormente. Legal, não?

Podemos ver, por exemplo, qual é a notícia com maior texto. Para isso, utilizaremos a função `which.max()`.

```{r}
tokeninfo[which.max(tokeninfo$Tokens), ]
```

Encontramos que a notícia "Católicos divergem da igreja em relação a divórcio, segundo casamento e uso de camisinha" de 2007 é a matéria mais longa de todas as obtidas no DataFolha.

## Ferramentas para objetos no formato corpus

### Adicionando dois objetos corpus em um só

Com um simples operador de adição, `+`, conseguimos concatenar dois ojetos corpos. Caso possuam conjuntos de variáveis diferentes no nível do documento, serão agrupados de forma que nenhuma informação seja perda. Os metadados no nível do corpus também são concatenados.

```{r}
corpus1 <- corpus(corpus_noticias[1:5])
corpus2 <- corpus(corpus_noticias[53:58])
corpus3 <- corpus1 + corpus2
summary(corpus3)
```

### Subconjuntos de corpus

Podemos extair subconjuntos do nosso corpus aplicando a função `corpus_subset()` que criará um novo corpus baseao em alguma condição lógica que aplicarmos às *docvars*. Podemos, por exemplo, pegar somente notícias a partir de 2018 ou antes de 2000.

```{r}
summary(corpus_subset(corpus_noticias, ano >= 2018))
summary(corpus_subset(corpus_noticias, ano < 2000))
```

## Explorando os textos do corpus

A função `kwic()` (keywords-in-context) realiza uma busca por uma palavra e nos pedimete ver os contextos em que ela aparece.

The `kwic` function (keywords-in-context) performs a search for a word and allows us to view the contexts in which it occurs:

```{r}
kwic(corpus_noticias, pattern = "vitória")
```

```{r}
kwic(corpus_noticias, pattern = "vitória", valuetype = "regex")
```

```{r}
kwic(corpus_noticias, pattern = "comunista")
```

Podemos aplicar a função para expressões com mais de uma palavra com a função `phrase()`.

```{r}
kwic(corpus_noticias, pattern = phrase("intenção de voto para prefeito")) %>%
    head() # ao aplicar a função, veremos somente as 6 primeiras ocorrências
```

Na função `summary()` são paresentadas as variáveis associadas a cada documento - "título", "data" e "ano". Podemos visualizar somente elas utilizando a função `docvars()`.

```{r}
# inspect the document-level variables
head(docvars(corpus_noticias))
```

Caso queiram utilizar outros exemplos de corpus para manipulação, os desenvolvedores do *quanteda* oferecem algumas opções com o pacote [*quanteda.corpora*](https://github.com/quanteda/quanteda.corpora).

## Extraindo recursos de um corpus 

Como vimos acima, o corpus é um objeto do qual obteremos o que precisamos para realizar nossas análise. Por exemplo, antes de realizar qualquer análise, devemos extrair uma matriz associando valores para certos recursos com cada documento. O *quanteda* possui a função `dfm()` para produzir essas matrizes. A expressão *dfm* significa *document-feature matrix* e sempre se refere aos documentos nas linhas e aos "recursos" nas colunas. A opção por recuros a termos ocorre porque recusos são mais gerais que os termos

Nós os chamamos de "recursos" em vez de termos, porque os recursos são mais gerais do que os termos, ou seja, podem ser definidos como termos brutos (sem alterações), termos derivados, classes gramaticais de termos, termos após remoção de palavras, etc. Assim, os recursos podem ser tanto gerais como específicos, como os n-gramas que vimos na aula passada. 

## Tokenização de textos

Como vimos na última aula, a tokenização é uma parte importante na análise quantitativa de textos. O pacote *quanteda* oferece uma função bastante poderosa e simples chamada `tokens()`. Com ela, produzimos um objeto intermediário que consiste em uma lista de tokens na forma de vetores *character*, na qual cada elemento da lista corresponde a um documento de entrada. É importante salientar que a função `tokens()`em sua configuração padrão é explicitamente conservadora, ou seja, não remove nada do texto a menos que seja indicado. A única remoção padrão é a dos separadores (pontos, vírgulas, etc.)
```{r}
tokens(noticias)                                             # tokenização padrão
tokens(noticias, remove_numbers = TRUE,  remove_punct = TRUE)  # tokenização removendo números e pontuação
tokens(noticias, remove_numbers = FALSE, remove_punct = TRUE)  # tokenização removendo pontuação
tokens(noticias, remove_numbers = TRUE,  remove_punct = FALSE) # tokenização removendo os números
tokens(noticias, remove_numbers = FALSE, remove_punct = FALSE) # tokenização sem remoção de números e pontuação, ou seja, a padrão
```

Também temos a opção de tokenizar as letras, onde temos uma nova opção chamada `remove_separators` para retirar os espaços entre as palavras da tokenização. A opção padrão é `TRUE`.

```{r}
tokens(noticias[1], what = "character")
tokens(noticias[1], what = "character",
         remove_separators = FALSE)
```

Também podemos tokenizar frases com a opção "sentence" em `what`.

```{r}
tokens(noticias[1], what = "sentence")
```

Ainda temos a opção de concatenar expressões e mantê-las como um único recurso para a análise com a função `tokens_compound()`.
```{r}
tokens(noticias[1]) %>% 
  tokens_compound(pattern = phrase("São Paulo"))
```


## Construindo uma matriz documento-recurso

Como dissemos acima, a tokenização de textos é uma etapa intermediária e muitos podem querer simplesmente ir diretamente para a construção de uma matriz documento-recurso. Para isso, o *quanteda* oferece a citada função `dfm()` que realiza a tokenização e tabula os recuros extraídos em uma matriz de documentos por recursos. Os desenvolvedores do pacote classificam a função como um "canivete suíço" pela diversidade de ações que podemos realizar com ela. 

Diferentemente da abordagem conservativa da função `tokens()`, a função `dfm()` aplica certas opções em sua configuração padrão, como `tolower()` (convertendo todas as letras em minúsculas, como vimos em tutoriais passados) e remove as pontuações. Mas é importante saber que todas as opções aplicadas à `tokens()` também podem ser utilizadas na `dfm()`.

Para execução das funções de forma mais eficiente no tutorial, tabalharemos com um subconjunto das notícias a partir de 2018.

```{r}
corpus_noticias_2018_2020 <- corpus_subset(corpus_noticias, ano >= 2018)

# make a dfm
dfm_noticias_2018_2020 <- dfm(corpus_noticias_2018_2020)
dfm_noticias_2018_2020[, 1:5]
```

Na função `dfm()` podemos aplicar outras opções, como a remoção das *stopwords* e a stemização dos tokens.

```{r}
dfm_noticias_2018_2020 <- dfm(corpus_noticias_2018_2020,
                            remove = stopwords("portuguese"),
                            stem = TRUE, remove_punct = TRUE,
                            remove_numbers = TRUE)
dfm_noticias_2018_2020[, 1:10]
```

Vimos que a opção `remove` oferece uma lista de tokens a ser ignorados. A maioria dos usuários utiliza uma lista pré-definida de *stopwords*, sendo definidas para diversas línguas no *quanteda*. Podemos acessá-las pela função `stopwords()`. 

```{r}
head(stopwords("pt"), 15)                         # Português
head(c("viram", "a retirada", stopwords("pt")))   # Acrescentando palavras ao português

head(stopwords("en"), 20)                         # Inglês
head(stopwords("ru"), 10)                         # Russo
head(stopwords("ar", source = "misc"), 10)        # Árabe
```

### Visualizando a matriz documento-recurso (*dfm*)

A *dfm* pode ser visualizada no painel do Ambiente Global do RStudio ou utilizando a função `View()`.

```{r}
View(dfm_noticias_2018_2020)
```

Chamar pela função `textplot_wordcloud()` em uma *dfm* criará uma nuvem de palavras.

```{r warning=FALSE, fig.width = 8, fig.height = 8}
textplot_wordcloud(dfm_noticias_2018_2020)
```

Assim como na função `wordcloud()` do pacote *wordcloud* visto na última aula, podemos limitar o número de palavras para tornar a núvem mais inteligível.

```{r warning=FALSE, fig.width = 8, fig.height = 8}
textplot_wordcloud(dfm_noticias_2018_2020, max_words = 50)
```

Para obter a lista dos recursos mais frequentes utilizamos a função `topfeatures()`. Vamos ver para os 10 recursos mais frequentes das notícias a partir de 2018.

```{r}
topfeatures(dfm_noticias_2018_2020, 10)
```
Vemos que as stopwords não removeam o "é", por exemplo. Para corrigir isso, acrescentamos a palavra quando criamos a nossa *dfm*.

```{r}
dfm_noticias_2018_2020 <- dfm(corpus_noticias_2018_2020,
                            remove = c("é", stopwords("pt")),
                            stem = TRUE, remove_punct = TRUE,
                            remove_numbers = TRUE)

topfeatures(dfm_noticias_2018_2020, 10)
```

Produzir uma nuvem de palavras usando o `textplot_wordcloud()` em um objeto *dfm* foi simples, não? Mais bacana é que o pacote *quanteda* possui integração com o *wordcloud* e algumas opções disponíveis na função `wordcloud()` se aplicam aqui. Não personalizamos nossas nuvens de palavra anteriormente, mas vejam o quanto fica mais bonito.

```{r warning=FALSE, fig.width = 7, fig.height = 7}
set.seed(123)
textplot_wordcloud(dfm_noticias_2018_2020, min_count = 35,
                   random_order = FALSE,
                   rotation = .25,
                   color = RColorBrewer::brewer.pal(8, "Dark2"))
```

O código acima precisa de alguns esclarecimentos, uma vez que traz alguns elementos novos. O primeiro é a função `set.seed()`, utilizada para tornar replicáveis ações que contam com certa aleatorização, ou seja, fazem com que a ação seja executada sempre da mesma forma mesmo que por definição tenha componentes aleatórios. Para entender as opções da função [`textplot_wordcloud`](https://quanteda.io/reference/textplot_wordcloud.html), acessem o link e vejam o quão poderosa é para a produção desses gráficos. Se terminarem o tutorial, podem voltar e se divertir por aqui hoje; caso não, vejam após o curso. E, por fim, vocês devem ter percebido que chamamos o pacote *RColorBrewer* para importarmos a paleta de cores utilizada com a função `brewer.pal()`.

### Agrupando documentos pela variável de documento

Em muitas análises, estamos interessados em identificar como os textos se diferem de acordo com fatores substantivos que podem ser codificados nas variáveis do documento, ultrapassando os limites dos próprios documentos. Para isso, podemos agrupar quais documentos que possuem o mesmo valor para uma variável documento quando criamos uma *dfm*. 

Por exemplo, podemos pensar que o ano de produção da notícia carrega contextos diferentes e seria importante formar grupos para cada um deles.

```{r}
dfm_noticias_anos <- dfm(corpus_noticias_2018_2020,
                         groups = "ano",
                         remove = c("é", stopwords("pt")),
                         stem = TRUE, remove_punct = TRUE,
                         remove_numbers = TRUE)
```

Podemos ordenar a *dfm* com a função `dfm_sort()` e inspecioná-la.

```{r}
dfm_sort(dfm_noticias_anos)
```

Ficou estranho, não? O agrupamento por ano é sensível à quantidade de notícias, por isso a grande diferença. Ainda que exista diferença no número de observações, vamos aplicar a classificação de notícias de Lula e Bolsonaro (ou que mencionam os dois) criando uma nova variável no nosso banco de notícias para aplicar no corpus. A função `grepl()` identifica expressões regulares, lembram?

```{r}
dados_noticias <- dados_noticias %>% 
  mutate(politico = ifelse(grepl("Lula", noticias), "Lula", NA),
         politico = ifelse(grepl("Bolsonaro", noticias), "Bolsonaro", politico),
         politico = ifelse(grepl("Lula", noticias) & grepl("Bolsonaro", noticias), "Lula e Bolsonaro", politico),
         politico = ifelse(is.na(politico), "Outros", politico))
```

Aplicamos no corpus a variável *político* como *docvars* e criamos um novo subset pós 2010 antes de criar uma nova *dfm* agrupando por políticos.

```{r}
docvars(corpus_noticias, "politico") <- dados_noticias$politico

corpus_noticias_2010_2020 <- corpus_subset(corpus_noticias, ano >= 2010)

dfm_noticias_politicos <- dfm(corpus_noticias_2010_2020,
                         groups = "politico",
                         remove = c("é", stopwords("pt")),
                         stem = TRUE, remove_punct = TRUE,
                         remove_numbers = TRUE)
```

Novamente ordenamos a *dfm* com a função `dfm_sort()` e inspecionamos.

```{r}
dfm_sort(dfm_noticias_politicos)
```

Faz muito mais sentido, não?

### Agrupando palavras por dicionários ou classes de equivalência

Se lembram que na apresentação do Maurício foi falado de dicionários que são aplicados em análises? Em alguns casos existem classificações prévias estabelecidas que indicam características que gostaríamos de medir no texto.

Como exemplos, a análise de sentimentos se baseia em uma lista de palavras positivas e negativas. Os trabalhos citados que utilizam dicionários de termos polítics que são associados a determinadas posições. 

Em alguns casos é útil tratar os grupos de palavras como equivalentes para a análise e somar quantos temos em cada classe. Aqui, ainda utilizando nossos dados do DataFolha, podemos criar um dicionário exemplo para analisar as notícias, definindo termos ligados ao tema "eleições" e outro a debates "liberais" (aqui não tivemos cuidado teórico algum, ok? é somente para fins didáticos).

```{r}
dicionario <- dictionary(list(eleicoes = c("candidato", "político", "eleições", "votos"),
                              liberal = c("mercado", "governo", "estado", "redução")))
```

Agora aplicamos o dicionário na *dfm*, que aqui será do nosso subconjunto de notícias a partir de 2010.

```{r}
corpus_noticias_2010_2020 <- corpus_subset(corpus_noticias, ano >= 2010)

dfm_noticias_dicionario <- dfm(corpus_noticias_2010_2020,
                               remove = c("é", stopwords("pt")),
                               stem = TRUE, remove_punct = TRUE,
                               remove_numbers = TRUE,
                               dictionary = dicionario)

dfm_noticias_dicionario
```

Como resultado, temos o número de veses que recursos estão associados a cada um dos grupos definidos. Bacana, não? Essa lógica estava presente em alguns dos trabalhos da apresentação de ontem. É importante saber que é possível utilizar dicionários externos com a função `dictionary()` (nos formatos LIWC and Provalis Research's Wordstat).

Caso tenha chegado até aqui, dê uma olhada no [Tutorial 9](https://github.com/thiagomeireles/cebraplab_texto_como_dados/blob/master/tutoriais/Tutorial_9.md). Basicamente, apresenta algumas funções de modelagem de dados textuais oferecidas pela própria documentação do pacote *quanteda* com algumas explicações adicionais. Para *topic modeling* utilizamos o corpus do DataFolha. Nos demais são corpus que fazem parte do próprio pacote *quanteda* para fins didáticos.
