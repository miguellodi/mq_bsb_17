# Tutorial 4 - Abrindo dados no R

Neste tutorial vamos cobrir uma série de métodos disponíveis para abrirmos arquivos de texto. excel e de outros softwares de análise de dados no R. Vamos dar atenção aos argumentos das funções de forma a solucionar dificuldades de abertura de dados com diferentes características ou em sistemas operacionais variados.

## Abrindo dados com da funções do pacote utils

Quando você inicia uma nova sessão de R, alguns pacotes já estão automaticamente carregados. _utils_ é um deles, e ele contém as funções mais conhecidas de abertura de dados em arquivos de texto.

A principal função é _read.table_. Use a função _args_ para explorar seus argumentos:

```{r}
args(read.table)
```

É imprescindível que a função _read.table_ receba como primeiro argumento um arquivo de dados. Note que o caminho para o arquivo deve estar completo (ex: "C:\\\\User\\\\Documents\\\\file.txt") ou ele deve estar no seu __working directory__ (wd). Mas como eu descubro meu wd?

## Caminhos no R

```{r}
getwd()
```

E como eu altero meu wd?

```{r}
setwd("C:\\User\\Documents")
```

Simples e muito útil para evitar escrever "labirintos de pastas" ao importar dados.

Um detalhe fundamental para quem usa Windows: os caminhos devem ser escritos com duas barras no lugar de uma, como no exemplo acima. É uma chatice e a melhor solução é mudar definitivamente para linux.

Vamos supor que você queira abrir diversos arquivos ("file1.txt" e "file2.txt", por exemplo) que estão em uma pasta diferente do seu wd, por exemplo "C:\\\\User\\\\Downloads\\\\". Mudar o wd pode não ser conviente, mas escrever o caminho todo é menos ainda. Uma solução é criar usar _file.path_ para cadar arquivo armazenando a pasta e o caminho dos arquivos em algum objeto.

```{r}
pasta <- "C:\\User\\Downloads\\"
path_file1 <- file.path(pasta, "file1.txt")
path_file2 <- file.path(pasta, "file2.txt")
```

O código acima pode parecer pouco inteligente neste momento, mas tente pensar a combinação dele com loops para abrir diversos arquivos.

Use as funções que acabamos de ver para gerenciar caminhos de pastas no R. Vale a pena.

## read.table

Voltando à função _read.table_, vamos examinar os argumentos seguintes a _file_ usando um exemplo de dados retirado do Portal da Transparência. Extraí dos pagamentos do programa uma amostra de tamanho 50 e salvei em diversos arquivos com caracteristicas distintas.

Para facilitar nossa vida, vamos usar como argumento "file" o endereço dos dados no repositório do curso. O primeiro arquivo está no enderço que guardaremos em "file1".

```{r}
file1 <- "https://raw.githubusercontent.com/leobarone/FLS6397/master/data/bf_amostra_hv.csv"
```

Esse arquivo contém cabeçalho, ou seja, a primeira linha traz o nome das colunas. Por esta razão, informaremos "header = T". Ignore por hora o argumento "sep".

```{r}
dados <- read.table(file1, header = T, sep = ",")
head(dados)
```

O que aconteceria se escolhessemos "header = F"?

```{r}
dados <- read.table(file1, header = F, sep = ",")
head(dados)
```

Em primeiro lugar, o nome das variáveis é inserido automaticamente e na sequência V1, V2, ..., Vn, onde n é o número de colunas. Além disso, os nomes das variávei é lido como se fosse uma observação. Disso resulta que todas as variáveis serão lidas com um texto na primeira coluna, resultando, em "character" ou "factor" a depender das características dos dados.

```{r}
str(dados)
```

Vamos observar agora uma versão dos dados que não contém a primeira linha como nome das variáveis. Os dados estão no url armazenado em file2:

```{r}
file2 <- "https://raw.githubusercontent.com/leobarone/FLS6397/master/data/bf_amostra_nv.csv"
```

Como já havíamos visto, quando não há cabeçalho na primeira linha, os nomes são inseridos automaticamente:

```{r}
dados <- read.table(file2, header = F, sep = ",")
head(dados)
```

E se cometermos o erro de indicar que há cabeçalho quando não há?

