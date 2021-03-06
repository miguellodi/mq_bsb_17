# Tutorial D1 - Introdução ao Aprendizado de Máquina

O Aprendizado de Máquina é um conjunto de técnicas de estatísticas usado para a construção de modelos de classificação, previsão e agrupamento de dados. Chamamos este conjunto de técnicas de aprendizado de máquina pois estamos trabalhando com algoritmos que "aprendem" com os dados para melhorar performance da tarefa.

Os algoritmos de aprendizado de máquina são separados em duas categorias: supervisionado e não-supervisionado. Os algoritmos supervisionados são aqueles em que há uma variável alvo, ou variável resposta, que será objetivo de previsão ou classificação. Os algoritmos não-supervisionados, por sua vez, servem para encontrar padrões e estrutura nos dados.

É importante pensar também no que aprendizado de máquina não é. Calcular frequências de características em uma população ou contruir indicadores e medidas a partir de características observadas não são objetivos do aprendizado de máquinas.

Neste tutorial veremos 4 dos algoritmos mais populares de aprendizado de máquina: Regressão Linear, Árvore de Decisão, Kmeans Cluster e Cluster Hierárquico. O caráter da atividade é introdutório, mas serve de guia para a identificação de problemas de aprendizado de máquina.

Diferentemente dos demais tópicos da apostila, cujo avanço nos tutoriais de maneira desacompanhada é possível, neste tópico trabalharemos a apresentação de sala de aula e apostila de maneira coordenada. Antes de começarmos a usar cada um dos algoritmos, teremos uma breve exposição sobre seu funcionamento.

Como utilizaremos dados dos exemplos clássicos da literatura, que são de pouco interesse substantivo para nós mas oferecem vantagens didáticas, ao final da apresentação de cada algoritmo faremos uma breve atividade de treino com os dados do Programa Bolsa Família e CADUNICO.

## Antes de começar

Neste tutorial vamos trabalhar com bases de dados de livro texto e bastante conhecidas na literatura de aprendizado de máquina para depois elaborar pequenos testes com os dados do MDS referentes ao Programa Bolsa Família e CADUNICO (PBF-CADUNICO).

Os dados do PBF-CADUNICO estão disponíveis no material do curso em formato .sav, que é o formato de dados do software SPSS.

Para abrí-los, vamos utilizar a função _read\_spss_, disponível no pacote _haven_:

```{r}
library(haven)

pbf_cadunico <- read_spss("/home/acsa/Desktop/PBF_cadastrounico_condicionalidades_pessoas_semID.sav")
glimpse(pbf_cadunico)

```

Veja que á 110 variáveis na base dados. Para fins didáticos, vamos selecionar algumas poucas variáveis e recodificá-las. Ficaremos com as seguintes variáveis: pbf, que indica se o indivíduo é beneficiário ou não do PBF; sexo, sendo 0 para mulheres e 1 para homens; idade, medida em anos; cor, codificada 1 para brancos e 0 para não brancos; ler, que indica se o indivíduo é alfabetizado (1) ou não (0); renda média familiar, medida em reais; e água, que indica se o domicílio em que a pesso reside tem água encanada ou não. Também manteremos apenas os indivíduos com mais de 12 anos de idade.

As alterações relizadas nos dados originais estão abaixo:

```{r}
library(lubridate)
pbf_cadunico <- pbf_cadunico %>%
  rename(pbf = MARC_PBF, 
         sexo = CO_SEXO_PESSOA, 
         nasc = DT_NASC_PESSOA,
         cor_cat = CO_RACA_COR_PESSOA,
         ler = CO_SABE_LER_ESCREVER_MEMB,
         renda = VL_RENDA_MEDIA_FAM,
         agua = CO_AGUA_CANALIZADA_FAM) %>%
  mutate(pbf = factor(pbf),
         sexo = factor(replace(sexo, sexo == 2, 0)),
         idade =  as.numeric(round((ymd("2017-04-01") - ymd(nasc)) / 365)),
         cor = 0, cor = replace(cor, cor_cat == 1, 1), 
         cor = factor(cor),
         ler = replace(ler, ler == 2, 0), 
         ler = replace(ler, is.nan(ler), 0), 
         ler = factor(ler),
         agua = replace(agua, agua == 2, 0), 
         agua = replace(agua, agua == 3, 0), 
         agua = replace(agua, is.nan(agua), 0), 
         agua = factor(agua))%>%
  select(pbf, sexo, idade, cor, ler, renda, agua) %>%
  filter(idade >= 12)
```

