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
## 1         javascript              1699310
## 2               java              1469802
## 3                 c#              1252377
## 4                php              1234894
## 5            android              1143501
## 6             python              1040750
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
## 1 javascript     jquery 515955
## 2 javascript       html 314482
## 3 javascript        css 145289
## 4 javascript  angularjs 116202
## 5 javascript        php 110600
## 6 javascript    node.js  90595
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
##  8 android android-studio     41247
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
visNetwork
```

<!--html_preserve--><div id="htmlwidget-daec3472b08028b6739e" style="width:672px;height:480px;" class="visNetwork html-widget"></div>
<script type="application/json" data-for="htmlwidget-daec3472b08028b6739e">{"x":{"nodes":{"id":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86],"label":["javascript","java","c%23","php","android","python","jquery","html","c%2b%2b","ios","css","mysql","sql","asp.net","ruby-on-rails","c","objective-c","arrays",".net","r","angularjs","node.js","json","sql-server","iphone","swift","ruby","regex","ajax","django","excel","xml","asp.net-mvc","linux","database","wpf","vba","spring","excel-vba","python-3.x","pandas","swing","xcode","wordpress","winforms","laravel","python-2.7","linq","eclipse","html5","css3","reactjs","android-layout","listview","numpy","codeigniter","hibernate","oracle","entity-framework","twitter-bootstrap","uitableview","android-studio","express","20c%2b%2b11","android-fragments","tsql","20qt","ruby-on-rails-3","20c","20c%2b%2b","list","maven","multithreading","xaml","jquery-ui","matplotlib","symfony","forms","mongodb","spring-mvc","string","android-intent","visual-studio","ruby-on-rails-4","sqlite","jsp"],"x":[-0.329534580784082,-0.293494870249229,0.206965455381412,-0.389334524284748,-0.358281859952452,0.864781214037364,-0.507846694784178,-0.488611039355129,-0.904442554318468,0.487555995019034,-0.52408822676946,-0.156391013765206,-0.0200867931639649,-0.0364694146693744,0.669784543932066,0.881701770580909,0.384776184719371,-0.308939110951688,0.347993472834352,0.988273104604225,-0.318180944089795,-0.27054965410408,-0.378319906024127,0.144382014503511,0.379545735142706,0.608916151299486,0.722135134361002,-0.284076353825128,-0.420716179099476,0.880687383762074,0.01281517911191,-0.381372742441418,0.0925492047254999,0.294866224951455,-0.0215010579683778,0.290738178839172,0.125213992820502,-0.0571167080563135,-0.0942838578891289,0.712990179946062,0.911788711664463,-0.171306175870604,0.452033739513342,-0.648488085898099,0.448510668093186,-0.64739099777747,0.99393106539305,0.413113660452267,-0.184300612580531,-0.456229726526672,-0.636514659195916,-0.187411489823947,-0.411149338673021,-0.373202588957319,0.787881908412027,-0.609597989458778,-0.289105812400654,0.176551207781743,0.474753210969756,-0.659889094113843,0.516511688309237,-0.558134942911829,-0.277613443112024,-1,-0.174639191852482,0.197967522841476,-0.839329477745851,0.756810764164126,-0.878554681158284,0.804296904992944,0.774528437114566,-0.0232227820632843,-0.532415521903129,0.351692941821497,-0.792167103549054,1,-0.711255090891217,-0.506847833081876,-0.134359332611118,-0.511448379207478,-0.0497209918742536,-0.520926277055188,0.222834068677831,0.537816919097295,-0.261380303104703,-0.154278396007422],"y":[-0.501799602423336,0.151961363868246,-0.609755993258957,-0.328568886693362,0.490917842315738,-0.25793960435485,-0.424518987283995,-0.530018122337947,0.476079250598292,0.703652879202139,-0.585664874203894,-0.317780092660439,-0.395457454100428,-0.607015370669395,0.254019337031572,0.487859982289608,0.711472701485296,-0.176799832182986,-0.606337504325424,0.222954888014106,-0.672599122357309,-0.791218298568517,-0.212360985353997,-0.467822199551143,0.634194841352075,0.698062789986597,0.345117064947637,-0.408895693983003,-0.434290325639136,-0.383129178102536,0.94941937181816,0.332070318335739,-0.694124914297466,0.977717785202659,-0.274834677529667,-0.741105824502755,0.92724957607178,0.316330041870249,1,-0.247487961153275,-0.0768927652334442,0.108933304101561,0.575097634783343,-0.344428628988551,-0.735807295130766,-0.198253944362788,-0.348809559308401,-0.474338213898852,0.268918364588591,-0.709759565885645,-0.709428800577232,-0.704560555021089,0.657410954331018,0.754752869762461,-0.438577921725507,-0.111389649367136,0.350536841412231,-0.21295673352412,-0.607460756463927,-0.563024252901238,0.852835658013127,0.549867891573958,-1,0.360677380264676,0.632093474672685,-0.345677137970584,0.616654080323153,0.157495996842955,0.345748591668372,0.573472574038117,-0.114683184025325,0.211680161757419,0.163966627225537,-0.823031421505477,-0.453386531220936,-0.174332958340216,-0.267007350420767,-0.128303020592319,-0.98111624810279,0.268659765421128,0.0972902700578946,0.676317569226369,-0.868117627510188,0.235786274207924,0.707734278408747,0.389207989072659]},"edges":{"from":[19,29,29,29,5,5,5,5,5,5,5,5,21,18,18,18,14,14,14,33,33,16,3,3,3,3,3,3,3,3,3,3,3,9,9,9,11,11,11,11,11,35,35,30,31,31,8,8,8,8,8,8,8,10,10,10,10,10,25,25,25,2,2,2,2,2,2,2,2,2,2,2,2,2,1,1,1,1,1,1,1,1,1,1,1,1,1,7,7,7,7,7,7,7,7,23,23,23,23,12,12,12,22,22,22,17,17,17,4,4,4,4,4,4,4,4,4,4,4,4,4,4,6,6,6,6,6,6,6,28,28,27,15,15,15,13,13,13,13,13,13,13,24,24,24,26,36,36,32,32],"to":[3,1,7,4,65,82,53,62,2,54,85,32,1,2,1,4,33,3,1,14,3,70,19,14,33,59,48,13,24,83,45,36,74,69,64,67,51,8,1,7,60,12,13,6,39,37,11,51,50,1,7,4,60,25,17,26,61,43,10,17,43,5,18,49,57,23,86,72,73,38,80,81,42,32,29,21,18,14,11,8,50,7,23,22,4,52,28,29,11,8,1,75,23,4,60,2,1,7,4,35,4,13,63,1,79,10,25,43,29,18,56,78,8,1,7,23,46,12,28,13,77,44,30,71,76,55,41,47,40,1,4,15,27,68,84,3,35,12,58,4,24,66,3,13,66,10,3,74,5,2],"weight":[169477,88028,107814,56893,36794,26439,49358,41247,218350,48417,25735,25783,116202,40310,48606,60513,33178,155177,31453,33178,67022,33987,169477,155177,67022,44121,53642,27811,29038,26281,65313,86111,30880,33987,39258,34683,50998,336794,145289,120538,42873,41389,33534,93287,79121,98624,336794,34900,28596,314482,203113,97670,34728,97743,162580,130709,42034,68507,97743,73387,27314,218350,40310,53021,45424,29439,25377,33049,32587,83884,26778,26586,69859,30989,88028,116202,48606,31453,145289,314482,51550,515955,57635,90595,110600,49460,28128,107814,120538,203113,515955,30346,34249,88904,26296,29439,57635,34249,34812,41389,215607,105780,39610,90595,27025,162580,73387,33473,56893,60513,45633,28173,97670,110600,88904,34812,61123,215607,27484,42937,28698,66901,93287,33305,28805,47371,73125,56005,78922,28128,27484,96614,96614,34434,25772,27811,33534,105780,44784,42937,105472,27835,29038,105472,35940,130709,86111,29868,25783,30989],"width":[3.45463922148975,1.79437316797737,2.19769333317026,1.15971364390576,0.750013249676912,0.538935704413977,1.00611931232138,0.840783728581388,4.45087223642316,0.986937856976872,0.524585285112663,0.525563722792298,2.36867531768648,0.821683809710179,0.990790455340435,1.2335041522449,0.676304277811072,3.16314632943181,0.641141673699188,0.676304277811072,1.36618437842708,0.692795029536587,3.45463922148975,3.16314632943181,1.36618437842708,0.899367684649536,1.09344487522881,0.566902714756879,0.591914027942549,0.535715013718512,1.33134792020841,1.75529681314694,0.629461573898544,0.692795029536587,0.800239717231511,0.706982375891296,1.03954926637558,6.86524874739593,2.96158816742699,2.45706085474685,0.873928304979025,0.843678273383641,0.683561023935032,1.90157324625238,1.61281182605009,2.0103632857568,6.86524874739593,0.711405729567979,0.582904247642577,6.41043829931224,4.14027942549401,1.99091683687405,0.707899661215953,1.99240487751183,3.31404995739719,2.66438772223785,0.856826029703737,1.39645479414079,1.99240487751183,1.49592929157035,0.556771807948991,4.45087223642316,0.821683809710179,1.08078633774853,0.925928190827962,0.600088059391167,0.517287770752052,0.673674726547053,0.664257263880565,1.70990138163554,0.545845920526399,0.541932169807859,1.42401412211718,0.631683442796049,1.79437316797737,2.36867531768648,0.990790455340435,0.641141673699188,2.96158816742699,6.41043829931224,1.05080129969138,10.5172877707521,1.17483865970345,1.84669919971951,2.25448348682574,1.00819849239061,0.573364480266135,2.19769333317026,2.45706085474685,4.14027942549401,10.5172877707521,0.618576454712604,0.698135668537929,1.8122296556307,0.536020775493398,0.600088059391167,1.17483865970345,0.698135668537929,0.709611927155315,0.843678273383641,4.39495859985568,2.15623203649573,0.807414926882168,1.84669919971951,0.550880797752855,3.31404995739719,1.49592929157035,0.682317592717162,1.15971364390576,1.2335041522449,0.93018847155804,0.574281765590793,1.99091683687405,2.25448348682574,1.8122296556307,0.709611927155315,1.2459384644236,4.39495859985568,0.560237108064365,0.875232888551871,0.584983427711801,1.36371790010967,1.90157324625238,0.67889306083844,0.587164528372654,0.965616069208158,1.49058865256901,1.14161254683251,1.60875538650327,0.573364480266135,0.560237108064365,1.96939120792208,1.96939120792208,0.701906730428189,0.525339497490715,0.566902714756879,0.683561023935032,2.15623203649573,0.912882355099495,0.875232888551871,2.1499537280514,0.567391933596696,0.591914027942549,2.1499537280514,0.732605212626738,2.66438772223785,1.75529681314694,0.608832846152905,0.525563722792298,0.631683442796049]},"nodesToDataframe":true,"edgesToDataframe":true,"options":{"width":"100%","height":"100%","nodes":{"shape":"dot","physics":false},"manipulation":{"enabled":false},"edges":{"smooth":false,"arrows":"middle"},"physics":{"stabilization":false}},"groups":null,"width":null,"height":null,"idselection":{"enabled":false},"byselection":{"enabled":false},"main":null,"submain":null,"footer":null,"background":"rgba(0, 0, 0, 0)","igraphlayout":{"type":"square"}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->