# Aula 04 - Manipulação
Curso de R: Do casual ao avançado  
2015-01-26  

## Hadley Wickham

<img src="figure/hadley.png" style="heigth: 300px;"/>

## Manipulação de dados com dplyr

A manipulação de dados é uma tarefa usualmente bastante
dolorosa e demorada, podendo muitas vezes tomar mais tempo do que desejaríamos. No entanto,
como nosso interesse geralmente é na modelagem dos dados, essa tarefa é muitas vezes negligenciada.

## Introdução

O `dplyr` é um dos pacotes mais úteis para realizar manipulação de dados, e procura aliar 
simplicidade e eficiência de uma forma bastante elegante. Os scripts em `R` que fazem uso 
inteligente dos verbos `dplyr` e as facilidades do operador _pipe_ tendem a ficar mais legíveis e 
organizados, sem perder velocidade de execução.

## Introdução

Por ser um pacote que se propõe a realizar um dos trabalhos mais árduos da análise estatística,
e por atingir esse objetivo de forma elegante, eficaz e eficiente, o `dplyr` pode ser considerado 
como uma revolução no `R`.


```r
library(dplyr)
data(pnud_muni, package = 'abjutils')
```

## Introdução

### Trabalhando com tbl e tbl_df


```r
pnud_muni <- tbl_df(pnud_muni)
pnud_muni
```

## Introdução

### Filosofia do Hadley para análise de dados

<img src="figure/hadley_view.png" style="width: 800px;"/>

## Introdução

### As cinco funções principais do dplyr

- `select`
- `filter`
- `mutate`
- `arrange`
- `summarise`

## Introdução

### Características

- O _input_  é sempre um `data.frame` (`tbl`), e o _output_  é sempre um `data.frame` (`tbl`).
- No primeiro argumento colocamos o `data.frame`, e nos outros argumentos colocamo o que queremos fazer.
- A utilização é facilitada com o emprego do operador `%>%`

### Vantagens

- Utiliza `C` e `C++` por trás da maioria das funções, o que geralmente torna o código mais eficiente.
- Pode trabalhar com diferentes fontes de dados, como bases relacionais (SQL) e `data.table`.

## select {.build}

- Utilizar `starts_with(x)`, `contains(x)`, `matches(x)`, `one_of(x)`, etc.
- Colocar nomes, índices, e intervalos de variáveis com `:`.

## select {.build}


```r
# por indice (nao recomendavel!)
pnud_muni %>%
  select(1:10)
```

## select {.build}


```r
# especificando nomes (maneira mais usual)
pnud_muni %>%
  select(ano, uf, municipio, idhm)
```

## select {.build}


```r
# intervalos e funcoes auxiliares (para economizar trabalho)
pnud_muni %>%
  select(ano:municipio, starts_with('idhm'))
```

## Exercício

Selecione município, estado, ano, coeficiente de gini e todas as medidas de idhm.

## filter

- Parecido com `subset`.
- Condições separadas por vírgulas é o mesmo que separar por `&`.

## filter


```r
# somente estado de SP, com IDH municipal maior que 80% no ano 2010
pnud_muni %>%
  select(ano, ufn, municipio, idhm) %>%
  filter(ufn==35, idhm > .8, ano==2010)
```

## filter


```r
# mesma coisa que o anterior
pnud_muni %>%
  select(ano, ufn, municipio, idhm) %>%
  filter(ufn==35 & idhm > .8 & ano==2010)
```

## filter


```r
# !is.na(x)
pnud_muni %>%
  select(ano, ufn, municipio, idhm, pea) %>%
  filter(!is.na(pea))
```

## filter


```r
# %in%
pnud_muni %>%
  select(ano, ufn, municipio, idhm) %>%
  filter(municipio %in% c('CAMPINAS', 'SÃO PAULO'))
```

## Exercício

Selecione ano, município, ufn, gini e as medidas de idh, e depois filtre apenas para os
casos em que o ano é 2010, o coeficiente de Gini é maior que 0.5 ou o idhm é maior que 0.7

## mutate

- Parecido com `transform`, mas aceita várias novas colunas iterativamente.
- Novas variáveis devem ter o mesmo `length` que o `nrow` do bd oridinal ou `1`.