Vamos deixar estes dados de lado e voltaremos a ele nas atividades do tutorial.

## Algoritmos de previsão - regressão linear simples

Regressão linear, ou mínimos quadrados ordinários, é uma das técnicas estísticas de maior popularidade em vários campos científicos, inclusive nas humanidades, e muito utilizada para avaliação de políticas públicas.

Nosso objetivo neste tópico não será discutir em profundidade modelos lineares, mas apresentar como podemos utilizá-los para fazer previsões. Em particular, tentaremos prever uma quantidade (variável contínua) a partir de uma ou mais variáveis.

Nosso primeiro exemplo de livro texto será com os _data frame_ Wage, disponível no pacote ISLR. Baixe o pacote (se não estiver instalado em seu computador), carregue-o e use a função _data_ para tornar "Wage" disponível. Utilize a função _glimpse_ do pacote _dplyr_ para conhecer os dados.

```{r}
install.packages("ISLR")
library(ISLR)
data("Wage")
glimpse(Wage)
```

Nosso objetivo será tentar prever a variável "wage" a partir da idade "age" dos indivíduos. A fórmula que representa este problema é: _wage ~ age_. Para construir o modelo, utilizamos a funçao _lm_ que recebe a fórmula do modelo como primeiro argumento. "data" é o argumento utilizado para indicar em qual _data frame_ as variáveis estão.

```{r}
modelo <- lm(wage ~ age, data = Wage)
```

Utilize a função _summary_ para conhecer o resultado do modelo:

```{r}
summary(modelo)
```

Vamos deixar a interpretação estatística de lado, mas vamos explorar os coeficientes estimados ("Estimate"). _(Intercept)_ indica o valor esperado no modelo para um indivíduo de idade igual a 0 (ainda que isso seja impossível). Ou seja, este é o ponto em que a reta de regressão encontra com o eixo "y". Veja graficamente o resultado com o comando abaixo. Note que quando trabalhamos com duas dimensões, como estamos fazendo, a reta é a representação gráfica do modelo.

```{r}
plot(Wage$age, Wage$wage, pch = 3)
abline(a = 81.70474, b = 0.70728, col = "red", lwd = 3)
```

Voltando ao modelo estimado, vemos que a estimativa para "age" é 0.7073. Este coeficiente indica que a reta tem inclinação tal que, para cada acréscimo de um ano, espera-se que o salário aumente em 0.7073. Ou seja, idade e salário têm entre si uma relação positiva.

Quão bom é nosso modelo? Em outras palavras, o modelo consegue prever adequadamente o salário a partir da idade? Bons modelos de previsão são aqueles cujos valores previstos. O que acabamos de produzir não parece ser muito bom. Para cada valor de idade, há uma variação muito grande nos valores observados de salário e vários deles bastante afastados da reta que representa nosso modelo. Podemos produir um vetor dos valores previstos com o comando _predict_ e tentar avaliar qual é a qualidade do modelo:

```{r}
prev <- predict(modelo)
```

Volte ao gráfico. Qual é o valor esperado (salário) para uma pessoa de 50 anos de idade? Basta olhar o ponto da reta que cruza com uma linha vertical no valor 50 do eixo horizontal. _predict_ gera um vetor de valores previstos sem precisarmos recorrer à inspeção gráfica.

Ao utilizarmos _predict_ como acima, estamos gerando os valores previstos para os indivíduos para os quais conhecemos os valores reais. E se, porém, quisermos calcular o valor previsto para indivíduos cuja idade conhecemos, mas não sabemos nada sobre seu salário? Vamos gerar um novo _data frame_ com a idade de indivíduos que não estavam nos dados originais:

