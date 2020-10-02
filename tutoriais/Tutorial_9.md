# Tutorial 9: Modelagem com exemplos do próprio pacote *quanteda*

## Outros exemplos

### Ainda nos dicionários: afinidade

Para fins didáticos, a partir de agora utilizaremos alguns dos exemplos fora dos nossos dados do DataFolha. São materiais de aprendizagem do próprio pacote *quanteda*. 

Um exemplo de dicionário externo é o Affective Norms for English Words (AFINN), com o qual podemos analisar o teor do discurso de alguns discursos de posse de presidentes americanos (exemplo que utilizaremos a partir de agora). O dicionario faz parte do pacote *quanteda.dictionaries*, enquanto o corpus que utilizaremos e do próprio pacote *quanteda*.

```{r}
summary(data_corpus_inaugural)
```

Vemos que o corpus contém 58 discursos de posse presidencial dos EUA. Mas vamos aplicar o dicionário de afinidade para ver a conotação positiva e negativa de palavras em cada um deles para as últimas 5 posses.


```{r, eval = FALSE}
dic_aff <- dictionary(quanteda.dictionaries::data_dictionary_AFINN)
dfmat_inaug_subset <- dfm(data_corpus_inaugural[54:58], dictionary = dic_aff)
dfmat_inaug_subset
```

### Semelhança entre textos

Para ver a semelhança entre os textos, utilizaremos apenas um subconjunto de discursos de posse depois de 1980. A partir dele, vamos comparar os discursos de posse do ex-presidente Obama com os demais e entre eles próprios 

```{r fig.width = 6, fig.height = 3}
dfmat_inaug_post1980 <- dfm(corpus_subset(data_corpus_inaugural, Year > 1980),
                            remove = stopwords("english"), stem = TRUE, remove_punct = TRUE)
```

Agora realizamos o teste de similaridade aplicado a esses discursos com o [método de cosseno](https://sites.temple.edu/tudsc/2017/03/30/measuring-similarity-between-texts-in-python/#:~:text=The%20cosine%20similarity%20is%20the,the%20similarity%20between%20two%20documents.). Caso queiram pesquisar sobre os métodos aplicados, olhem a [documentação da função](https://www.rdocumentation.org/packages/quanteda/versions/2.1.1/topics/textstat_simil).

```{r fig.width = 6, fig.height = 3}
tstat_obama <- textstat_simil(dfmat_inaug_post1980,
                              dfmat_inaug_post1980[c("2009-Obama", "2013-Obama"), ],
                              margin = "documents", method = "cosine")

tstat_obama
```

De forma intuitiva, quanto maior o valor (de 0 a 1), mais semelhantes são os discursos. Podemos observar isso graficamente

```{r fig.width = 6, fig.height = 3}
dotchart(as.list(tstat_obama)$"2013-Obama", xlab = "Similaridade de cosseno (Obama 2013)", pch = 19)
```

Outra possibilidade é a utilização dessas distâncias para produzir um dendograma clusterizando os presidentes. Primeiro carregamos os dados para o exemplo e criamos uma *dfm* com os discursos a partir de 1980.

```{r, eval = FALSE}
data_corpus_sotu <- readRDS(url("https://quanteda.org/data/data_corpus_sotu.rds"))
dfmat_sotu <- dfm(corpus_subset(data_corpus_sotu, Date > as.Date("1980-01-01")),
                  stem = TRUE, remove_punct = TRUE,
                  remove = stopwords("english"))
```

Em um segundo momento, utilizamos a funçao `dfm_trim()` para selecionar os recursos a partir de um "corte" na frequência. Aqui pegamos somente termos que apareçam ao menos 5 vezes e em três documentos. 

```{r, eval = FALSE}
dfmat_sotu <- dfm_trim(dfmat_sotu, min_termfreq = 5, min_docfreq = 3)

dfmat_sotu
```
Agora obtemos as distâncias da *dfm* normalizadas.

```{r}
tstat_dist <- textstat_dist(dfm_weight(dfmat_sotu, scheme = "prop"))
```

Em seguida, realizamos a clusterização hirárquica do objeto da distância:

```{r}
pres_cluster <- hclust(as.dist(tstat_dist))
```

E atribuímos rótulos com os nomes dos documentos:

```{r}
pres_cluster$labels <- docnames(dfmat_sotu)
```

Por fim, plotamos o dendograma.

```{r, fig.width = 8, fig.height = 5}
plot(pres_cluster, ylab = "", xlab = "", sub = "",
     main = "Distância Euclidiana da frequência de tokens normalizada")
```

E também podemos procurar pela similaridade de termos.

```{r}
tstat_sim <- textstat_simil(dfmat_sotu, dfmat_sotu[, c("fair", "health", "terror")],
                          method = "cosine", margin = "features")
lapply(as.list(tstat_sim), head, 10)
```

### Posições de Documentos

Aqui temos um exemplo de dimensionamento da posição de documentos não supervisionada utilizando um corpus com a discussão do orçamento irlandês de 2010. É o Modelo "Wordfish" citado na apresentação de ontem. Para exemplos de modelagem de textos observem a [Documentação do Pacote *quanteda.textmodels*](https://www.rdocumentation.org/packages/quanteda.textmodels/versions/0.9.1). 

```{r fig.width = 7, fig.height = 5}
if (require("quanteda.textmodels")) {
  dfmat_ire <- dfm(data_corpus_irishbudget2010)
  tmod_wf <- textmodel_wordfish(dfmat_ire, dir = c(2, 1))
  
  # plot the Wordfish estimates by party
  textplot_scale1d(tmod_wf, groups = docvars(dfmat_ire, "party"))
}
```

### Topic models

O *quanteda* oferece boas ferrametnas para *topic models*. Aqui retomamos os dados do DataFolha. Mas utilizaremos apenas o subconjunto de 2010 a 2020. Assim como fizemos anteriormente, aplicaremos a função `dfm_trim()` para realizar "cortes" nos nossos recursos. 

```{r}
corpus_noticias_2010_2020 <- corpus_subset(corpus_noticias, ano >= 2010)

dfm_noticias_2010_2020 <- dfm(corpus_noticias_2010_2020,
                               remove = c("é", stopwords("pt")),
                               stem = TRUE, remove_punct = TRUE,
                               remove_numbers = TRUE)


quant_dfm <- dfm_trim(dfm_noticias_2010_2020, min_termfreq = 3, min_docfreq = 5)

quant_dfm
```

Agora podemos estimar um [*Structural Topic Model*](https://www.rdocumentation.org/packages/stm/versions/1.3.6/topics/stm) com o pacote `stm` e a função `stm()`. Vamos utilizar um exemplo de 10 tópicos, representado na opção `K` da função. Utilizamos o valor `FALSE` na opção `verbose` para que não imprima na tela as iterações realizadas durante os cálculos para modelagem realizados pela a função - a opção padrão é `TRUE`, a utilize se quiser observar cada um dos passos no console do seu RStudio.

```{r fig.width = 7, fig.height = 5}
set.seed(123)
if (require("stm")) {
    my_lda_fit10 <- stm(dfm_noticias_2018_2020, K = 10, verbose = FALSE)
    plot(my_lda_fit20, main = "Principais Tópicos", xlab = "Proporções esperadas dos tópicos")    
}
```
