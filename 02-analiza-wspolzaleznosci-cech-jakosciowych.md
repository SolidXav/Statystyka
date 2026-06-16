---
engines:
- path: "C:`\\Users\\xaver\\AppData\\Local\\Programs\\Quarto\\share\\extension`{=tex}``{=tex}``{=tex}``{=tex}``{=tex}``{=tex}``{=tex}``{=tex}-subtrees`\\julia`{=tex}-engine_extensions`\\julia`{=tex}-engine`\\julia`{=tex}-engine.js"
title: Kolokwium R
toc-title: Table of contents
---

Wczytaj potrzebne pakiety

::: cell
``` {.r .cell-code}
library(readxl)
library(DescTools)
```
:::

Usuń wszystkie obiekty

::: cell
``` {.r .cell-code}
remove(list = ls())
```
:::

Wczytaj dane z pliku bael.xlsx

::: cell
``` {.r .cell-code}
bael <- read_xlsx("../dane-surowe/bael.xlsx")
head(bael)
```
:::

Skonstruuj tabelę kontyngencji dla poziomu wykształcenia (WYKSZ) i
statusu na rynku pracy (KAT)

::: cell
``` {.r .cell-code}
x <- bael$WYKSZ
y <- bael$KAT
```
:::

::: cell
``` {.r .cell-code}
# rozkład WYKSZ
table(x)
prop.table(table(x))
```
:::

::: cell
``` {.r .cell-code}
# rozkład KAT
table(y)
prop.table(table(y))
```
:::

::: cell
``` {.r .cell-code}
# łączny rozkład WYKSZ i KAT
table(x, y) # tabela kontyngencji
prop.table(table(x, y))
prop.table(table(x, y), margin = 1)
prop.table(table(x, y), margin = 2)
```
:::

::: cell
``` {.r .cell-code}
mosaicplot(table(x,y), color = c("green", "red", "blue"))
```
:::

Napisz funkcję do wyznaczania statystyki chi-kwadrat w oparciu o wektory
x, y
$$\chi^2 = \sum_{i=1}^w\sum_{j=1}^k \frac{(n_{ij}-\hat{n}_{ij})^2}{\hat{n}_{ij}}$$

::: cell
``` {.r .cell-code}
chi2 <- function(x, y) {
  
  nij <- table(x,y)
  ni. <- table(x)
  n.j <- table(y)
  n <- sum(nij)
  nij_hat <- (ni. %*% t(n.j))/n
  chi.kwadrat <- sum((nij - nij_hat)^2/nij_hat)
  chi.kwadrat
  
}
```
:::

::: cell
``` {.r .cell-code}
chi2(x, y)
```
:::

Wyznacz wartość statystyki chi-kwadrat za pomocą funkcji chisq.test

::: cell
``` {.r .cell-code}
test <- chisq.test(x = x, y = y)
test$statistic
test$observed # tabela kontyngencji - liczebności empiryczne
test$expected # liczebności teoretyczne
```
:::

Sprawdź własności liczebności teoretycznych

::: cell
``` {.r .cell-code}
colSums(test$expected)
rowSums(test$expected)
#liczebności brzegowe odpowiadają obserwowanym 

prop.table(test$expected, margin = 1)
prop.table(test$expected, margin = 2)
# takie same struktury wszystkich wierszy i wszystkich kolumn
```
:::

Wyzna miary współzależności pomiędzy poziomem wykształcenia a statusem
na rynku pracy
$$T = \sqrt{\frac{\chi^2}{n\sqrt{(w-1)(k-1)}}}, \quad V =\sqrt{\frac{\chi^2}{n\cdot min(w-1, k-1)}}$$
$$C = \sqrt{\frac{\chi^2}{\chi^2+n}}, \quad C_{kor} = \frac{C}{C_{max}}, \quad C_{max} = \frac{\sqrt{\frac{w-1}{w}}+\sqrt{\frac{k-1}{k}}}{2}$$

::: cell
``` {.r .cell-code}
# ze wzoru
chi2_value <- chi2(x, y)
n <- length(x)
w <- length(unique(x))
k <- length(unique(y))

T_Czuprow <- sqrt(chi2_value/(n*sqrt((w-1)*(k-1))))
T_Czuprow

V_Cramer <- sqrt(chi2_value/(n*min(w-1,k-1)))
V_Cramer

C_Pearson <- sqrt(chi2_value/(chi2_value + n))
C_Pearson

C_max <- (sqrt((w-1)/w)+sqrt((k-1)/k))/2
C_max

C_kor <- C_Pearson/C_max
C_kor

# Pomiędzy badanymi cechami występuje umiarkowana współzależność
```
:::

::: cell
``` {.r .cell-code}
# za pomocą pakietu DescTools
TschuprowT(x = x , y = y)
CramerV(x = x, y = y)
ContCoef(x = x, y = y)
ContCoef(x = x, y = y, correct = TRUE)
```
:::