```{r}
dados_novos <- data.frame(age = c(35, 50, 18, 73))
```

A seguir, vamos repetir o comando _predict_, mas agora indicando o argumento "newdata", que receberá os dados das observações para as quais não conhecemos o salário:

```{r}
prev_novo <- predict(modelo, newdata = dados_novos)
print(prev_novo)
```

O roteiro básico que acabamos de seguir é uma das técnicas mais utilizadas de aprendizado de máquina supervisionado. Em primeiro lugar, usamos os dados disponíveis, para os quais conhecemos a variável resposta (salário), e produzirmos um modelo. A seguir, utilizamos os dados para fazer previsões sobre observações para as quais não temos informação sobre os valores da variável resposta.

Como saber se o modelo está suficientemente bom sem precisar coletar novos dados além dos dados de nossa amostra? Se queremos prover a variável resposta para observações novas e para as quais não queremos ou podemos coletar o valor desta variável, nunca saberemos se o modelo está prevendo adequadamente ou não. O "segredo" utilizado em aprendizado de máquina é dividir a amostra em duas partes, uma para "treino" do modelo e outra para "teste" do modelo.

Ao separarmos os dados da amostra em duas parte, podemos treinar um modelo e testá-lo na sequência com dados __diferentes__ dos que foram utilizados para construí-lo. Se o modelo for capaz de prever adequadamente os dados que não fizeram parte de sua estimação, então será um bom modelo.

Ao separar em duas partes a amostra, temos que garantir que "treino" e "teste" são parecidos. Podemos fazer isso sorteando quais observações vão para qual _data frame_. Utilizamos a função "sample" para produzir uma amostra sem reposição do tamanho dos dados originais. Na prática, o resultado é uma versão "embaralhada"" dos dados originais.

```{r}
n <- nrow(Wage)
wage2 <- Wage[sample(n),]
```

Uma vez que os dados estão embaralhados, podemos calcular qual será o tamanho de cada parte, treino e teste. Uma prática comum é manter mais dados para o treino do que para o teste do modelo e 75/% é uma conveção utilizada, mas que não precisa ser seguida.

```{r}
n_treino <- round(0.75 * n)
treino <- wage2[1:n_treino,]
teste <- wage2[(n_treino + 1):n,]
```

Veja que agora utilizaremos "treino" como argumento "data" para gerar nosso novo modelo.

```{r}
modelo_treino <- lm(wage ~ age, data = treino)
```

E ao calcular os valores previstos, utilizaremos "teste" como argumento "newdata", já que os dados em "teste" não foram utilizados para a construção do modelo.

```{r}
prev_teste <- predict(modelo_treino, newdata = teste)
```

Diferentemente do exemplo anterior, temos agora um conjunto de dados (teste) para os quais temos o valor observado do salário e também o valor previsto a partir de um modelo construído (treinado) a partir de outros dados.

A separação entre treino e teste nos permite avaliar a qualidade do modelo sem precisarmos coletar dados adicionais e com prejuízo pequeno no número de observações (no exemplo, apenas um quarto).

A medida mais comumente utilizada para avaliar a qualidade de um modelo de regressão para precisão é a "raiz dos erros quadrados médios" (RMSE - Root Mean Squared Error). Isso nada mais é do que calcular a diferença entre o valor previsto e o valor real nos dados de teste, elevá-los ao quadrado, tirar a média e, ao final, tirar a raiz quadrada da média. Em linguagem R fazemos:

```{r}
rmse <- sqrt(mean((teste$wage - prev_teste)^2))
print(rmse)
```

## Algoritmos de previsão - regressão linear múltipla

No nosso exemplo anterior, utilizamos apenas uma variável (age) para prever salário. Podemos, porém, utilizar mais variáveis para melhorar nosso modelo.

Vamos utilizar como exemplo o _data frame_ _diamonds_, que faz parte do pacote _ggplot2_. _diamonds_ contém dados sobre as características de diamantes e queremos prever seu preço a partir de tais características. Comecemos carregando os dados:

