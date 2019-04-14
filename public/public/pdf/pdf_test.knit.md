---
title: "A Minimal PDF Example in blogdown"
author: "Frida Gomam"
date: "2017/07/28"
output:
  bookdown::pdf_document2:
    toc: false
---

I just want to show my favorite pie chart (Figure \@ref(fig:pie)):


```r
par(mar = c(0, 1, 0, 1))
pie(
  c(280, 60, 20),
  c('Sky', 'Sunny side of pyramid', 'Shady side of pyramid'),
  col = c('#0292D8', '#F7EA39', '#C4B632'),
  init.angle = -50, border = NA
)
```

\begin{figure}[h]

{\centering \includegraphics[width=0.8\linewidth]{pdf_test_files/figure-latex/pie-1} 

}

\caption{A fancy pie chart.}(\#fig:pie)
\end{figure}

More boring stuff:


```r
summary(cars)
```

```
##      speed           dist       
##  Min.   : 4.0   Min.   :  2.00  
##  1st Qu.:12.0   1st Qu.: 26.00  
##  Median :15.0   Median : 36.00  
##  Mean   :15.4   Mean   : 42.98  
##  3rd Qu.:19.0   3rd Qu.: 56.00  
##  Max.   :25.0   Max.   :120.00
```
