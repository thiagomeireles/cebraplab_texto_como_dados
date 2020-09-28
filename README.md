# Textos como dados: raspagem e mineração de dados em R" do cebrap.lab

## Informações básicas

### Instrutor: 
	
[Thiago Meireles](https://thiagomeireles.github.io/)

### Data, Hora e Local

De 28 de setembro a 2 de outubro de 2020, das 18h30 às 21h30.

Os encontros serão nos dias 28 e 30 de setembro e 02 de outubro via Zoom.

## Apresentação

O laboratório apresenta as principais ferramentas de captura de dados na Internet e análise quantitativa de texto utilizando R. Além de ser um software livre voltado para estatística computacional e análise de dados, R é uma linguagem focada na aplicação de funções que, entre outras possibilidades, permite a captura de dados de forma automatizada na internet.  A partir de informações disponíveis em portais de notícias, apresentaremos esse processo de raspagem de dados de páginas web (especialmente de tabelas e de páginas construídas em html) e construção de bases de dados com textos de Internet tratados como informações quantitativas, o que permitirá introduzir algumas das práticas de mineração de texto. Faremos um exercício empírico partindo de uma questão de pesquisa que conduzirá a experimentação, de forma a capacitar os participantes com ferramentas e procedimentos que depois poderão ser usadas para a construção de suas próprias bases de dados. Para participação no curso, espera-se conhecimento prévio da linguagem R ou uma preparação de nivelamento por meio de tutoriais indicados antes do início das aulas.

Esse repositório será alimentado ao longo do curso com roteiros de aula e tutoriais atualizados tentado atender as particularidades da turma.

### Dinâmica das aulas

As aulas terão conteúdo expositivo sobre conceitos e ferramentas básicas utilizados durante o curso, mas a maior parte do tempo será dedicada à realização de tutoriais assistidos. Trabalharemos em dupla, cada um em seu computador. O professor acompanhará o andamento de cada dupla, tirando as dúvidas (sim, elas surgirão).

Não esqueçam de preencher a planilha enviada por e-mail e disponível [aqui](https://docs.google.com/spreadsheets/d/16-hL0y4DJnrzVtDJyGCRqdJTn2pldK2tzs7u3_jW1RY/edit) indicando seu nível de R entre: **nunca usei**; **usei pouco ou há muito tempo**; ou **utilizo com frequência**. Isso será usado para a formação das duplas.

### Presença e avaliação

Em todos os roteiros teremos links para a sala do Zoom e para a lista virtual.

O requisito para a emissão de certificado é a presença em dois dos três encontros virtuais.

No entanto, ressalto a importância das atividades de terça e quinta-feira. O primeiro por ser um desafio de colocar em prática com seu material o que veremos na segunda-feira. O segundo por dar uma visão mais ampla sobre *text mining* com os quais trabalharemos no último encontro.

## Requisitos

### Preparação
A participação no curso requer uma exposição prévia à linguagem R e ao ambiente de tabalho do RStudio.

Caso não tenha nenhum contato com a linguagem, é mandatória a realização de um [tutorial de preparação](https://www.datacamp.com/courses/free-introduction-to-r) antes do início das aulas. 

Ainda que tenha conhecimento básico das estruturas da linguagem, é fortemente recomendado que tambem o façam.

O tempo estimado para o tutorial é de *aproximadamente 4 horas*.

### Equipamento

Como a maior parte do curso é baseada em tutoriais em que vocês aprenderão "colocando a mão na massa", é mandatório que acompanhem as aulas no computador.

### Softwares

Foi preparado um [Roteiro pré-curso](https://github.com/thiagomeireles/cebraplab_texto_como_dados/blob/master/roteiros/instalacao.md) onde estão as instruções para a instalação dos softwares necessários.

## Objetivos

Os participantes, ao fim do curso, serão capazes de:
- Coletar dados de sites de estrutura mais simples, como jornais e legislativos brasileiros;
- Realizar tarefas relacionadas a mineração de texto a partir de diferentes abordagens
- Produzir gráficos e grafos mais simples a partir dos dados coletados
- Entender e aplicar conceitos básicos de *text mining*

## Roteiros e tutoriais

### Roteiros

Todas os dias de curso terão roteiros a cumprir. Pouco antes de cada encontro, as linhas abaixo serão preenchidas com links com as descrições do que esperamos em cada dia de curso e como o faremos.

[28/09/2020](https://github.com/thiagomeireles/cebraplab_texto_como_dados/blob/master/roteiros/dia_01.md) - O básico da raspagem de dados

30/09/2020 - Desafios de raspagem de dados

01/10/2020 - Introdução à manipulação de textos como dados

02/10/2020 - A pesquisa com *text mining*

03/10/2020 - *Text mining* em R

### Tutoriais

Os links para os tutoriais estarão abaixo antes de cada aula.

[Tutorial 1](https://github.com/thiagomeireles/cebraplab_texto_como_dados/blob/master/tutoriais/Tutorial_1.md): Páginas com tabelas

[Tutorial 2](https://github.com/thiagomeireles/cebraplab_texto_como_dados/blob/master/tutoriais/Tutorial_2.md): Realizar a extração de qualquer conteúdo de uma página utilizando os "caminhos" dos elementos da página no código html - Introdução ao XPath

[Tutorial 3](https://github.com/thiagomeireles/cebraplab_texto_como_dados/blob/master/tutoriais/Tutorial_3.md): Extrair informações de uma sequência páginas (ex. portal de notícias) - Captura de notícias da Folha

[Tutorial 4] Captura de notícias do Data Folha

[Tutorial 5] Mineração de Texto - pacote stringr

[Tutorial 6] Mineração de Texto - pacote tm

[Tutorial 7] Mineração de Texto - pacote tidytext

[Tutorial 8] Texto como dados e o pacote quanteda

## Referências

- Grolemund, Garrett (2014). Hands-On Programming with R. Ed: O'Reilly Media. Não distribuído gratuitamente. Informações no site da editora [aqui](http://shop.oreilly.com/product/0636920028574.do)
- Wichkam, Hadley e Grolemund, Garrett (2016). R for Data Science. Ed: O'Reilly Media. Disponível gratuitamente Disponível gratuitamente [aqui](http://r4ds.had.co.nz/data-visualisation.html)
- Wichkam, Hadley (2014). Advanced R. Ed: Chapman and Hall/CRC. Disponível gratuitamente Disponível gratuitamente [aqui](http://adv-r.had.co.nz/)
- Gillespie, Colin e Lovelace, Robin (2016). Efficient R programming. Ed: O'Reilly Media. Disponível gratuitamente Disponível gratuitamente [aqui](https://csgillespie.github.io/efficientR/)