```{r}
library(ggplot2)
data("diamonds")
glimpse(diamonds)
```

A mecânica de uma regressão com várias variáveis é semelhante à que vimos acima. A diferença está apenas na fórmula e em detalhes de sua interpretação que não nos interessam para fins de previsão.

Se quisermos prever o preço ("price") a partir das variáveis "depth" e "carat", escrevemos a seguinte fórmula: "price ~ depth + carat". Se, por outro lado, quisermos prever o preço a partir de todas as demais variáveis, basta fazer "price ~ .", onde o ponto representa "todas as demais variáveis".

Veja que algumas dessas variáveis são categórias e estão codificadas como factor. Não há problema: a função _lm_ automaticamente lida com este problema transformado todas em variáveis binárias de acordo com o número de categorias. Veja o exemplo:

```{r}
model <- lm(price ~ ., diamonds)
summary(model)
```

Uma vez que sabemos estimar um modelo linear com várias variáveis, podemos repetir o procedimento de separar os dados em "treino" e "teste":

```{r}
n <- nrow(diamonds)
n_treino <- round(0.75 * n)
diamonds <- diamonds[sample(n),]
treino <- diamonds[1:n_treino,]
teste <- diamonds[(n_treino + 1):n,]
```

E, finalmente, estimar o modelo com os dados de treino, fazer a previsão dos dados de teste e  

```{r}
modelo <- lm(price ~ ., treino)
prev_teste <- predict(modelo, teste)
```
Vamos ver num gráfico como que a estimativa do preço do diamante dada pelo modelo nos dados de treino se aproxima do dados de teste:

```{r}
plot(prev_teste, teste$price)
```

## Atividade com dados do MDS

Vamos utilizar os dados do _data frame_ "pbf\_cadunico" para treinar o que acabamos de aprender. Nosso objetivo será prever a renda de um indivíduo a partir da demais características. Faremos em conjunto em sala de aula.

## Algoritmos de classificação - Árvore de Decisão

E se nosso objetivo for prever não uma quantidade, como fizemos anteriormente com regressão linear, mas a probabilidade de uma observação pertencer a uma determinada classe? Por exemplo, podemos querer prever a probabilidade uma família estar ou não inserida no PBF ou ser escolhida para revisão de cadastro. Ou podemos querer saber a probabildiade de um pessoa ser boa ou má pagadora, abandonar ou não o ensino médio/universidade, em qual candidato votará na próxima eleição, e assim por diante.

Os problemas de classificação podem ser binários (apenas duas categorias) ou conter categorias múltiplas. Tal como em modelos de regressão linear, vamos tentar prever a variável alvo (que agora é categórica) a partir de um conjunto de outras variáveis.

Nosso exemplo de livro texto, bastante popular na literatura de aprendizado de máquina, será a probabilidade um indivíduo sobreviver ao naufrágio do Titanic com base em suas características: sexo, idade, classe em que viajava, etc.

Vamos começar abrindo os dados que estão disponíveis no pacote _titanic_. Vamos abrir os dados de treino, apenas, e utilizá-los para o exercício. Vamos selecionar apenas 4 variáveis: Survived, Pclass, Age, Sex e Fare.

```{r}
install.packages("titanic")
library(titanic)
data.frame(titanic_train)
titanic <- titanic_train %>% 
  select(Survived, Pclass, Age, Sex, Fare)
```

A primeira técnica que utilizaremos será árvore de decisão. Seu uso é bastante semelhante à aplicação de regressão linear para previsão, como vimos acima. Faremos, inclusive, a separação dos dados em "treino" e "teste". Antes de começar, vamos baixar o pacote _rpart_:

```{r}
install.packages("rpart")
library(rpart)
```

Tal como com a função _lm_, precisamos de uma fórmula que contém do lado esquerdo a variável que queremos explicar e do lado direito o conjunto de variáveis explicativas:

```{r}
tree <- rpart(Survived ~ ., data = titanic, method = "class")
```

A função _rpart_ tem outros usos além de classificador binário e, por esta razão, precisaos informar o método "class" como argumento na função.

