Tutorial 4

Nos atividades anteriores utilizamos as ferramentas básicas de captura de dados disponíveis na biblioteca *rvest*. Primeiro, aprendemos a capturar várias páginas contendo tabelas em formato HTML de uma só vez. Depois, aprendemos como um documento XML está estruturado e que podemos a extrair com precisão os conteúdos de tags e os valores dos atributos das tags de páginas escritas em HTML.

Neste tutorial vamos colocar tudo em prática e construir um banco de dados de notícias. O nosso exemplo será o conjunto de notícias (670 na data da construção deste tutorial) publicadas sobre eleições no site do instituto de pesquisa DataFolha. Ainda que o DataFolha não seja um portal, por estar vinculado ao jornal Folha de São Paulo e ao portal UOL, a busca do DataFolha se assemelha muito às ferramentas de busca destes últimos.

Clique no [link](http://search.folha.uol.com.br/search?q=eleicoes&site=datafolha&skin=datafolha&results_count=669&search_time=1%2C067&url=http%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Deleicoes%26site%3Ddatafolha%26skin%3Ddatafolha&sr=) e veja como está estruturada a busca do DataFolha sobre eleições:

# Raspando uma notícia no site DataFolha

Antes de começar, vamos chamar a biblioteca _rvest_ para tornar suas funções disponíveis em nossa sessão do R. Na carona, vamos chamar também a biblioteca _dplyr_.

```{r}
library(rvest)
library(dplyr)
```

Nossa primeira tarefa será escolher uma única notícia (a primeira da busca, por exemplo), e extrair dela 4 informações de interesse: o título da notícia; a data e hora da notícia; o link para a pesquisa completa em .pdf; e o texto da notícia.

O primeiro passo é criar um objeto com endereço URL da notícia e outro que contenha o código HTML da página:

```{r}
url_noticia <- "http://datafolha.folha.uol.com.br/eleicoes/2020/09/1988871-com-29-russomanno-lidera-disputa-para-prefeito-de-sp.shtml"
pagina <- read_html(url_noticia)
```

Pronto! Agora temos um objeto adequadamente identificado como XML pelo R. Com o objeto XML preparado e representando a página com a qual estamos trabalhando, vamos à caça das informações que queremos.

Volte para a página da notícia. Procure o título da notícia e examine-o, inspencionando o código clicando com o botão direito do mouse e selecionando "Inspecionar". Note o que encontramos:

```{xml}
<h1 class="main_color main_title"><!--TITULO-->"Com 29%, Russomanno lidera disputa para prefeito de SP"<!--/TITULO--></h1>
```

Tente sozinh@ e por aproximadamente 1~2 minutos construir o "xpath" (caminho em XML) que nos levaria a este elemento antes de avançar. (Tente sozinh@ antes de copiar a resposta abaixo!)
  
A resposta é:

```{xml}
//h1[@class = 'main_color main_title']
```

Usando agora as funções _html\_nodes_ e _html\_text_, como vimos no tutorial anterior, vamos capturar o título da notícia:

```{r}
node_titulo <- html_nodes(pagina, xpath = "//h1[@class = 'main_color main_title']")
titulo <- html_text(node_titulo)
print(titulo)
```

Simples, não? Repita agora o mesmo procedimento para data e hora (tente sozinh@ antes de copiar a resposta abaixo!):

```{r}
node_datahora <- html_nodes(pagina, xpath = "//time")
datahora <- html_text(node_datahora)
print(datahora)
```

E também para o link do .pdf disponibilizado pelo DataFolha com o conteúdo completo da pesquisa -- dica: o link é o valor do atributo "href" da tag "a" que encontramos ao inspecionar o botão para donwload:

```{r}
node_pdf <- html_nodes(pagina, xpath = "//p[@class = 'stamp download']/a")
link_pdf <- html_attr(node_pdf, name = "href")
print(link_pdf)
```

Finalmente, peguemos o texto. Note que o texto está dividido em vários parágrafos cujo conteúdo está inseridos em tags "p", todas filhas da tag "article". Se escolhemos o xpath sem especificar a tag "p" ao final, como abaixo, capturamos um monte de "sujeira", como os botões de twitter e facebook. Por esta razão, vamos especificar a tag "p" ao final do xpath. Neste caso, recebemos um vetor contendo cada um dos parágrafos do texto. Precisaríamos "juntar" (concatenar) todos os parágrafos para formar um texto único. Faremos isso a seguir.

```{r}
node_texto <- html_nodes(pagina, xpath = "//article[@class = 'news']/p")
texto <- html_text(node_texto)
print(texto)
texto <- gsub("\n", "", texto)
# Mais uma vez substituímos o marcador de parágrafo
```

A função _paste_ (que é igual _paste0_, mas tem por padrão deixar espaço entre os textos "colados") permite juntarmos todos os textos de um vetor do tipo "character" em um só utilizando o argumento "collapse". Vamos, assim, juntar todos os parágrafos com _paste_ separando-os por um espaço simples:

```{r}
texto <- paste(texto, collapse = " ")
```

### Tarefa 1: Tente raspar a notícia seguinte na busca do DataFolha

Tente agora raspar a notícia seguinte usando a mesma estratégia. É fundamental notar que variamos a notícia, mas as informações continuam tendo o mesmo caminho. Essa é a propriedade fundamental do portal raspado que nos permite obter todas as notícias sem nos preocuparmos em abrir uma por uma. O link para a próxima notícia está no objeto "url" abaixo:

```{r}
url_noticia <- "http://datafolha.folha.uol.com.br/opiniaopublica/2019/09/1988409-maioria-acredita-que-crescimento-economico-nao-vira-no-curto-prazo.shtml"
```

Obs: agora que criarão seus próprios *scripts* com os códigos para raspagem dessa página, não se esqueçam de documentar o que estão fazendo. Isso é muito importante para qualquer pessoa que vá replicar o que fizeram, mas, também, para que consiga entender o que fez mesmo depois de muito tempo. Como muitos pacotes sofrem alterações, inclusive com substituição de funções, isso auxilia a resolver problemas que possam aparecer. Parte importante na programação é deixar claro para você e para os outros o que e o porquê está fazendo cada etapa.

# Download de arquivos

Por vezes, queremos fazer donwload de um arquivo cujo link encontramos na página raspada. Por exemplo, no datafolha seria interessante obter o relatório em .pdf da pesquisa (para extrair seu conteúdo no futuro, por exemplo). Vamos ver como fazer download de um arquivo online.

Em primeiro lugar, obtemos seu endereço URL, como acabamos de fazer com a notícia que capturamos na busca do DataFolha (tente ler o código e veja se o entende por completo):

```{r}
url_noticia <- "http://datafolha.folha.uol.com.br/eleicoes/2020/09/1988871-com-29-russomanno-lidera-disputa-para-prefeito-de-sp.shtml"
pagina <- read_html(url_noticia)
node_pdf <- html_nodes(pagina, xpath = "//p[@class = 'stamp download']/a")
link_pdf <- html_attr(node_pdf, name = "href")
```

O link está no objeto "pesquisa":

```{r}
print(link_pdf)
```

Usando a função _download.file_, rapidamente salvamos o link no *working directory* (use "getwd()" para descobrir qual é o seu) e com o nome "pesquisa.pdf" (poderíamos salvar com o nome que quisessemos). Caso queira alterar seu *working directory*, use a função *setwd*, como no exemplo não executável no *chunk*.

```{r}
getwd()
#setwd("/home/thiago/Documentos/Cebrap_lab_2020")
download.file(link_pdf, "pesquisa.pdf", mode = 'wb')
```

Vá ao "working directory" e veja o arquivo!

Sempre que estiver em posse de um conjunto de links que contém arquivos, você pode colocar a função "download.file" em loop e capturar todos os objetos ao mesmo tempo. Há uma dificuldade boba: nomear sem repetir os nomes diversos arquivos. Uma dica é usar o final do endereço URL como nome, mas você pode salvar os arquivos com nomes que sejam uma sequência numérica ou que provenham de um vetor que contenha os nomes todos. Use a criatividade!

Vamos voltar agora às notícias do DataFolha em HTML e ignorar o donwload de arquivos com os relatórios das pesquisas.

# Um código, duas etapas: raspando todas as notícias de eleições do DataFolha

Vamos fazer um breve roteiro do que precisamos fazer para criar um banco de dados que contenha todos os títulos, data e hora e texto de todas as notícias sobre eleições do DataFolha (Obs: por enquanto vamos ignorar os links de pesquisa, pois nem todas as notícias contêm os links e isso causa interrupção do código)

### Etapa 1
* Passo 1: conhecer a página de busca (e compreender como podemos "passar" de uma página para outra)
* Passo 2: raspar (em loop!) as páginas de busca para obter todos os links de notícia

Esta é a primeira etapa da captura. Em primeiro lugar temos que buscar todos os URLs que contêm as notícias buscadas. Em outras palavras, começamos obtendo "em loop" os links das notícias e, só depois de termos os links, obtemos o conteúdo destes links. Nossos passos seguintes, portanto, são:

### Etapa 2
* Passo 3: conhecer a página da notícia (e ser capaz de obter nela as informações desejadas). Já fizemos isso acima!
* Passo 4: raspar (em um novo loop!) o conteúdo dos links capturados no Passo 2.

Vamos construir o código da primeira etapa da captura e, uma vez resolvida a primeira etapa, faremos o código da segunda.

### Código da etapa 1

AVISO: Esse código é bastante semelhante ao código do [Tutorial 3](https://github.com/thiagomeireles/cebraplab_captura_R/blob/master/tutorials/webscraping_cebrap_03.md). Mudam apenas os urls e o xpath. Você pode passar por ele repidamente se quiser.

Comecemos criando o URL base sem a numeração de notícias:

```{r}
url_base <- "http://search.folha.uol.com.br/search?q=eleicoes&site=datafolha&skin=datafolha&results_count=670&search_time=0%2C133&url=http%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Deleicoes%26site%3Ddatafolha%26skin%3Ddatafolha&sr="
```

Em segundo lugar, vamos observar o URL da página de busca (poderíamos buscar termos chave, mas, neste caso, vamos pegar todas as notícias relacionadas a eleições). Na página 2 da busca vemos que o final é "sr=26". Na página 3 o final é "sr=51". Há um padrão: as buscas são realizadas de 25 em 25. De fato, a 27a. é última página da busca. Para "passarmos" de página em página, portanto, temos que ter um "loop" que conte não mais de 1 até 25, mas na seguinte sequência numérica: {1, 26, 51, 76, ..., 626, 651}.

Precisamos, então, que "i" seja recalculado dentro do loop para coincidir com a numeração da primeira notícia de cada página. Parece difícil, mas é extremamente simples. Veja o loop abaixo, que imprime a sequência desejada multiplicando (i - 1) por 25 e somando 1 ao final:

```{r}
for (i in 1:27){
  i <- (i - 1) * 25 + 1
  print(i)
}
```

Precisamos agora é incluir nas "instruções do loop". Assim, construímos o url de cada página do resultado da busca:

```{r}
url_pesquisa <- paste(url_base, i, sep = "")
```

A seguir, capturamos o código HTML da página:

```{r}
pagina <- read_html(url_pesquisa)
```

Escolhemos apenas os "nodes" que nos interessam:

```{r}
nodes_titulos <- html_nodes(pagina, xpath = "//h2/a")
```

Extraímos os títulos e os links com as funções apropriadas:

```{r}
titulos <- html_text(nodes_titulos)
links <- html_attr(nodes_titulos, name = "href")
```

Combinamos os dois vetores em um data frame:

```{r}
tabela_titulos <- data.frame(titulos, links)
```

Falta "empilhar" o que produziremos em cada iteração do loop de uma forma que facilite a visualização. Criamos um objeto vazio antes do loop. 

Usaremos a função _bind\_rows_ (ou _rbind_ se estiver com problemas com o _dplyr_) para combinar data frames. A cada página agora, teremos 25 resultados em uma tabela com duas variáveis. O que queremos é a junção dos 25 resultados de cada uma das 27 páginas. Vamos também chamar a biblioteca _dplyr_ para usar sua função _bind\_rows_.

```{r}
library(dplyr)
dados_pesquisa <- data.frame()
dados_pesquisa <- bind_rows(dados_pesquisa, tabela_titulos)
```

Chegou o momento de colocar dentro loop tudo o que queremos que execute em cada uma das vezes que ele ocorrer. Ou seja, que imprima na tela a página que está executando, que a URL da página de resultados seja construída com a função paste, para todas elas o código HTML seja examinado, lido no R e transformado em objeto XML, colete todos os links e todos os títulos e que "empilhe". Lembrando que não podemos esquecer de definir a URL que estamos usando e criar um data frame vazio para colocar todos os links e títulos coletados antes de iniciar o loop.

```{r}
url_base <- "http://search.folha.uol.com.br/search?q=eleicoes&site=datafolha&skin=datafolha&results_count=670&search_time=0%2C133&url=http%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Deleicoes%26site%3Ddatafolha%26skin%3Ddatafolha&sr="

dados_pesquisa <- data_frame()

for (i in 1:27){
  
  print(i)

  i <- (i - 1) * 25 + 1
  
  url_pesquisa <- paste(url_base, i, sep = "")
  
  pagina <- read_html(url_pesquisa)
  
  nodes_titulos <- html_nodes(pagina, xpath = "//h2/a")
  
  titulos <- html_text(nodes_titulos)
  links <- html_attr(nodes_titulos, name = "href")
  
  tabela_titulos <- data.frame(titulos, links)
  
  dados_pesquisa <- bind_rows(dados_pesquisa, tabela_titulos)
}
```

Agora todos os títulos e links de todos os resultados de site do DataFolha estão em um único banco de dados. Bom trabalho!

### Código da etapa 2

No começo do tutorial resolvemos a captura do título, data e hora, link para o relatório completo de pesquisa e texto para uma única notícia no portal do instituto DataFolha. Nos resta agora capturar, em loop, o conteúdo de cada uma das páginas cujos links estão guardados no vetor "links_datafolha".

Vamos rever o procedimento, para uma URL qualquer, da captura do título, data e hora e texto (vamos deixar o link para o relatório de pesquisa de lado por enquanto, posto que algumas notícias não contêm o link e esta pequena ausência interromperia o funcionamento do código).

```{r}
url_noticia <- "http://datafolha.folha.uol.com.br/eleicoes/2020/09/1988871-com-29-russomanno-lidera-disputa-para-prefeito-de-sp.shtml"

pagina <- read_html(url_noticia)

node_titulo <- html_nodes(pagina, xpath = "//h1[@class = 'main_color main_title']")
titulo <- html_text(node_titulo)

node_datahora <- html_nodes(pagina, xpath = "//time")
datahora <- html_text(node_datahora)

node_texto <- html_nodes(pagina, xpath = "//article[@class = 'news']/p")
texto <- html_text(node_texto)
texto <- gsub("\n", "", texto)
texto <- paste(texto, collapse = " ")
```

Para fazermos a captura de todos os links em "loop" deve ter o seguinte aspecto, como se vê no código abaixo que imprime todos os 670 links cujo conteúdo queremos capturar. Note que a forma de utilizar o loop é ligeiramente diferente da que havíamos visto até então. No lugar de uma variável "i" que "percorre" um vetor numérico (1:27, por exemplo), temos uma variável "link" que recebe, a cada iteração, um endereço URL da notícia, em ordem.

Esses endereços estão armazenados na variável "links" do banco de dados que criamos na Etapa 1, "dados\_pesquisa". Assim, na primeira iteração temos que "link" será igual "dados_pesquisa\$links[1]", na segunda "dados_pesquisa\$link[2]" e assim por diante até a última posição do vetor "links_datafolha" -- no nosso caso a posição 669.

```{r}
for (link in dados_pesquisa$links){
  print(link)
}
```

Combinando os dois código, e criando um data frame "dados\_noticias" que é vazio antes do loop temos o código completo da captura. Tal como quando trabalhamos com tabelas, utilizando a função "bind_rows" para combinar o data frame que resultou da iteração anterior com a linha que combina o conteúdo armazenado em "titulo", "datahora" e "texto". Faremos isso para as 100 primeiras notícias por agora, voltaremos mais tarde para buscar de todos.

```{r}
dados_noticias <- data.frame()

for (link in dados_pesquisa$links[1:100]){
  print(link)
  
  pagina <- read_html(link)
  
  node_titulo <- html_nodes(pagina, xpath = "//h1[@class = 'main_color main_title']")
  titulo <- html_text(node_titulo)
  
  node_datahora <- html_nodes(pagina, xpath = "//time")
  datahora <- html_text(node_datahora)
  
  node_texto <- html_nodes(pagina, xpath = "//article[@class = 'news']/p")
  texto <- html_text(node_texto)
  texto <- paste(texto, collapse = " ")
  
  tabela_noticia <- data_frame(titulo, datahora, link, texto)
  
  dados_noticias <- bind_rows(dados_noticias, tabela_noticia)

}
```

O resultado do código é um data frame ("dados\_noticias") que contém 4 variáveis em suas colunas: "titulo", "datahora", "link" e "texto". A partir de agora você poderia, por exemplo, usar as ferramentas de "text mining" para criar uma nuvem de palavras ("wordcloud"), fazer a contagem de termos, examinar a semelhança da linguagem usada pelo instituto DataFolha com a usada por outros institutos de opinião pública, fazer análise de sentimentos, etc.

### Tarefa 2: Comentando o código

Antes disso, sua tarefa é a seguinte: executar ambas as etapas do código e comentá-lo por completo (use # para inserir linhas de comentário). Comentar o código alheio é uma excelente maneira de ver se você conseguiu compreendê-lo por completo e serve para você voltar ao código no futuro quando for usá-lo de modelo para seus próprios projetos em R.

Dica: Compare o número de obsevações dos dataframes _dados_noticias_ e _dados_pesquisa_. O que ocorreu?

```{r}
url_base <- "http://search.folha.uol.com.br/search?q=eleicoes&site=datafolha&skin=datafolha&results_count=670&search_time=0%2C133&url=http%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Deleicoes%26site%3Ddatafolha%26skin%3Ddatafolha&sr="

dados_pesquisa <- data_frame()

for (i in 1:27){
  
  print(i)
  
  i <- (i - 1) * 25 + 1
  
  url_pesquisa <- paste(url_base, i, sep = "")
  
  pagina <- read_html(url_pesquisa)
  
  nodes_titulos <- html_nodes(pagina, xpath = "//h2/a")
  
  titulos <- html_text(nodes_titulos)
  links <- html_attr(nodes_titulos, name = "href")
  
  tabela_titulos <- data.frame(titulos, links)
  
  dados_pesquisa <- bind_rows(dados_pesquisa, tabela_titulos)
}

dados_noticias <- data.frame()

for (link in dados_pesquisa$links){
  print(link)
  
  pagina <- read_html(link)
  
  node_titulo <- html_nodes(pagina, xpath = "//h1[@class = 'main_color main_title']")
  titulo <- html_text(node_titulo)
  
  node_datahora <- html_nodes(pagina, xpath = "//time")
  datahora <- html_text(node_datahora)
  
  node_texto <- html_nodes(pagina, xpath = "//article[@class = 'news']/p")
  texto <- html_text(node_texto)
  texto <- paste(texto, collapse = " ")
  
  tabela_noticia <- data_frame(titulo, datahora, link, texto)
  
  dados_noticias <- bind_rows(dados_noticias, tabela_noticia)
  
}
```

## URL inexistente

Como vocês devem ter percebido, no momento do curso alguns URLs de notícias do Data Folha estavam fora do ar ou deixaram de existir. Ao tentarmos capturar uma página inexistente, o código é interrompido. Como lidar com esse problema?

Vamos incluir uma cláusula de "tentativa" na captura da página com a função _try_. Veja um exemplo com um URL inexistente:

```{r}
url_inexistente <- "http://datafolha.folha.uol.com.br/eleicoes/2018/10/42424242-ronaldinho-gaucho-e-eleito-senador-por-minas-gerais-com-record-de-votos.shtml"
pagina <- try(read_html(url_inexistente))
```

Quando o URL não existe, o código não é interrompido e o objto página, que deveria ser da classe "xml_document", passa a ser da classe "try-error"

```{r}
class(pagina)
```

Podemos, então, escrever uma cláusula condicional para evitar raspar esta página inexistente:

```{r}
if (!(class(pagina) %in% "try-error")){
  print("Página existente")
} else {
  print("Página inexistente")  
}
```

Pronto. Vamos adicionar esse pedaço de código (sem as mensagens) ao nosso programa completo e tentar novamente. Tente compreender como o código funciona. O resultado final, no momento da elaboração do tutorial, é de 662 entradas (uma não é notícia, mas qual?).

```{r}
url_base <- "http://search.folha.uol.com.br/search?q=eleicoes&site=datafolha&skin=datafolha&results_count=669&search_time=1%2C067&url=http%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Deleicoes%26site%3Ddatafolha%26skin%3Ddatafolha&sr="

dados_pesquisa <- data_frame()

for (i in 1:27){
  
  print(i)
  
  i <- (i - 1) * 25 + 1
  
  url_pesquisa <- paste(url_base, i, sep = "")
  
  pagina <- read_html(url_pesquisa)
  
  nodes_titulos <- html_nodes(pagina, xpath = "//h2/a")
  
  titulos <- html_text(nodes_titulos)
  links <- html_attr(nodes_titulos, name = "href")
  
  tabela_titulos <- data.frame(titulos, links)
  
  dados_pesquisa <- bind_rows(dados_pesquisa, tabela_titulos)
}

dados_noticias <- data.frame()

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
    
    tabela_noticia <- data_frame(titulo, datahora, link, texto)
    
    dados_noticias <- bind_rows(dados_noticias, tabela_noticia)
  } 
  
}
```

Amanhã retomaremos os textos do DataFolha para iniciar a manipulação de texto como dado.    
