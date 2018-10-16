---
title: "Scraping"
comments: true
date: 2018-09-02
tags: [machine learning, data science]
header:
  image: "/images/perceptron/percept.jpg"
excerpt: "Machine Learning, Data Science"
mathjax: "true"
---

#Scraping a stackoverflow

Hace poco me encontraba platicando con un amigo, quien me preguntaba que actualmente que lenguaje de pregramacion con mas demanda, mi primer respuesta fue que en la ciencia de datos los mas demandados son Python y R, sin embargo en no hablaba para algo especifico como la ciencia de datos.

En este [articulo](link) el autor analiza las preguntas mas populares de [stackoverflow](link) pagina que todos conocemos y si aun lo haz hecho no faltara mucho para que lo hagas

Lo que haremos es recolectar y organizar los temas mas recurrente en dicha pagina

Estos son los paquete necesitaremos de igual manera usaremos un plugin de [google chrome](https://chrome.google.com/webstore/detail/selectorgadget/mhjhnkcfbdhnjickkkdbjoemdmbfginb)(tambien disponible para firefox)



```r
library(tidyverse)
library('rvest')
library('dplyr')
library(tidygraph)
```

Vicitemos la url [https://stackoverflow.com/tags?page=1&tab=popular](https://stackoverflow.com/tags?page=1&tab=popular) como podemos ver en esta url tiene un tag  "page=1" el cual nos sirve para identificar la pagina en la que estamos y el tag "tab=popular" lo que nos indica que está ordenado del mas popular al menos popular para nuestro ejemplo solo tomaremos las cien primeros ranks, por lo que iremos desde la pagina uno hasta la pagina tres(cada pagina tiene 36 tags)

![imagen 1](images/captura1.png)


```r
url <- 'https://stackoverflow.com/tags?page=1&tab=popular'
```


```r
### nombre de las columnas que crearemos
names <- c("rank_lenguaje_name","rank_number_question")
```

usamos selector de css para cada variable

```r
### nombre de los nodos en el archivo html
node_name <- c('.post-tag','.item-multiplier-count')
```

Creamos la funcion info_extract la cual se encargara de estraer y convertir en un data frame los nodos que busquemos



```r
info_extract <- function(url,names,node_name) {

  #leemos la url
  webpage <- read_html(url)

  # ciclo for para cada nodo
  for (i in 1:length(node_name)) {
    #extraemos la infomacion del nodo
    html <- html_nodes(webpage,node_name[i])
    #lo convertimos en texto
    name <- html_text(html)

    if(i==1){
      result <- name

    }else {
      result <- cbind(result,name)
    }
  }

  #lo convertimos en un dataframe
  result <- data.frame(result,stringsAsFactors = F)
  names(result) <- names
  result
}
```





```r
extract <- info_extract(url,names,node_name)
head(extract)
```

```
##   rank_lenguaje_name rank_number_question
## 1         javascript              1699319
## 2               java              1469813
## 3                 c#              1252383
## 4                php              1234902
## 5            android              1143507
## 6             python              1040767
```

Creamos la funcion related_tags para obtener un data.frame con los lenguajes que mas se relacionan con los lenguajes populares


```r
related_tags <- function(lenguaje) {
  # creamos esta bandera para identificar que es el primer ciclo for
  flag_first <- T
  for (i in lenguaje) {
    cat("----------------------\n",
        "Extrayendo: ",i,"\n",
        "======================\n")
    #en caso de que el nombre sea "c#" la url de stackoverflow la combierte en el tag
    # "c%223" lo mismo sucede con "c++" lo vuelve "c%2b%2b" por lo tanto haremos lo mismo
    i <- ifelse(i=="c#","c%23",i)
    i <- ifelse(i=="c++","c%2b%2b",i)

    # creamos la url con el nombre del tag
    url <- paste0('https://stackoverflow.com/questions/tagged/',i)
    webpage <- read_html(url)
    # buscamos cuantas veces el tag secundario se asocia con el tag popular
    count_html <- html_nodes(webpage,'.item-multiplier-count')
    count <- html_text(count_html)

    # buscamos el nombre de los lenguajes que mas se taggean con los lenguajes populares
    rank_data_html <- html_nodes(webpage,'.js-gps-related-tags')
    character<- as.character(rank_data_html)
    strsplit <- strsplit(character," ")%>% unlist()

    #despues de separar buscamos en que filas se encuentran los tag que nos importan
    grep <- grep(paste0("/questions/tagged/",i),strsplit, value = T)
    grep <- gsub(paste0("href=\"/questions/tagged/",i,"+"),"",grep)
    grep <- gsub("\"","",grep)
    second_tag <- substr(grep,2,nchar(grep))
    popular <- rep(i,length(substr))
    cbind <-  cbind(popular,second_tag) %>% cbind(count)

    if(flag_first)  {
      related_tags <- cbind
      flag_first <- F
      }else {related_tags <- rbind(related_tags,cbind)}
  }

  related_tags <- related_tags %>% data.frame(stringsAsFactors = F)
  related_tags$count <-  related_tags$count %>% as.numeric()
  related_tags
}
```


```r
tabla_result <- related_tags(extract$rank_lenguaje_name)
```

```
## ----------------------
##  Extrayendo:  javascript
##  ======================
## ----------------------
##  Extrayendo:  java
##  ======================
## ----------------------
##  Extrayendo:  c#
##  ======================
## ----------------------
##  Extrayendo:  php
##  ======================
## ----------------------
##  Extrayendo:  android
##  ======================
## ----------------------
##  Extrayendo:  python
##  ======================
## ----------------------
##  Extrayendo:  jquery
##  ======================
## ----------------------
##  Extrayendo:  html
##  ======================
## ----------------------
##  Extrayendo:  c++
##  ======================
## ----------------------
##  Extrayendo:  ios
##  ======================
## ----------------------
##  Extrayendo:  css
##  ======================
## ----------------------
##  Extrayendo:  mysql
##  ======================
## ----------------------
##  Extrayendo:  sql
##  ======================
## ----------------------
##  Extrayendo:  asp.net
##  ======================
## ----------------------
##  Extrayendo:  ruby-on-rails
##  ======================
## ----------------------
##  Extrayendo:  c
##  ======================
## ----------------------
##  Extrayendo:  objective-c
##  ======================
## ----------------------
##  Extrayendo:  arrays
##  ======================
## ----------------------
##  Extrayendo:  .net
##  ======================
## ----------------------
##  Extrayendo:  r
##  ======================
## ----------------------
##  Extrayendo:  angularjs
##  ======================
## ----------------------
##  Extrayendo:  node.js
##  ======================
## ----------------------
##  Extrayendo:  json
##  ======================
## ----------------------
##  Extrayendo:  sql-server
##  ======================
## ----------------------
##  Extrayendo:  iphone
##  ======================
## ----------------------
##  Extrayendo:  swift
##  ======================
## ----------------------
##  Extrayendo:  ruby
##  ======================
## ----------------------
##  Extrayendo:  regex
##  ======================
## ----------------------
##  Extrayendo:  ajax
##  ======================
## ----------------------
##  Extrayendo:  django
##  ======================
## ----------------------
##  Extrayendo:  excel
##  ======================
## ----------------------
##  Extrayendo:  xml
##  ======================
## ----------------------
##  Extrayendo:  asp.net-mvc
##  ======================
## ----------------------
##  Extrayendo:  linux
##  ======================
## ----------------------
##  Extrayendo:  database
##  ======================
## ----------------------
##  Extrayendo:  wpf
##  ======================
```


```r
tabla_result %>% head()
```

```
##      popular second_tag  count
## 1 javascript     jquery 515956
## 2 javascript       html 314488
## 3 javascript        css 145291
## 4 javascript  angularjs 116203
## 5 javascript        php 110602
## 6 javascript    node.js  90594
```


![pop](images/pop_js.png)

![pop](images/pop_java.png)

![pop](images/pop_py.png)

![pop](images/pop_r.png)




```r
muestra <- tabla_result %>% arrange(desc(count)) %>% head(150)
```


```r
library(visNetwork)
library(tidygraph)

sources <- tabla_result %>%
  distinct(popular) %>%
  rename(label = popular)

destinations <- muestra %>%
  distinct(second_tag) %>%
  rename(label = second_tag)

nodes <- full_join(sources, destinations, by = "label")
#nodes
nodes <- nodes %>% rowid_to_column("id")
#nodes


per_route <- muestra %>%  
  group_by(popular, second_tag) %>%
  summarise(weight = sum(count)) %>%
  ungroup()
per_route
```

```
## # A tibble: 150 x 3
##    popular second_tag        weight
##    <chr>   <chr>              <dbl>
##  1 .net    c%23              169477
##  2 ajax    javascript         88028
##  3 ajax    jquery            107814
##  4 ajax    php                56893
##  5 android android-fragments  36794
##  6 android android-intent     26439
##  7 android android-layout     49358
##  8 android android-studio     41249
##  9 android java              218350
## 10 android listview           48417
## # ... with 140 more rows
```

```r
edges <- per_route %>%
  left_join(nodes, by = c("popular" = "label")) %>%
  rename(from = id)

edges <- edges %>%
  left_join(nodes, by = c("second_tag" = "label")) %>%
  rename(to = id)


edges <- select(edges, from, to, weight)
#edges




edges <- mutate(edges, width = weight/(max(weight)-min(weight))*10 )
visNetwork <- visNetwork(nodes, edges) %>%
  visIgraphLayout(layout = "layout_with_fr") %>%
  visEdges(arrows = "middle")

library(widgetframe)
```

```
## Loading required package: htmlwidgets
```

```r
frameWidget(visNetwork)
```

<!--html_preserve--><div id="htmlwidget-6b8b5a9a17b0d4b3bc9e" style="width:100%;height:480px;" class="widgetframe html-widget"></div>
<script type="application/json" data-for="htmlwidget-6b8b5a9a17b0d4b3bc9e">{"x":{"url":"scrap-stack_files/figure-html//widgets/widget_unnamed-chunk-11.html","options":{"xdomain":"*","allowfullscreen":false,"lazyload":false}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