_predict_, tem exatamente o mesmo uso que já examinamos, com a adição do argumento "type", como abaixo:

```{r}
prev <- predict(tree, type = "class")
```

Veja o resultado no vetor "prev": temos um vetor de categorias que indica se cada passageiro sobreviveu ou não.

```{r}
prev
```

Quão bom é o modelo? Uma simples tabela é capaz de indicar:

```{r}
conf <- table(titanic$Survived, prev)
conf
```

Dos 891 indivíduos classificados, 743 foram classificados corretamente e 148 incorretamente. A tabela acima é chamada de "Confusion Matrix" e a partir delas podemos ter uma boa noção da qualidade do modelo. Uma medida simples para sintetisar o modelo é denominada precisão, e pode ser calculada dividindo o total que foi corretamente classificado (soma dos elementos da diagonal) pelo total da tabela:

```{r}
sum(diag(conf)) / sum(conf)
```

## Exercício

Seguindo o modelo utilizado com os algoritmos de regressão, separe os dados entre "treino" e "teste" e produza um modelo de classificação.

## Atividade com dados do MDS

Vamos utilizar os dados do _data frame_ "pbf\_cadunico" para treinar o que acabamos de aprender. Nosso objetivo será prever se um indivíduo está ou não inscrito no PBF a partir da demais características. Faremos em conjunto em sala de aula.

## Algoritmos de agrupamento - Kmeans Cluster e Cluster Hierárquico

Até agora, vimos algoritmos de previsão e classificação, ou seja, de aprendizado de máquina supervisionado. Em todos os casos, uma quantidade ou um conjunto de classes era o alvo do modelo e queríamos obter a partir dos dados de treino que melhor se aproximasse dos valores observados para os dados de teste.

Quando tratamos de algoritmos de classificação, entretanto, não temos uma variável alvo. Nosso objetivo é encontrar estrutura nos dados. Em particular, com os algoritmos de agrupamento que veremos a seguir, nosso objetivo será encontrar grupos homogêneos em uma população heterogênea.

Análise de cluster tem aplicações diversas. Poderíamos por exemplo classificar municípios 

Neste exercício, vamos trabalhar com outro conjunto de dados de livro texto, "iris", que contém informações sobre tamanho e espessura de partes das flores de diferentes espécies de plantas. Vamos abrir os dados, examiná-los e criar uma cópia dos dados sem a informação sobre a espécie e com as variáveis de comprimento, apenas:

```{r}
data("iris")
glimpse(iris)
iris2 <- iris[,c(1,3)]
```

Omitindo a informação sobre a espécie podemos fazer a seguinte pergunta: é possível agrupar as plantas de acordo com as características de suas flores?

Antes de tentarmos responder à pergunta, vamos observar graficamente a distribuição conjunta das variáveis de comprimento:

```{r}
plot(iris2)
```

Apenas com base na visualização, como você diria que há grupos homogêneos no conjunto os dados?

Mesmo quando fazemos agrupamentos instintivos, utilizamos a distância entre os pontos como critério. As duas técnicas de agrupamento que veremos, kmeans e cluster hierárquico, utilizam a distância entre os pontos para agrupá-los.

Vamos começar com a função _kmeans_, cujo algoritmo é mais simples e seu funcionamento direto e claro.

A característica central do algorimto é que a quantidade de grupos é definida antes do agrupamento ser produzido. Kmeans é um algoritmo iterativo: produz-se aleatoriamente um agrupamento. A seguir calcula-se o centro de cada grupo e os pontos são reclassificados de acordo as distâncias aos  novos centros. Repete-se esse processo até não haver mais alteração nos centros e agrupamentos.

Na função _kmeans_, inserimos como argumento o _data frame_ com as variáveis que serão incluídas na análise (no nosso primeiro exemplo apenas 2), o número de grupos ("centers") e o número de conjuntos iniciais aleatórios pelos quais o algoritmo começará.

```{r}
km_iris <- kmeans(iris2, centers = 2, nstart = 20)
```

