Esta será el inicio de una serie de ejemplos para poder comprender como es el funcionamiento de un algoritmo de machine learning comencemos con una simple regresion lineal.

Regresion
=========

Una regresion es una tectnoa que con la ayuda de parametros (variables) ayuda en la prediccion de variables continuas

esta regrecion es representada por:

``` math
a + b = c
```

Cargaremos los paquete necesarios

``` r
library(readxl)
library(data.table)
library(ggplot2)
library(magrittr)
library(purrr)
```

    ## 
    ## Attaching package: 'purrr'

    ## The following object is masked from 'package:magrittr':
    ## 
    ##     set_names

    ## The following object is masked from 'package:data.table':
    ## 
    ##     transpose

``` r
library(magick)
```

    ## Linking to ImageMagick 6.9.9.14
    ## Enabled features: cairo, freetype, fftw, ghostscript, lcms, pango, rsvg, webp
    ## Disabled features: fontconfig, x11

Ahora cargamos los datos necesarios para este ejemplo, si gustas puedes descargarlos [aquí](pagina%20gith)

``` r
dat <- fread("_data/data.csv")

dat %>% head()
```

    ##           X1        X2
    ## 1:  6.500469  6.341401
    ## 2: 10.685361 13.755519
    ## 3: 12.306072 12.512476
    ## 4:  9.495128 14.309326
    ## 5: 11.962642 17.446185
    ## 6: 11.028438 15.642304

las dimenciones de estos datos son 100 filas y 2 columnas

Veamos como estarian graficadas

``` r
ggplot(dat,aes(X1,X2)) +
  geom_point()
```

![](regresion_lineal_files/figure-markdown_github/unnamed-chunk-3-1.png)

Basicamente lo que haremos es encontrar un linea que pase atraves de los puntos y que busque minimizar la distancia de esta recta con respecto a los puntos en otras palabras dicha recta buscara que la diferencia entre el valor predicho y el valor real sea el menor.

Add a new chunk by clicking the *Insert Chunk* button on the toolbar or by pressing *Ctrl+Alt+I*.

When you save the notebook, an HTML file containing the code and output will be saved alongside it (click the *Preview* button or press *Ctrl+Shift+K* to preview the HTML file).

The preview shows you a rendered HTML copy of the contents of the editor. Consequently, unlike *Knit*, *Preview* does not run any R code chunks. Instead, the output of the chunk when it was last run in the editor is displayed.
