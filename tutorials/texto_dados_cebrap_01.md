# Pacotes no R

Muitas ações que precisamos em atividades diversas executadas não fazem parte da biblioteca básica do R, mas outros desenvolvedores já desenvolveram funções para isso. Em muitos casos, diversas funções são compiladas em novas bibliotecas direcionadas para atividades bem específicas. Essas bibliotecas (ou pacotes de funções) são disponibilziadas pela comunidade de R e, após aprovação, vão para um diretório com bibliotecas já testadas, o [CRAN](https://cran.r-project.org/web/packages/policies.html).

Nesse primeiro momento, precisamos de uma biblioteca chamada _rvest_. Ela possui funções utilizadas para facilitar o download e a manipulação de conteúdo proveniente de *html* e *xml*. Assim, nossa primeira ação é realizar o processo de instalação de uma biblioteca.Para isso, basta executarmos o comando abaixo:

```{r}
install.packages("rvest")
```

No entanto, mesmo com a biblioteca instalada as funções não ficam disponíveis automaticamente. É necessário carregar a biblioteca para torná-las disponíveis. Assim, vamos executar o comando para tornar as funções da biblioteca _rvest_ disponíveis. Basta executar o comando abaixo:

```{r}
library(rvest)
```

Excelente! Já temos boa parte das funções que precisamos disponíveis para o nosso primeiro tutorial. Vamos utilizá-las logo mais.

# Tidyverse

O pacote _rvest_ faz parte de um "universo" de pacotes camado _tidyverse_. O _tidyverse_ é uma compilação de diversas bibliotecas que, grosso modo, compõem uma linguagem "alternativa" dentro do R. Os pacotes mais conhecidos são o _dplyr_ e o _ggplot2_.

Diversas funções que fazem parte do _tidyverse_ serão utilizadas ao longo da semana e serão destacadas quando surgirem. No entanto, já vamos realizar a instalação do pacote. Aqui, no entanto, faremos um processo um pouco mais complexo: pediremos para que o R cheque se o pacote já está instalado e, caso não esteja, realize a instalação.



```{r}
if (!require("tidyverse")) install.packages("tidyverse"); library(tidyverse)
```

Dica: se você "chamar" o pacote _tidyverse_, não precisará chamar _rvest_, pois a função do _tidyverse_ é carregar todos os pacotes que o compõem.

# Capturando o conteúdo de uma página com _rvest_

Para acessar o conteúdo de uma página, precisamos de funções que façam algo semelhante a um navegador de internet, ou seja, que se comuniquem com o servidor da página e receba o seu conteúdo. Para capturar uma página, ou melhor, o código HTML no qual a página está escrita, utilizamos a função _read\_html_, do pacote _rvest_. Vamos usar um exemplo com wikipedia.

```{r}
url <- "https://pt.wikipedia.org/wiki/Lista_de_pa%C3%ADses_e_territ%C3%B3rios_por_%C3%A1rea"
pagina <- read_html(url)
print(pagina)
```

O resultado é um documento "xml_document" que contém o código html que podemos inspecionar usando o navegador. Vamos entender nos próximos tutoriais o que é um documento XML, por que páginas em HTML são documentos XML e como navegar por eles. Por enquanto, basta saber que ao utilizarmos _read\_html_, capturamos o conteúdo de uma página e o armezenamos em um objeto bastante específico.

Neste primeiro tutorial, trabalharemos com um objeto bastante específico em páginas de internet: as tabelas. No caso da página que abrimos, temos a lista dos países de maior área no mundo divididos por categorias. Abrindo a página na [Wikipedia](https://pt.wikipedia.org/wiki/Lista_de_pa%C3%ADses_e_territ%C3%B3rios_por_%C3%A1rea), podemos ver que esses dados estão dentro de uma tabela quando clicamos com o botão direito sobre os dados e inspecionamos o conteúdo.

O _rvest_ possui uma função específica para ler as tabelas e fazer o download para nosso ambiente global, a _html_table_. Ela lê um "xml_document" e extrai TODAS as tabelas escritas em HTML da url e retorna uma lista (lembra delas do tutorial pré curso? Voltaremos a elas mais adiante).

```{r}
tabelas_wiki <- html_table(pagina)
class(tabelas_wiki)
str(tabelas_wiki)
```

Utilizando as funções _class_ e _str_ conseguimos identificar o tipo de objeto gerado e como ele foi estruturado. Geramos em uma lista com 7 tabelas. Voltaremos a elas mais tarde.

# Atividade inicial - Pesquisa de Proposições na ALESP

## For loop e links com numeração de página

Vamos começar visitando o site da ALESP e entrar na ferramenta de pesquisa de proposições. Clique [aqui](http://www.al.sp.gov.br/alesp/pesquisa-proposicoes/) para acessar a página.

Ao acessar o site, podemos elaborar uma pesquisa que retorne um númedo de respostas que tornaria ineficiente a coleta manual, como, por exemplo, a palavra "previdência".

Os 3533 resultados estão divididos em 177 páginas com 20 observações cada. Em cada um dos resultados, existem 4 tipos de informação: autor, tipo do projeto com a data de proposição, nome do pojeto, e uma pequena ementa da proposta. 

Podemos prosseguir, clicando nos botões de navegação ao final da página, para as demais páginas da pesquisa. Por exemplo, podemos ir para a página 2 clicando uma vez na seta indicando à direita.

OBS: Há uma razão importante para começarmos nosso teste com a segunda página da busca. Em diversos servidores web, como este da ALESP, o link (endereço url) da primeira página é "diferente" dos demais. Em geral, os links são semelhantes da segunda página em diante.

Nossa primeira tarefa consiste em capturar estas informações. Vamos, no decorrer da atividade aprender bastante sobre R, objetos, estruturas de dados, loops, captura de tabelas em HTML e manipulação de dados.

Vamos armazenar a URL em um objeto ("url_base", mas você pode dar qualquer nome que quiser).

```{r}
url_base <- "https://www.al.sp.gov.br/alesp/pesquisa-proposicoes/?direction=acima&lastPage=177&currentPage=1&act=detalhe&idDocumento=&rowsPerPage=20&currentPageDetalhe=1&tpDocumento=&method=search&text=previdencia&natureId=&legislativeNumber=&legislativeYear=&natureIdMainDoc=&anoDeExercicio=&strInitialDate=&strFinalDate=&author=&supporter=&politicalPartyId=&stageId="
```

Há muitas informações nesse link, basicamente todos os campos que poderiam ter sido especificados na busca são apresentados no endereço. Como preenchemos apenas com o termo ("previdencia"), esse será o único parametro definido na URL ("text=previdencia").

Também podemos observar que a pesquisa retornou 177 páginas ("lastPage=177") e que a página atual (2) é a de número 1 ("currentPage=1") -- a página 1 da pesquisa é numerada como 0 nesta ferramenta de busca, mas o padrão muda de página para página.

Podemos ver que há muitas páginas de resultado para a palavra-chave que utilizamos. Nosso desafio é conseguir "passar" de forma eficiente por elas, ou seja, acessar os 177 links e "raspar" o seu conteúdo. Para isso, usaremos uma função essencial na programação, o "for loop".

Loops são processos iterativos e são extremamente úteis para instruir o computador a repetir uma tarefa por um número finito de vezes. Por exemplo, vamos começar "imprimindo" na tela os números de 1 a 9:

```{r}
for (i in 1:9) {
  print(i)
}
```

Simples, não? Vamos ler esta instrução da seguinte maneira: "para cada número i no conjunto que vai de 1 até 9 (essa é a parte no parênteses) imprimir o número i (instrução entre chaves)". E se quisermos imprimir o número i multiplicado por 7 (o que nos dá a tabuada do 7!!!), como devemos fazer?

```{r}
for (i in 1:9) {
  print(i * 7)
}
```

Tente agora construir um exemplo de loop que imprima na tela os números de 3 a 15 multiplicados por 10 como exerício.

```{r, echo=FALSE, eval=FALSE}
for (i in 3:15) {
  print(i * 10)
}
```

## Substituição com gsub

Cumprimos uma etapa importante: observamos o funcionamento dos loops de forma intuitiva. O próximo passo é fazer com que ele "passe" pelas páginas que contém a informação que nos interessa. Assim, devemos instruir o programa a passar pelas páginas de 1 a 177, substituindo apenas o número da página atual -- "currentPage" -- no endereço URL que guardamos no objeto url_base.

Para isso precisamos, no entando, de uma função que permita substituir no texto básico do URL ("url_base") o número da página. Ainda que existam diferentes formas para se realizar essa tarefa, aqui utilizaremos a função _gsub_. Ela é uma função básica da linguagem e permite a substituição de um pedaço de um objeto de texto por outro a partir de um critério especificado. 

Os argumentos (o que vai entre os parenteses) da função são, em ordem, o termo a ser substituído, o termo a ser colocado no lugar e o objeto no qual a substituição ocorrerá. Na prática, ela funciona da seguinte forma:

```{r}
o_que_procuro_para_susbtituir <- "palavra"
o_que_quero_substituir_por <- "batata"
meu_texto <- "quero substituir essa palavra"

texto_final <- gsub(o_que_procuro_para_susbtituir, o_que_quero_substituir_por, meu_texto)

print(texto_final)
```

Agora que sabemos substituir partes de textos e fazer loops, podemos mudar o número da página do nosso endereço de pesquisa.

Já observamos que na URL, o que varia de uma página para outra é o resultado da expressão "currentPage". De outra forma, vamos da "currentPage=1" para "currentPage=2" e assim por diante. Sabemos que, em nossa pesquisa, o "1" é a segunda página e o "2" a terceira, e assims se segue. As páginas possuem peculiaridades e isso torna importante conhecer a página que queremos "raspar". No fim, a captura de informações em páginas de internet é quase uma atividade "artesanal". 

Posto isso, vamos substituir na URL da página 2 de nossa busca o número por uma expressão que "guarde o lugar" do número da página. Esse algo é um "placeholder" e pode ser qualquer texto. No caso, usaremos "NUMPAG". Veja abaixo onde "NUMPAG" foi introduzido no endereço URL. 

Obs: Lembremos que ao colocar na URL, não devemos usar as aspas. Ainda assim devemos usar aspas ao escrever "NUMPAG" como argumento de uma função, pois queremos dizer que procuramos a palavra "NUMPAG" e não o objeto chamado NUMPAG.

```{r}
url_base <- "https://www.al.sp.gov.br/alesp/pesquisa-proposicoes/?direction=acima&lastPage=177&currentPage=NUMPAG&act=detalhe&idDocumento=&rowsPerPage=20&currentPageDetalhe=1&tpDocumento=&method=search&text=previdencia&natureId=&legislativeNumber=&legislativeYear=&natureIdMainDoc=&anoDeExercicio=&strInitialDate=&strFinalDate=&author=&supporter=&politicalPartyId=&stageId="
```

Por exemplo, se quisermos gerar o link da página 6, podemos escrever:

```{r}
url <- gsub("NUMPAG", "6", url_base)

print(url)
```

Ou, em vez de usar um número diretamente na substituição, podemos usar um vetor que represente um número -- por exemplo a variável i, que já usamos no loop anteriormente.

```{r}
i <- 6
url <- gsub("NUMPAG", i, url_base)

print(url)
```

Agora que temos o código substituindo funcionando, vamos implementar o loop para que as URLs das páginas sejam geradas automaticamente. Por exemplo, se quisermos "imprimir" na tela as páginas 0 a 5, podemos usar o seguinte código:

```{r}
url_base <- "https://www.al.sp.gov.br/alesp/pesquisa-proposicoes/?direction=acima&lastPage=177&currentPage=NUMPAG&act=detalhe&idDocumento=&rowsPerPage=20&currentPageDetalhe=1&tpDocumento=&method=search&text=previdencia&natureId=&legislativeNumber=&legislativeYear=&natureIdMainDoc=&anoDeExercicio=&strInitialDate=&strFinalDate=&author=&supporter=&politicalPartyId=&stageId="

for(i in 0:5){
  url <- gsub("NUMPAG", i, url_base)
  print(url)
}
```

## Capturando o conteúdo da pesquisa com _rvest_

Muito mais simples do que parece, não? Mas veja bem, até agora tudo que fizemos foi produzir um texto que, propositalmente, é igual ao endereço das páginas cujo conteúdo nos interessa. Porém, ainda não acessamos o seu conteúdo. Aqui recorreremos mais uma vez à função _read_html_ do _rvest_.

Para capturar uma página, ou melhor, o código HTML no qual a página está escrita, utilizamos a função _read\_html_, do pacote _rvest_. Vamos observar o que encontramos com o último *url* da busca no site da ALESP.

```{r}
pagina <- read_html(url)
tables <- html_table(pagina)
print(tables)
```

O "xml_document" resultante contém o código html e lembramos que capturamos o conteúdo de uma página e o armezenamos em um objeto bastante específico. Não podemos esquecer isso!

## Função read_table

Nosso objetivo, uma vez que capturamos o conteúdo da página, é extrair de lá a tabela com as informações que nos interessam. Utilizaremos, mais uma vez, a função _read\_table_ da biblioteca _rvest_ (lembra que chamamos esta biblioteca e extraímos uma tabela da wikipedia lá no começo da atividade?).

Esta função serve bem ao nosso caso: ela recebe uma página capturada como argumento (ou melhor, um "xml\_document", como acabamos de ver), extrai todas (sim, todas) as tabelas da url, escritas em HTML, e retorna uma lista contendo as tabelas.

Vamos ver como ela funciona para a página 2 (ou seja, currentPage=1) contendo a tabela com os resultados em que aparecem o nosso termo pesquisado:

```{r}
url_base <- "https://www.al.sp.gov.br/alesp/pesquisa-proposicoes/?direction=acima&lastPage=177&currentPage=1&act=detalhe&idDocumento=&rowsPerPage=20&currentPageDetalhe=NUMPAG&tpDocumento=&method=search&text=previdencia&natureId=&legislativeNumber=&legislativeYear=&natureIdMainDoc=&anoDeExercicio=&strInitialDate=&strFinalDate=&author=&supporter=&politicalPartyId=&stageId="

i <- 1
url <- gsub("NUMPAG", i, url_base)

pagina <- read_html(url)

lista_tabelas <- html_table(pagina, header = TRUE)

print(lista_tabelas)
```

Simples não? Um pouco bagunçado, mas intuitivo. Geramos um endereço de URL e, com o endereço em mãos, capturamos o conteúdo da página usando o link como argumento da função _read\_html_ para, a seguir, extraírmos uma lista das tabelas da página. Antes de avançar, vamos falar um pouco sobre listas no R (pois o resultado da função _html\_table_ é uma lista). Podemos observar que o conteúdo também está um pouco bagunçado, trabalharemos um pouco mais nisso depois.

Note que a função _html\_table_ tem um segundo argumento, "header = TRUE". Esse argumento serve para indicarmos ao nosso "extrator de tabelas HTML" que a primeira linha da tabela deve ser considerada cabeçalho e, portanto, servirá de nome às colunas da tabela.

## Listas

Um detalhe fundamental do resultado da função _html\_table_ é que o resultado dela é uma lista. Por que uma lista? Porque pode haver mais de uma tabela na página e cada tabela ocupará uma posição na lista. Para o R, uma lista pode combinar objetos de diversas classes: vetores, data frames, matrizes, etc.

Ao rasparmos o site da ALESP, a função _html\_table_ retorna várias tabelas e não apenas a dos resultados das proposições, que é o que queremos.

Como acessar objetos em uma lista? Podemos ulitizar colchetes. Porém, se utilizarmos apenas um colchete, estamos obtendo uma sublista. Por exemplo, vamos criar diferentes objetos e combiná-los em uma lista:

```{r}
# Objetos variados
matriz <- matrix(c(1:6), nrow=2)
vetor.inteiros <- c(42:1)
vetor.texto <- c("a", "b", "c", "d", "e")
vetor.logico <- c(T, F, T, T, T, T, T, T, F)
texto <- "meu texto aleatorio"
resposta <- 42

# Lista
minha.lista <- list(matriz, vetor.inteiros, vetor.texto, vetor.logico, texto, resposta)
print(minha.lista)
```

Para produzirmos uma sublista, usamos um colchete (mesmo que a lista só tenha um elemento!):

```{r}
print(minha.lista[1:3])
class(minha.lista[1:3])
print(minha.lista[4])
class(minha.lista[4])
```

Se quisermos usar o objeto de uma lista, ou seja, extraí-lo da lista, devemos usar dois colchetes:

```{r}
print(minha.lista[[4]])
class(minha.lista[[4]])
```

Ao obtermos uma lista de tabelas de uma página (nem sempre vai parecer que todos os elementos são tabelas, mas são, pelo menos para um computador que "lê" HTML), devemos utilizar dois colchetes para extrair a tabela que queremos. 

Se lembra da lista com os países de maior área no mundo extraídos da Wikipedia? Podemos extrair, por exemplo, os países com mais de um milhão de metros quadrados de área em um _dataframe_ - o que normalmente manipulamos como base de dados em outros softwares. Para isso, criamos um novo objeto contendo apenas o conteúdo _[[1]]_ da nossa lista _tabelas_wiki_:

```{r}
gigantes <- tabelas_wiki[[1]]
head(gigantes)
```

Utilizando nossa pesquisa no site da ALESP, já sabemos que a tabela que queremos ocupa a posição 1 da lista, mas é necessário examinar sempre:

```{r}
url_base <- "https://www.al.sp.gov.br/alesp/pesquisa-proposicoes/?direction=acima&lastPage=177&currentPage=NUMPAG&act=detalhe&idDocumento=&rowsPerPage=20&currentPageDetalhe=1&tpDocumento=&method=search&text=previdencia&natureId=&legislativeNumber=&legislativeYear=&natureIdMainDoc=&anoDeExercicio=&strInitialDate=&strFinalDate=&author=&supporter=&politicalPartyId=&stageId="

i <- 1
url <- gsub("NUMPAG", i, url_base)

pagina <- read_html(url)

lista_tabelas <- html_table(pagina)

tabela <- lista_tabelas[[1]]

class(tabela)

View(tabela)
```

Bem mais bonito, não? No entanto, os dados estão bagunçados. Retomaremos adiante.


## Captura das tabelas

Podemos juntar tudo que vimos até agora: loop com a função "for", substituição com "gsub", captura de tabelas em HTML, listas e seus elementos.

Vamos tentar capturar as cinco primeiras páginas do resultado da pesquisa de proposições por meio da palavra-chave "previdencia". Para podermos saber que estamos capturando, vamos usar a função "head", que retorna as 6 primeiras linhas de um data frame, e a função "print".

Avance devagar neste ponto. Leia o código abaixo com calma e veja se entendeu o que acontece em cada linha. Já temos um primeiro script de captura de dados quase pronto e é importante estarmos seguros para avançar.

```{r}
url_base <- "https://www.al.sp.gov.br/alesp/pesquisa-proposicoes/?direction=acima&lastPage=177&currentPage=NUMPAG&act=detalhe&idDocumento=&rowsPerPage=20&currentPageDetalhe=1&tpDocumento=&method=search&text=previdencia&natureId=&legislativeNumber=&legislativeYear=&natureIdMainDoc=&anoDeExercicio=&strInitialDate=&strFinalDate=&author=&supporter=&politicalPartyId=&stageId="

for (i in 0:4) {
  
  url <- gsub("NUMPAG", i, url_base)
  
  pagina <- read_html(url)
  
  lista_tabelas <- html_table(pagina, header = TRUE)
  
  tabela <- lista_tabelas[[1]]
  
  print(head(tabela))
}
```

Vamos traduzir o que estamos fazendo: "para cada i de 0 a 4, vamos criar um link que é a combinação da URL base ('url_base') com i, vamos usar esta combinação ('url') como argumento da função _read\_html_ para capturar a página (e criar o objeto 'página'), vamos extrair a lista de tabelas da página com _html\_table_ (e criar a 'lista_tabelas'), vamos escolher a primeira tabela da lista (e criar 'tabela') e imprimir as 6 primeiras linhas da tabela de cada página".

Ufa! Leia de nova. Veja se entendeu o que aconteceu.

## Data Frames

Excelente, não? Mas e aí? Cadê os dados? O problema é que até agora ainda não fizemos nada com os dados, ou seja, ainda não guardamos eles em novos objetos para depois podermos utilizá-los na análise. Da mesma forma, vimos que as informações estão um pouco bagunçadas, não?

Neste último passo, vamos fazer o seguinte: precisamos de uma estrutura que armazene as informações, então criamos um data frame vazio (chamado "dados") e, para cada iteração no nosso loop (ou seja, para cada "i"), vamos inserir a tabela da página i como novas linhas no nosso data frame. A função nova que precisamos se chama _bind\_rows_, que é parte do pacote _dplyr_, protagonista do _tidyverse_. Ela serve para unir diferentes data frames (ou vetores ou matrizes), colocando suas linhas uma debaixo da outra. Vejamos um exemplo antes de avançar:

```{r}
# Criando 2 data frames separados
meus.dados1 <- data_frame("id" = 1:10, "Experimento" = rep(c("Tratamento"), 10))
print(meus.dados1)

meus.dados2 <- data_frame("id" = 11:20, "Experimento" = rep(c("Controle"), 10))
print(meus.dados2)

# Combinando os dois data.frames
meus.dados.completos <- bind_rows(meus.dados1, meus.dados2)
print(meus.dados.completos)
```

## Captura das tabelas com armazenamento em data frames

Pronto. Podemos agora criar um data frame vazio ("dados") e preenchê-lo com os dados capturados em cada iteração. O resultado final será um objeto com todas as tabelas de todas as páginas capturadas, que é o nosso objetivo central. 

Novamente vamos trabalhar apenas com as cinco primeiras páginas, mas bastaria alterar um único número para que o processo funcionasse para todas as páginas de resultados - desde que sua conexão de internet e a memória RAM do seu computador sejam boas! 

Obs: vamos inserir um "contador" das páginas capturadas com "print(i)". Isso será muito útil quando quisermos capturar um número grande de páginas, pois o contador nos dirá em qual iteração (sic, é sem "n" mesmo) do loop estamos.

```{r}
url_base <- "https://www.al.sp.gov.br/alesp/pesquisa-proposicoes/?direction=acima&lastPage=177&currentPage=NUMPAG&act=detalhe&idDocumento=&rowsPerPage=20&currentPageDetalhe=1&tpDocumento=&method=search&text=previdencia&natureId=&legislativeNumber=&legislativeYear=&natureIdMainDoc=&anoDeExercicio=&strInitialDate=&strFinalDate=&author=&supporter=&politicalPartyId=&stageId="

dados <- data.frame()

for (i in 0:4) {

  print(i)
  
  url <- gsub("NUMPAG", i, url_base)
  
  pagina <- read_html(url)
  
  lista_tabelas <- html_table(pagina, header = TRUE)
  
  tabela <- lista_tabelas[[1]]
  
  dados <- bind_rows(dados, tabela)
}
```

São 100 observações (5 páginas com 20 resultados) e 4 variáveis (Autor, Tipo de Proposta, Nome do Projeto, Ementa). São 4 variáveis  do tipo "character", mas precisamos checar se estão estruturadas. As 6 observações apresentam o resultado adequado, o que nos dá uma boa dica que que tudo ocorreu bem até a última página capturada.

Vamos observar o resultado utilizando a função "str" (abreviação de structure), que retorna a estrutura do data frame, e "tail", que é como a função "head", mas retorna as 6 últimas em vez das 6 primeiras observações.

```{r}
# Estrutura do data frame
str(dados)

# 6 primeiras observações
head(dados)

# 6 últimas observações
tail(dados)
```

Com a função _View_ (com V maiúsculo, algo raríssimo nas funções de R), podemos ver os dados em uma planilha.

```{r}
View(dados)
```

No entanto, vimos que nossas informações estão basntate bagunçadas, não? Vemos que o tipo de proposta (com sua data), o número do projeto e sua ementa estão em apenas uma coluna Da mesma forma, todas as linhas pares da tabela são "vazias" pela estrutura da tabela no HTML.

Primeiro vamos selecionar apenas as linhas ímpares, as que contém a informação que procuramos. Para isso, criaremos uma sequência com apenas os valores ímpares. Após isso, "filtramos" apenas as linhas dessa sequência dentre as 100 do nosso _dataframe_.

```{r}
seq_impar <- seq(1,100,2)
dados <- data.frame(dados[seq_impar,])
```

Simples, não? Agora vamos observar os separadores da nossa segunda coluna. Vamos extrair o conteúdo da segunda coluna na primeira linha da tabela utilizando separadores de texto. Como isso funciona? Vamos identificar o que separa o que separa o texto relacionado ao tipo de projeto (com data) ao nome do projeto.

```{r}
dados[1,2]
# Identificado o da primeira para a segunda coluna
# "\r\n \t\t\t\t\t\t\t\t-"
```

Identificado o separador, utilizaremos a função _separate_ do pacote _dplyr_, parte do _tidyverse_, utilizando os separadores criados acima. Também conheceremos os _pipes_ (%>%), as quais permitem a execução de mais de uma função em sequência para o mesmo _dataframe_ permitido nas aplicações do _tidyverse_.

```{r}
dados <- dados %>%
  separate(Documento, into = c("Tipo de Proposta", "Nome do Projeto"), sep = "\r\n \t\t\t\t\t\t\t\t-") 
```

No entanto algumas linhas não possuíam o "tipo de proposta", mas os *missing values* ficaram na variável "Nome do Projeto". Vamos utilizar as funções do _dplyr_ _filter_ para selecionar as linhas em que temos ou não o problema, e, no fim, corrigir com _mutate_ e juntar novamente em um único _dataframe_:

```{r}
dados2 <- dados %>%
  filter(!is.na(`Nome do Projeto`)) 
# ! indica "diferente", is.na() que é um missing value
dados <- dados %>%
  filter(is.na(`Nome do Projeto`)) %>%
  mutate(`Nome do Projeto` =`Tipo de Proposta`,
         `Tipo de Proposta` = NA) %>%
  bind_rows(dados2)
```

A próxima etapa é separar o "Nome do Projeto" de sua ementa. Realizaremos o mesmo processo com _separate_, mas com o segundo separador:

```{r}
dados[1,3]
# "\r\n \t\t\t\t\t\t \r\n \t\t\t\t\t\t \r\n \t\t\t\t\t\t"

dados <- dados %>%
  separate("Nome do Projeto", into = c("Nome do Projeto", "Ementa"), sep = "\r\n \t\t\t\t\t\t \r\n \t\t\t\t\t\t \r\n \t\t\t\t\t\t") 
```

Pronto! Conseguimos fazer nossa primeira captura de dados. Se quiser, você pode repetir o procedimento para pegar o resto dos resultados ou reproduzir o mesmo processo para capturar outras informações do site.
 
# Desafio

Passamos por um tutorial bastante puxado, não? Agora podemos explorar o site de outra Assembleia Estadual, a do [Amapá](http://www.al.ap.gov.br/pagina.php?pg=buscar_proposicao) e buscar proposições diveras. Escolha uma temática que torne muito onerosa a captura manual e divirta-se. 

Obs: Preste atenção nas informações que a tabela nos oferece. Lembre-se que é uma atividade "artesanal".