```{r}
dados <- read.table(file2, header = T, sep = ",")
head(dados)
```

A primeira linha de dados se torna o nome das variáveis (inclusive os números antecedidos por um "X").

Ambos arquivos têm o mesmo separador: vírgula. O argumento "sep" permite indicar qual é o separador.

Não há muita graça em observar os exemplos com separadores diferente, mas vejamos como abrí-los. Os mais comuns, além da vírgula, são o ponto e vírgula e o tab, este último representado pelo símbolo "\t"

```{r}
# Ponto e virgula
file3 <- "https://raw.githubusercontent.com/leobarone/FLS6397/master/data/bf_amostra_hp.csv"
dados <- read.table(file3, header = T, sep = ";")

file4 <- "https://raw.githubusercontent.com/leobarone/FLS6397/master/data/bf_amostra_ht.csv"
dados <- read.table(file4, header = T, sep = "\t")
```

Há outras funções da mesma família de _read.table_ no pacote _utils_. A diferença entre elas é o separador de colunas -- vírgula para _read.csv_, ponto e vírgula para _read.csv2_, tab para _read.delim_ e _read.delim2_ -- e o separador de decimal.

Por default, _read.table_ considera que os campos em cada coluna estão envolvidas por aspas duplas (quote = "\\"". Para indicar que não há nada, use quote = "".

"dec" é o argumento para o separador decimais. Como o padrão brasileiro é a vírgula, e não o ponto, este argumento costuma ser importante.

Por vezes é conveniente importar apenas um subconjunto das linhas. "skip" permite que pulemos algumas linhas e "nrows" indica o máximo de linhas a serem abertas. Se o banco de dados for desconhecido e muito grande, abrir uma fração permite conhecer (tentando e errando) os demais argumentos ("header", "sep", etc) adequados para abrir os dados com um baixo custo de tempo e paciência.

Por exemplo, para pular as 3 primeiras linhas:

```{r}
dados <- read.table(file2, header = T, sep = ",", skip = 3)
head(dados)
```

Para abrir apenas 20 linhas:

```{r}
dados <- read.table(file2, header = T, sep = ",", nrows = 20)
head(dados)
```

Combinando, para abrir da linha 11 à linha 30:

```{r}
dados <- read.table(file1, header = T, sep = ",", skip = 10, nrows = 30)
head(dados)
```

Por vezes, é interessante definir as classes das variáveis a serem importadas. O argumento deve ser um vetor com uma classe para cada coluna. Por exemplo:

```{r}
dados <- read.table(file1, header = T, sep = ",", 
  colClasses = c("character", "numeric","character", "numeric", "numeric"))
str(dados)
```

Perceba que quando abrimos os dados sem especificar o tipo da coluna, a função _read.table_ tenta identificá-los. Uma das grandes chatices das funções de abertura de dados pacote _utils_ é que colunas de texto são normalmente identificadas como "factors", mesmo quando claramente não são. Veja os exemplos anteriores

Para evitar que textos sejam lidos como "factors" é importante informar o parâmetro "stringsAsFactors = F", pois o padrão é "T". Este argumento incomoda tanto que diversas pessoas chegam a alterar a configuração básica da função para não ter de informá-lo diversas vezes.


```{r}
dados <- read.table(file1, header = T, sep = ",", stringsAsFactors = F)
str(dados)
```

Note que, agora, "uf" e "munic" são importadas como "character".

Finalmente, é comum termos problemas para abrir arquivos que contenham caracteres especiais, pois há diferentes formas do computador transformar 0 e 1 em vogais acentuadas, cecedilha, etc. O "encoding" de cada arquivo varia de acordo com o sistema operacional e aplicativo no qual foi gerado.

Para resolver este problema, informamos ao R o parâmetro "fileEncoding", que indica qual é o "encoding" esperado do arquivo. Infelizmente não há formas automáticas de descobrir o "encoding" de um arquivo e é preciso conhecer como foi gerado -- seja por que você produziu o arquivo ou por que teve acesso à documentação -- ou partir para tentativa e erro. Alguns "encodings" comuns são "latin1", "latin2" e "utf8", mas há diversos outros. Como arquivo com o qual estamos trabalhando não contém caracteres especiais, não é preciso fazer nada.

## Tibbles e tidyverse

O desenvolvimento de pacotes em R levou à criação de um tipo específico de _data frame_, chamado _tibble_. A estrura é idêntica à de um _data frame_ regular e suas diferenças se resumem à forma que os dados aparecem no console ao "imprimirmos" o objeto, ao "subsetting", ou seja, à seleção de linhas e à adoção de "stringsAsFactors = F" como padrão. Você pode ler mais sobre _tibbles_ [aqui](https://cran.r-project.org/web/packages/tibble/vignettes/tibble.html).

O pacote _readr_, parte do _tidyverse_ (conjunto de pacotes com o qual vamos trabalhar), contém funções para abertura de dados em .txt semelhantes às do pacote _utils_, mas que trazem algums vantagens: velocidade de abertura, simplicação de argumentos e a produção de _tibbles_ como resultado da importação.

```{r}
library(readr)
```

A função análoga à _read.table_ em _readr_ chama-se _read\_delim_. Veja:

```{r}
dados <- read_delim(file1, delim = ",")
dados
```

Observe que não utilizamos _head_ para imprimir as primeiras linhas. Essa é uma característica de _tibbles_: o output contém uma fração do banco, a informação sobre número de linhas e colunas, e os tipos de cada variável abaixo dos nomes das colunas. "delim" é o argumento que entra no lugar de "sep" ao utilizarmos as funções do _readr_.

O padrão de _read\_delim_ é importar a primeira coluna como nome das variáveis. No lugar de _header_, temos agora o argumento "col_names", que deve ser igual a "FALSE" para os dados armezenados em "file2", por exemplo:

```{r}
dados <- read_delim(file2, col_names = F, delim = ",")
dados
```

X1, X2, ..., Xn, com "n" igual ao número de colunas, passam a ser os nomes das variáveis neste caso.

Além dos valores lógicos, "col_names" também aceita um vetor com novos nomes para as colunas como argumento:

```{r}
dados <- read_delim(file2, col_names = c("estado", "municipio_cod", "municipio_nome",
                                         "NIS", "transferido"),
                    delim = ",")
dados
```

"skip" e "n_max" são os argumentos de _read\_delim_ correspondentes a "skip" e "nrows".

Finalmente, "col_types" cumpre a função de "colClasses", com uma vantagem: não é preciso usar um vetor com os tipos de dados para cada variável. Basta escrever uma sequência de caracteres onde "c" = "character", "d" = "double", "l" = "logical" e "i" = "integer":

```{r}
dados <- read_delim(file1, delim = ",", col_types = "cicid")
dados
```

Mais econômico, não?

_read\_csv_ e _read\_tsv_ são as versões de _read\_delim_ para arquivos separados por vírgula e arquivos separado por tabulações.

Do ponto de vista forma, as três funções de importação de dados de texto do pacote _readr_ geram objetos que pertecem à classe de _data frames_ e também às classes _tbl_ e _tbl\_df_, que são as classes de _tibbles_.

Ainda no pacote _readr_, duas funções são bastante úteis


## Outra gramática para dados em R: data.table

No curso vamos trabalhar com duas "gramáticas" para dados em R: a dos pacotes básicos e a fornecida pelos pacotes do _tidyverse_. O pacote _data.table_ oferece uma terceira alternativa, que não trabalharemos no curso. Esta "gramática" guarda semelhanças (poucas, ao meu ver) com a linguagem SQL.

```{r}
library(data.table)
```


Há, porém, duas funções excepcionalmente boas neste pacote: _fread_ e _fwrite_, que servem, respectivamente, para importar e exportar dados de texto.

```{r}
class(dados)
```


As duas grandes vantagens de utilizar _fread_ são: a função detecta os automaticamente as características do arquivo de texto para definir delimitador, cabeçalho, tipos de dados das colunas, etc; e é extremamente rápida em comparação às demais. "f" vem de "Fast and friendly file finagler".

```{r}
dados <- fread(file1)
head(dados)
```

Além de ser um _data.frame_, os objeto criado com _fread_ também é da classe _data.table_ e aceita a "gramática" do pacote de mesmo nome.

Obviamente, se você precisar especificar os argumentos para _fread_ ler um arquivo, não há problemas. Eles são muito parecidos aos de _read.table_.

## Dados em arquivos editores de planilhas

Editores de planilha são, em geral, a primeira ferramenta de análise de dados que aprendemos. Diversas organizações disponibilizam (infelizmente) seus dados em formato .xls ou .xlsx e muitos pesquisadores utilizam editores de planilha para construir bases de dados.

Vamos ver como obter dados em formato .xls ou .xlsx diretamente, sem precisar abrir os arquivos e exportá-los para um formato de texto.

Há dois bons pacotes com funções para dados em editores de planilha: _readxl_ e _gdata_. Vamos trabalhar apenas com o primeiro, mas convém conhecer o segundo se você for trabalhar constantemente com planilhas e quiser editá-las, e não só salvá-las. _readxl_ também é parte do _tidyverse_. Importe o pacote:

```{r}
library(readxl)
```


## Um pouco sobre donwload e manipulação de arquivos

Nosso exemplo será a Pesquisa Perfil dos Municípios Brasileiros de 2005, produzida pelo IBGE e apelidade de MUNIC. Diferentemente das demais funções deste tutorial, precisamos baixar o arquivo para o computador e acessá-lo localmente. Faça o download diretamente do [site do IBGE](ftp://ftp.ibge.gov.br/Perfil_Municipios/2005/base_MUNIC_2005.zip) e descompacte. Ou, mais interessante ainda, vamos automatizar o download e descompactação do arquivo (aviso: pode dar erro no Windowns e tentaremos corrigir na hora).

Em primeiro lugar, vamos guardar o endereço url do arquivo em um objeto e fazer o download. Note que na função _download.file_ o primeiro argumento é o url e o segundo é o nome do arquivo que será salvo (nota: você pode colocá-lo em qualquer pasta utilizando _file.path_ para construir o caminho completo para o arquivo a ser gerado).

```{r}
url_arquivo <- "ftp://ftp.ibge.gov.br/Perfil_Municipios/2005/base_MUNIC_2005.zip"
download.file(url_arquivo, "temp.zip", quiet = F)
```

O argumento "quiet = F" serve para não imprimirmos no console "os números" do download (pois o tutorial ficaria poluído), mas você pode retirá-lo ou alterá-lo caso queira ver o que acontece.

Com _unzip_, vamos extrair o conteúdo da pasta:

```{r}
unzip("temp.zip")
```

Use _list.files_ para ver todos os arquivos que estão na sua pasta caso você não saiba o nome do arquivo baixado. No nosso caso utilizaremos o arquivo "Base 2005.xls"

```{r, eval = F}
list.files()
```

Vamos aproveitar e excluir nosso arquivo .zip temporário: 

```{r}
file.remove("temp.zip")
```

## Voltando às planilhas

Para não repetir o nome do arquivo diversas vezes, vamos criar o objeto "arq" que contém o endereço do arquivo no seu computador (ou só o nome do arquivo entre aspas se você tivê-lo no seu wd):

```{r}
arquivo <- "Base 2005.xls"
```

Com _excel\_sheets_ examinamos quais são as planilhas existentes do no arquivo:

```{r, echo = F, results = 'hide'}
excel_sheets(arquivo)
```

No caso, temos 11 planilhas diferentes (e muitas de mensagens de erro estranhas). O dicionário, para quem já trabalhou alguma vez com a MUNIC, não é uma base de dados, apenas textos espalhados entre células. As demias, no entanto, têm formato adequado para _data frame_.

Vamos importar os dados da planilha "Variáveis externas". Todas duas maneiras abaixo se equivalem:

```{r, echo = F, results = 'hide'}
# 1
transporte <- read_excel(arquivo, "Variáveis externas")

# 2
transporte <- read_excel(arquivo, 11)
```

A função _read\_excel_ aceita os argumentos "col_names", "col_types" e "skip" tal como as funções de importação do pacote _readr_.

```{r, include = F}
file.remove("Base 2005.xls")
```

## Dados de SPSS, Stata e SAS

R é bastante flexível quanto à importação de dados de outros softwares estatísticos. Para este fim também há dois pacotes, _foreign_, mais antigo e conhecido, e _haven_, que é, advinhe só, parte do _tidyverse_. São parecidos entre si e recomendo o uso do segundo, que examinaremos abaixo.

```{r}
library(haven)
```

Basicamente, há cinco funções de importação de dados em _haven_: _read\_sas_, para dados em SAS; _read\_stata_ e _read\_dta_, idênticas, para dados em formato .dta gerados em Stata; e _read\_sav_ e _read\_por_, uma para cada formato de dados em SPSS. O uso, como era de se esperar, é bastante similar ao que vimos no tutorial todo.

Vamos usar como exemplo o [Latinobarômetro 2015](http://www.latinobarometro.org/latContents.jsp), que está disponível para SAS, Stata, SPSS e R. Como os arquivos são grandes demais e o portal do Latinobarômetro é "cheio de javascript" (dá mais trabalho pegar dados de um portal com funcionalidades construídas nesta linguagem), vamos fazer o processo manual de baixar os dados, descompactá-los e abrí-los. Vamos ignorar SAS por razões que não interessam agora e por não ser uma linguagem popular nas ciências sociais, mas se você tiver interesse em saber mais, me procure.

Deixo abaixo uma breve descrição do Latinobarômetro que "roubei" de outro curso que ministrei. É possível que façamos exercícios com o Latinobarômetro no futuro, dado que é um banco de dados com muitas (muitas mesmo) variáveis categóricas, posto que é survey.

#### 2011 Latinobarometer

The second dataser is the 2015 Latinobarometer. This is a very popular survey on democracy and elections in Latin America and the original can be found [here](http://www.latinobarometro.org/latContents.jsp). Latinobarometer contains data on several Latin American Countries. We are going to use the data on Brazil only. "Barometers" are very good source of public opinion data regarding issues of political regime, civil liberties, economic performance of governments and etc. I am sure this dataset can be source of lots of dissertations of students in political economy and comparative politics.

We will use the dataset to formulate hypothesis about the brazilian electorate and explore our creativity. This is why it will be very important that we spend some time exploring the dictionary, whose .pdf is available both in [english](https://github.com/leobarone/IPSA_USP_EADA_2017/blob/master/Data/Latinobarometro_2011_eng.pdf) and [spanish](https://github.com/leobarone/IPSA_USP_EADA_2017/blob/master/Data/Latinobarometro_2011_esp.pdf) (although not in portuguese, even though it exists). It is a consolidated survey, so don't be frustrated if the data you are looking for is not ther and look for something else.

## Abrindo os dados com haven

Vejamos o uso das funções em arquivos de diferentes formatos:

```{r}
# SPSS
lat <- read_spss("/home/acsa/FLS/Latinobarometro_2015_Eng.sav")
head(lat)

# Stata
lat <- read_stata("/home/acsa/FLS/Latinobarometro_2015_Eng.dta")
head(lat)
```

Simples assim.

Há critérios de conversão de variáveis categóricas, rótulos e etc, adotados pelo R ao importar arquivos de outras linguagens, mas você pode descobrí-los testando sozinh@.

## Arquivos .RData

Faça download do arquivo do Latinobarômetro 2015 para formato R. Você verá que o arquivo tem a extensão .RData. Este é o formato de dados do R? Sim e não.

Começando pelo "não": um arquivo .RData não é um arquivo de base de dados em R, ou seja, não contém um _data frame_. Ele contém um workspace inteiro! Ou seja, se você salvar o seu workspace agora usando o "botão de disquete" do RStudio que está na aba "Enviroment" (provavelmente no canto superior à direita), você salvará todos os objetos que ali estão -- _data frames_, vetores, funções, gráficos, etc -- e não apenas um único _data frame_.

Não há um arquivo de dados do R que se pareça com os arquivos de SPSS e Stata. Em um tutorial futuro veremos como exportar arquivos de texto com as famílias de funções "write", primas das funções "read", dos mesmos pacotes que usamos neste tutorial.

Para salvar um arquivo .RData sem usar o "botão do disquete", use a função _save.image_:

```{r}
save.image("myWorkspace.RData")
```

Para abrir um arquivo .RData, por exemplo, o do Latinobarômetro ou o que você acabou de salvar, use a função _load_:

```{r}
# Latinobarometro
load("/home/acsa/FLS/Latinobarometro_2015_Eng.rdata")

# Workspace do tutorial
load("myWorkspace.RData")
```

```{r, include = F}
file.remove("myWorkspace.RData")
```
 
