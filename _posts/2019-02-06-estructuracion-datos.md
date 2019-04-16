---
title: "Estructuración de datos"
comments: true
date: 2019-02-06
tags: [machine learning, data science]
excerpt: "Machine Learning, Data Science"
mathjax: "true"
---
---
<img src="https://raw.githubusercontent.com/alonmar/estructuracion-datos/master/image/baldo2.jpg" alt="Autorretrato" width="300px"/>

Baldomero Romero Ressendi, Autorretrato
---

El código completo lo encontraras [aquí](https://github.com/alonmar/estructuracion-datos/blob/master/swapi.Rmd)

---

Normalmente cuando nos encontramos trabajando con datos en pocas ocasiones se encuentran organizados o estructurados, y la mayor parte del tiempo radica en la estructuración de estos, por lo que a continuación veremos como:

-   Solicitar datos de una APIweb, ya que es normal que muchos datos se entreguen de esta manera.
-   Dar forma a datos en formato json para poder analizarlos, ya que muchas de las herramientas estadísticas funcionan con dataframes.

### Hora de ensuciarse las manos!

Primero cargaremos las librerías necesarias

``` r
# En caso de no tener los paquetes necesarios debemos de instalarlos con los siguientes comandos
# install.packages(c("tidyverse","rjson","httr"))

library(tidyverse) #Estructuración de datos
```

``` r
library(rjson)     #Lectura de formato json
library(httr)      #Estatus de la conexión a la api
```

La api donde haremos la conexión es la de <https://swapi.co/> la cual tiene una base de datos de los personajes, planetas, razas etc. Del universo de Star Wars.

Primero veremos de que manera están estructurados los datos, nos apoyaremos en la [documentación](https://swapi.co/documentation).  Necesitaremos esta información principalmente para saber cuantos elementos contiene cada base

``` r
json_file <- "https://swapi.co/api/people/?page=1&format=json"
json_data <- fromJSON(file=json_file)

names(json_data)
```

    ## [1] "count"    "next"     "previous" "results"

Después de consultar la api observamos cuantos personajes existen

``` r
json_data$count
```

    ## [1] 87

Como vemos son 87 por lo tanto consultaremos uno a uno la url correspondiente a cada personaje, y crearemos nuestro propio dataset, de esta manera se tendrá un mejor control en su elaboración

``` r
# Creamos una lista vacía para ir llenándola
data <- list()
count <- json_data$count

for (i in 1:count) {

#Creamos la url
    url <- str_interp("https://swapi.co/api/people/${i}/?format=json")
#Vemos el estatus de la conexión
    resp <- GET(url)$status_code

  if(resp == 404) next #En caso de que dicha conexión nos regrese un estatus 404 pasaremos a la siguiente solicitud

# Guardamos en la lista
  data[[i]] <- fromJSON(file=url)



  if(i %% 5 == 0 | i == count ) cat(paste0(round(i/count*100),"% \n")) #Solo para ver el progreso

  }
```


Veamos la primer lista que contiene los datos de Luke Skywalker

``` r
data[[1]]
```

    ## $name
    ## [1] "Luke Skywalker"
    ##
    ## $height
    ## [1] "172"
    ##
    ## $mass
    ## [1] "77"
    ##
    ## $hair_color
    ## [1] "blond"
    ##
    ## $skin_color
    ## [1] "fair"
    ##
    ## $eye_color
    ## [1] "blue"
    ##
    ## $birth_year
    ## [1] "19BBY"
    ##
    ## $gender
    ## [1] "male"
    ##
    ## $homeworld
    ## [1] "https://swapi.co/api/planets/1/"
    ##
    ## $films
    ## [1] "https://swapi.co/api/films/2/" "https://swapi.co/api/films/6/"
    ## [3] "https://swapi.co/api/films/3/" "https://swapi.co/api/films/1/"
    ## [5] "https://swapi.co/api/films/7/"
    ##
    ## $species
    ## [1] "https://swapi.co/api/species/1/"
    ##
    ## $vehicles
    ## [1] "https://swapi.co/api/vehicles/14/" "https://swapi.co/api/vehicles/30/"
    ##
    ## $starships
    ## [1] "https://swapi.co/api/starships/12/"
    ## [2] "https://swapi.co/api/starships/22/"
    ##
    ## $created
    ## [1] "2014-12-09T13:50:51.644000Z"
    ##
    ## $edited
    ## [1] "2014-12-20T21:17:56.891000Z"
    ##
    ## $url
    ## [1] "https://swapi.co/api/people/1/"

Muy bien todo salió como esperábamos, ahora crearemos una función para hacer la obtención de datos mas rápida, y poder elegir que categoría solicitaremos.

``` r
 swapi <- function(resource) {

  url_resource <- str_interp("https://swapi.co/api/${resource}/?page=1&format=json")
  json_data <- fromJSON(file=url_resource)


   data <- list()
   count <- json_data$count

      for (i in 1:count) {

      #Creamos la url
          url <- str_interp("https://swapi.co/api/${resource}/${i}/?format=json")
      #Vemos el estatus de la conexión
          resp <- GET(url)$status_code

        if(resp == 404) next #En caso de que dicha conexión nos regrese un estatus 404 pasaremos a la siguiente solicitud

      # Guardamos en la lista
        data[[i]] <- fromJSON(file=url)

        if(i %% 5 == 0 | i == count ) cat(paste0(round(i/count*100),"% \n"))

      }

      data

       }
```

Las posibles categorías de búsqueda son:

-   films
-   people
-   planets
-   species
-   starships
-   vehicles

Comencemos con los personajes (people)

``` r
personajes <-  swapi("people")
```


Ahora con las razas(species)

``` r
especie <-  swapi("species")
```


Y por último los planetas(planets)

``` r
planeta <-  swapi("planets")
```


Ya tenemos los datos que nos interesan sin embargo al provenir de un formato json o por así decirlo "Nosql", tendremos problemas al convertir en una forma estructura por tablas, por lo tanto cambiaremos algunas cosas

``` r
# Pasamos de lista a dataframe


personajes_df <-  do.call(rbind, personajes) %>%  data.frame()

planeta_df <- do.call(rbind,planeta)  %>%  data.frame()

especie_df <- do.call(rbind,especie) %>%  data.frame()
```

Si observamos el caso de Luke Skywalker veremos que en la columna de *films* se encuentran las url a cada una de las películas en las que aparece, sin embargo es mejor solo saber el numero de la película y no la url, de manera que sea mas legible.

``` r
personajes_df$films[[1]]
```

    ## [1] "https://swapi.co/api/films/2/" "https://swapi.co/api/films/6/"
    ## [3] "https://swapi.co/api/films/3/" "https://swapi.co/api/films/1/"
    ## [5] "https://swapi.co/api/films/7/"

Por lo tanto haremos uso de las expresiones regulares y al paquete [stringr](https://resources.rstudio.com/rstudio-cheatsheets/stringr-cheat-sheet) incluido en tidyverse

Crearemos una función que eliminara todos los caracteres excepto los números

``` r
extract  <- function(variables) {
  lapply(variables, function(x)(str_replace_all(x,"\\D","")) %>% paste0(collapse = ","))
}
```

Reescribiremos ciertas variables aplicando la función antes hecha, crearemos la variable ID la cual es el número correspondiente a cada elemento

``` r
personajes_di <-  personajes_df %>% mutate(species = extract(species), films =  extract(films),
                                           homeworld = extract(homeworld),vehicles = extract(vehicles),
                                           starships = extract(vehicles),id_personajes = extract(url))


especie_di <-  especie_df %>% mutate(homeworld = extract(homeworld), people =  extract(people),
                                        films =  extract(films),id_especie = extract(url) )


planeta_di <-  planeta_df %>% mutate(residents = extract(residents),films =  extract(films),
                                     id_planeta = extract(url) )
```

Veamos ahora

``` r
personajes_di$films[[1]]
```

    ## [1] "2,6,3,1,7"

Muy bien ya está en un mejor formato, es importante mencionar que las películas están enumeradas de acuerdo al orden cronológico de estreno por lo tanto el número 1 hace referencia a "Episodio 4 - A New Hope" y no a "Episodio 1 - The Phantom Menace" (Obviamente esto no seria star wars sin el conflicto del orden de las películas)

Perfecto ahora solo queda convertir todo en un dataframe de vectores, ya que actualmente son dataframes de listas

``` r
personajes_ti <-  personajes_di  %>% map_dfr(unlist)

especie_ti <-  especie_di %>% map_dfr(unlist)

planeta_ti <-  planeta_di %>% map_dfr(unlist)


personajes_ti %>% head()
```

    ## # A tibble: 6 x 17
    ##   name  height mass  hair_color skin_color eye_color birth_year gender
    ##   <chr> <chr>  <chr> <chr>      <chr>      <chr>     <chr>      <chr>
    ## 1 Luke~ 172    77    blond      fair       blue      19BBY      male  
    ## 2 C-3PO 167    75    n/a        gold       yellow    112BBY     n/a   
    ## 3 R2-D2 96     32    n/a        white, bl~ red       33BBY      n/a   
    ## 4 Dart~ 202    136   none       white      yellow    41.9BBY    male  
    ## 5 Leia~ 150    49    brown      light      brown     19BBY      female
    ## 6 Owen~ 178    120   brown, gr~ light      blue      52BBY      male  
    ## # ... with 9 more variables: homeworld <chr>, films <chr>, species <chr>,
    ## #   vehicles <chr>, starships <chr>, created <chr>, edited <chr>,
    ## #   url <chr>, id_personajes <chr>

``` r
especie_ti %>% head()
```

    ## # A tibble: 6 x 16
    ##   name  classification designation average_height skin_colors hair_colors
    ##   <chr> <chr>          <chr>       <chr>          <chr>       <chr>      
    ## 1 Human mammal         sentient    180            caucasian,~ blonde, br~
    ## 2 Droid artificial     sentient    n/a            n/a         n/a        
    ## 3 Wook~ mammal         sentient    210            gray        black, bro~
    ## 4 Rodi~ sentient       reptilian   170            green, blue n/a        
    ## 5 Hutt  gastropod      sentient    300            green, bro~ n/a        
    ## 6 Yoda~ mammal         sentient    66             green, yel~ brown, whi~
    ## # ... with 10 more variables: eye_colors <chr>, average_lifespan <chr>,
    ## #   homeworld <chr>, language <chr>, people <chr>, films <chr>,
    ## #   created <chr>, edited <chr>, url <chr>, id_especie <chr>

``` r
planeta_ti %>% head()
```

    ## # A tibble: 6 x 15
    ##   name  rotation_period orbital_period diameter climate gravity terrain
    ##   <chr> <chr>           <chr>          <chr>    <chr>   <chr>   <chr>  
    ## 1 Tato~ 23              304            10465    arid    1 stan~ desert
    ## 2 Alde~ 24              364            12500    temper~ 1 stan~ grassl~
    ## 3 Yavi~ 24              4818           10200    temper~ 1 stan~ jungle~
    ## 4 Hoth  23              549            7200     frozen  1.1 st~ tundra~
    ## 5 Dago~ 23              341            8900     murky   N/A     swamp,~
    ## 6 Besp~ 12              5110           118000   temper~ 1.5 (s~ gas gi~
    ## # ... with 8 more variables: surface_water <chr>, population <chr>,
    ## #   residents <chr>, films <chr>, created <chr>, edited <chr>, url <chr>,
    ## #   id_planeta <chr>

Ahora es momento de darle un formato homogéneo a los NAs ya que tienen la forma de "n/a" "unknown" o simplemente es un caracter vacío

``` r
personajes_ti[personajes_ti == "n/a"|personajes_ti =="unknown"|personajes_ti ==""] <- NA
especie_ti[especie_ti == "n/a"|especie_ti =="unknown"|especie_ti ==""] <- NA
planeta_ti[planeta_ti == "n/a"|planeta_ti =="unknown"|planeta_ti ==""] <- NA
```

Y ahora el paso final! convertir las variables que hasta este momento eran del tipo caracter a integer

``` r
planeta_ti[c(2,3,4,8,9,15)] <- lapply(planeta_ti[c(2,3,4,8,9,15)], as.integer)
```

    ## Warning in lapply(planeta_ti[c(2, 3, 4, 8, 9, 15)], as.integer): NAs
    ## introduced by coercion to integer range

``` r
especie_ti[c(4,8,16)] <- lapply(especie_ti[c(4,8,16)], as.integer)
```

    ## Warning in lapply(especie_ti[c(4, 8, 16)], as.integer): NAs introducidos
    ## por coerción

``` r
personajes_ti[c(2,3,17)] <- lapply(personajes_ti[c(2,3,17)], as.integer)
```

    ## Warning in lapply(personajes_ti[c(2, 3, 17)], as.integer): NAs introducidos
    ## por coerción

Y este es el resultado

``` r
personajes_ti %>% head() %>% knitr::kable()
```

| name           |  height|  mass| hair\_color | skin\_color | eye\_color | birth\_year | gender | homeworld | films         | species | vehicles | starships | created                     | edited                      | url                              |  id\_personajes|
|:---------------|-------:|-----:|:------------|:------------|:-----------|:------------|:-------|:----------|:--------------|:--------|:---------|:----------|:----------------------------|:----------------------------|:---------------------------------|---------------:|
| Luke Skywalker |     172|    77| blond       | fair        | blue       | 19BBY       | male   | 1         | 2,6,3,1,7     | 1       | 14,30    | 1430      | 2014-12-09T13:50:51.644000Z | 2014-12-20T21:17:56.891000Z | <https://swapi.co/api/people/1/> |               1|
| C-3PO          |     167|    75| NA          | gold        | yellow     | 112BBY      | NA     | 1         | 2,5,4,6,3,1   | 2       | NA       | NA        | 2014-12-10T15:10:51.357000Z | 2014-12-20T21:17:50.309000Z | <https://swapi.co/api/people/2/> |               2|
| R2-D2          |      96|    32| NA          | white, blue | red        | 33BBY       | NA     | 8         | 2,5,4,6,3,1,7 | 2       | NA       | NA        | 2014-12-10T15:11:50.376000Z | 2014-12-20T21:17:50.311000Z | <https://swapi.co/api/people/3/> |               3|
| Darth Vader    |     202|   136| none        | white       | yellow     | 41.9BBY     | male   | 1         | 2,6,3,1       | 1       | NA       | NA        | 2014-12-10T15:18:20.704000Z | 2014-12-20T21:17:50.313000Z | <https://swapi.co/api/people/4/> |               4|
| Leia Organa    |     150|    49| brown       | light       | brown      | 19BBY       | female | 2         | 2,6,3,1,7     | 1       | 30       | 30        | 2014-12-10T15:20:09.791000Z | 2014-12-20T21:17:50.315000Z | <https://swapi.co/api/people/5/> |               5|
| Owen Lars      |     178|   120| brown, grey | light       | blue       | 52BBY       | male   | 1         | 5,6,1         | 1       | NA       | NA        | 2014-12-10T15:52:14.024000Z | 2014-12-20T21:17:50.317000Z | <https://swapi.co/api/people/6/> |               6|

``` r
especie_ti %>% head() %>% knitr::kable()
```

| name           | classification | designation |  average\_height| skin\_colors                      | hair\_colors              | eye\_colors                             |  average\_lifespan| homeworld | language       | people                                                                                             | films         | created                     | edited                      | url                               |  id\_especie|
|:---------------|:---------------|:------------|----------------:|:----------------------------------|:--------------------------|:----------------------------------------|------------------:|:----------|:---------------|:---------------------------------------------------------------------------------------------------|:--------------|:----------------------------|:----------------------------|:----------------------------------|------------:|
| Human          | mammal         | sentient    |              180| caucasian, black, asian, hispanic | blonde, brown, black, red | brown, blue, green, hazel, grey, amber  |                120| 9         | Galactic Basic | 1,4,5,6,7,9,10,11,12,14,18,19,21,22,25,26,28,29,32,34,43,51,60,61,62,66,67,68,69,74,81,84,85,86,35 | 2,7,5,4,6,3,1 | 2014-12-10T13:52:11.567000Z | 2015-04-17T06:59:55.850671Z | <https://swapi.co/api/species/1/> |            1|
| Droid          | artificial     | sentient    |               NA| NA                                | NA                        | NA                                      |                 NA| NA        | NA             | 2,3,8,23,87                                                                                        | 2,7,5,4,6,3,1 | 2014-12-10T15:16:16.259000Z | 2015-04-17T06:59:43.869528Z | <https://swapi.co/api/species/2/> |            2|
| Wookiee        | mammal         | sentient    |              210| gray                              | black, brown              | blue, green, yellow, brown, golden, red |                400| 14        | Shyriiwook     | 13,80                                                                                              | 2,7,6,3,1     | 2014-12-10T16:44:31.486000Z | 2015-01-30T21:23:03.074598Z | <https://swapi.co/api/species/3/> |            3|
| Rodian         | sentient       | reptilian   |              170| green, blue                       | NA                        | black                                   |                 NA| 23        | Galactic Basic | 15                                                                                                 | 1             | 2014-12-10T17:05:26.471000Z | 2016-07-19T13:27:03.156498Z | <https://swapi.co/api/species/4/> |            4|
| Hutt           | gastropod      | sentient    |              300| green, brown, tan                 | NA                        | yellow, red                             |               1000| 24        | Huttese        | 16                                                                                                 | 3,1           | 2014-12-10T17:12:50.410000Z | 2014-12-20T21:36:42.146000Z | <https://swapi.co/api/species/5/> |            5|
| Yoda's species | mammal         | sentient    |               66| green, yellow                     | brown, white              | brown, green, yellow                    |                900| 28        | Galactic basic | 20                                                                                                 | 2,5,4,6,3     | 2014-12-15T12:27:22.877000Z | 2014-12-20T21:36:42.148000Z | <https://swapi.co/api/species/6/> |            6|

``` r
planeta_ti %>% head() %>% knitr::kable()
```

| name     |  rotation\_period|  orbital\_period|  diameter| climate             | gravity                                | terrain                            |  surface\_water|  population| residents              | films     | created                     | edited                      | url                               |  id\_planeta|
|:---------|-----------------:|----------------:|---------:|:--------------------|:---------------------------------------|:-----------------------------------|---------------:|-----------:|:-----------------------|:----------|:----------------------------|:----------------------------|:----------------------------------|------------:|
| Tatooine |                23|              304|     10465| arid                | 1 standard                             | desert                             |               1|       2e+05| 1,2,4,6,7,8,9,11,43,62 | 5,4,6,3,1 | 2014-12-09T13:50:49.641000Z | 2014-12-21T20:48:04.175778Z | <https://swapi.co/api/planets/1/> |            1|
| Alderaan |                24|              364|     12500| temperate           | 1 standard                             | grasslands, mountains              |              40|       2e+09| 5,68,81                | 6,1       | 2014-12-10T11:35:48.479000Z | 2014-12-20T20:58:18.420000Z | <https://swapi.co/api/planets/2/> |            2|
| Yavin IV |                24|             4818|     10200| temperate, tropical | 1 standard                             | jungle, rainforests                |               8|       1e+03| NA                     | 1         | 2014-12-10T11:37:19.144000Z | 2014-12-20T20:58:18.421000Z | <https://swapi.co/api/planets/3/> |            3|
| Hoth     |                23|              549|      7200| frozen              | 1.1 standard                           | tundra, ice caves, mountain ranges |             100|          NA| NA                     | 2         | 2014-12-10T11:39:13.934000Z | 2014-12-20T20:58:18.423000Z | <https://swapi.co/api/planets/4/> |            4|
| Dagobah  |                23|              341|      8900| murky               | N/A                                    | swamp, jungles                     |               8|          NA| NA                     | 2,6,3     | 2014-12-10T11:42:22.590000Z | 2014-12-20T20:58:18.425000Z | <https://swapi.co/api/planets/5/> |            5|
| Bespin   |                12|             5110|    118000| temperate           | 1.5 (surface), 1 standard (Cloud City) | gas giant                          |               0|       6e+06| 26                     | 2         | 2014-12-10T11:43:55.240000Z | 2014-12-20T20:58:18.427000Z | <https://swapi.co/api/planets/6/> |            6|

Conclusiones
------------

Sin duda alguna la ingeniería de datos (data engineering) dentro de la ciencia de datos es lo menos "glamuroso" sin embargo es el paso mas importante ya que sin datos que analizar, de nada sirve el ultimo algoritmo de deeplearning. Los datos en la vida real no se encuentran estructurados sin embargo aquí se mostraron formas para lograr esto, iniciando por una consulta de una API, modificando documentos Json hasta pasando por expresiones regulares y manejos de NAs incluso intentar explicar el conflicto del número en las películas de star wars.

Te invito a que por tu cuenta hagas la solicitud a *films*, *starships*, *vehicles*. De tal manera que veas lo fácil que es!

### Fuentes:

<https://www.tutorialspoint.com/r/r_json_files.htm>
<https://github.com/Ironholds/rwars>