## mutate


```r
# media de idhm_l e idhm_e
pnud_muni %>%
  select(ano, ufn, municipio, starts_with('idhm')) %>%
  filter(ano==2010) %>%
  mutate(idhm2 = (idhm_E + idhm_L)/2)
```

## mutate


```r
# errado
pnud_muni %>%
  select(ano, ufn, municipio, starts_with('idhm')) %>%
  filter(ano==2010) %>%
  mutate(idhm2 = mean(c(idhm_E, idhm_L)))

# uma alternativa (+ demorada)
pnud_muni %>%
  select(ano, ufn, municipio, starts_with('idhm')) %>%
  filter(ano==2010) %>%
  rowwise %>%
  mutate(idhm2 = mean(c(idhm_E, idhm_L)))
```

## Exercício

Selecione ano, município, ufn, gini e as medidas de idh, depois filtre apenas para os
casos em que o ano é 2010, e depois escreva o idhm em forma de porcentagem, com 1 casa decimal

## arrange

- Simplesmente ordena de acordo com as opções.
- Utilizar `desc` para ordem decrescente.

## arrange


```r
pnud_muni %>%
  select(ano, ufn, municipio, idhm) %>%
  filter(ano==2010) %>%
  mutate(idhm_porc = idhm * 100,
         idhm_porc_txt = paste(idhm_porc, '%')) %>%
  arrange(idhm)
```

## arrange


```r
pnud_muni %>%
  select(ano, ufn, municipio, idhm) %>%
  filter(ano==2010) %>%
  mutate(idhm_porc = idhm * 100,
         idhm_porc_txt = paste(idhm_porc, '%')) %>%
  arrange(desc(idhm))
```

## Exercício

Obtenha os 10 municipios com maior idhm em 2010 e mostre esses idhm./

## summarise

- Retorna um vetor de tamanho `1` a partir de uma conta com as variáveis.
- Geralmente é utilizado em conjunto com `group_by`.
- Algumas funções importantes: `n()`, `n_distinct()`.

## summarise


```r
pnud_muni %>%
  filter(ano==2010) %>%  
  group_by(ufn) %>%
  summarise(n=n(), 
            idhm_medio=mean(idhm),
            populacao_total=sum(popt)) %>%
  arrange(desc(idhm_medio))
```

## summarise


```r
pnud_muni %>%
  filter(ano==2010) %>%  
  count(ufn)
```

## summarise


```r
pnud_muni %>%
  group_by(ano, ufn) %>%
  tally() %>%
  head # nao precisa de parenteses!
```

## Exercício

Calcule a expectativa de vida média de cada estado, ponderada pela população dos municípios no ano 2000.

## Data Tidying com tidyr

- Cada observação é uma linha do bd.
- Cada variável é uma coluna do bd.
- Para cada unidade observacional temos um bd separado (possivelmente com chaves de associacao).


```r
library(tidyr)
```

## spread

- "Joga" uma variável nas colunas


```r
pnud_muni %>%
  group_by(ano, ufn) %>%
  summarise(populacao=sum(popt)) %>%
  ungroup() %>%
  spread(ano, populacao)
```

## gather

- "Empilha" o banco de dados


```r
pnud_muni %>%
  filter(ano==2010) %>%
  select(ufn, municipio, starts_with('idhm_')) %>%
  gather(tipo_idh, idh, starts_with('idhm_'))
```

## Exercício

Verifique se `gather(spread(dados))`, `spread(gather(dados))` e `dados` são equivalentes.

## Funções auxiliares

- `unite` junta duas ou mais colunas usando algum separador (`_`, por exemplo).
- `separate` faz o inverso de `unite`, e uma coluna em várias usando um separador.

## Um pouco mais de manipulação de dados

- Para juntar tabelas, usar `inner_join`, `left_join`, `anti_join`, etc.
- Para realizar operações mais gerais, usar `do`.
- Para retirar duplicatas, utilizar `distinct`.

## Outros pacotes úteis para limpar bases de dados

- `stringr` para trabalhar com textos.
- `lubridate` para trabalhar com datas.
- `rvest` para trabalhar com arquivos HTML.

<hr/ >