O principal resultado da aplicação do algoritmo é um vetor numérico com a informação de qual grupo cada observação pertence e que pode ser adicionado aos dados originais como variávei:

```{r}
km_iris$cluster
```

Vamos refazer o gráfico colorindo os pontos com os grupos aos quais cada ponto pertence:

```{r}
plot(iris2, col = km_iris$cluster, 
     main = "Iris: k-means com e clusters")
```

Bastante simples, não?

Vamos agora classificar os dados novamente, porém em 3 grupos, e refazer o gráfico?

```{r}
km_iris <- kmeans(iris2, centers = 3, nstart = 20)
plot(iris2, col = km_iris$cluster, 
     main = "Iris: k-means com e clusters")
```

A principal dificuldade ao construir uma análise de clusters é decidir qual é a quantidade de grupos adequada. Há possibilidades de explorar graficamente o tamanho ideal de cada cluster, mas o conhecimento do problema é essencial.

Espeficamente no caso dos dados com os quais estamos trabalhando, temos a informação de que as plantas são de espécies diferentes e podemos confrontar essa informação os grupos encontrados.

```{r}
table(iris$Species, km_iris$cluster)
```

Aparentemente, a análise de cluter a partir das características das flores gera uma classificação de grupos relativamente semelhante à classificação de espécies.

Vamos agora sair do mundo bidimensional e produzir o agrupamento com mais variáveis. Infelizmente, perderemos a possibilidade de explorar graficamente o problema. A mecânica é idêntica à que já vimos:

```{r}
iris3 <- iris[,1:4]
km_iris <- kmeans(iris3, centers = 3, nstart = 20)
table(iris$Species, km_iris$cluster)
```

O algoritmo Kmeans calcula iterativamente os grupos até não ter possibilidade de melhorar a classificação. O algoritmo de cluster hierárquico tem uma mecânica diferente. Na sua versão "bottom-up", procura-se os pontos mais próximos entre si para formar um grupo. A seguir, procura-se os próximos pontos com menor distância entre si (ou o ponto e o grupo com menor distância) e gera-se um novo grupo. Repete-se o processo até que se forme um grupo que contém todos os pontos.

Como os grupos vão se formando a partir de grupos já existência, produz-se um hierarquia entre os grupos, e daí o nome.

Diferentemente de todos algoritmos que vimos até agora, o "input" principal para a função _hclust_ não é um _data frame_, mas uma matriz de ditâncias. Use a função _dist_ abaixo e descubra o que é a matriz de distâncias:

```{r}
distancias_iris <- dist(iris3)
distancias_iris
```

A matriz acima é uma matriz quadrada de tamanho de linhas e colunas igual ao número de observações dos dados que a geraram. Basicamente, é uma matriz que relaciona cada ponto a cada ponto, teno como conteúdo das células as distâncias necessárias para que o algoritmo construa os grupos.

Vejamos agora como executar o algoritmo:

```{r}
hc_iris <- hclust(distancias_iris)
```

Usando o comando _plot_ podemos observar um dendograma, que é um gráfico que representa as distâncias e a hierarquia entre os grupos:

```{r}
plot(hc_iris)
```

E onde estão os grupos produzidos? Após a análise, você deve decidir qual é a quantidade adequada de grupos e o dendograma é uma ferramenta disponível para tal tarefa. Quantos grupos você imagina que há nos dados?

Se julgarmos que há 3 grupos, por exemplo, podemos gerar um vetor com os grupos para cada observação usando a função _cutree_:

```{r}
hc_clusters <- cutree(hc_iris, k = 3)
```

Para finaliza, vamos comparar o resultado de ambas análises com as espécies das plantas (que já conheciamos previamente) e entre si:

```{r}
#comparando as classificações que conhecemos com as previstas por cluster hierárquico
table(iris$Species, hc_clusters)

#comparando as classificações que conhecemos com as previstas por cluster kmeans
table(iris$Species, km_iris$cluster)

#comparando as classificações previstas pelo cluster hierárquico com as previstas por cluster kmeans
table(km_iris$cluster, hc_clusters)
```
