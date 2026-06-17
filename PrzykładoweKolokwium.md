# Przykładowe kolokwium


**\#Wczytaj Potrzebne Biblioteki**

``` r
library(readxl)
library(dplyr)
library(ppcor)
library(DescTools)
```

**\#Wczytaj plik**

``` r
dane <- read_excel("D:/Pulpit/R/Kolokwium2026/Dane-1.xlsx")
head(dane)
```

\#zad 1

**Pytanie 1. Średnia arytmetyczna stażu pracy męża V2 (0,1)**

``` r
Sr <- sum(dane$V2)/length(dane$V2)
Sr
```

odp: 276.5

\#zad 2

**Pytanie 2. Mediana stażu pracy męża V2 (0,1)**

``` r
me <- median(dane$V2)
me 
```

odp:280

\#zad 3

**Pytanie 3. Odchylenie standardowe (populacja) stażu pracy męża V2
(0,1)**

``` r
n <- length(dane$V2)
Sd <- sqrt(sum((dane$V2 - mean(dane$V2))^2)/n)
Sd
```

odp:40.4

\#zad 4

**Pytanie 4. Klasyczny współczynnik zmienności (populacja) stażu pracy
męża V2 (0,001); UWAGA: 0,123 - dobrze, 12,345% - źle**

``` r
kier_wsp_zmienn <- Sd/mean(dane$V2)
kier_wsp_zmienn
```

odp:0.146

\#zad 5

**Pytanie 5. Trzeci moment centralny standaryzowany (populacja) stażu
pracy męża V2 (0,001)**

``` r
m3 <- (sum((dane$V2 - mean(dane$V2))^3)/n)/(Sd)^3
m3
```

odp: -0.171 \< 0

\#zad 6

**Pytanie 6. Czwarty moment centralny standaryzowany (populacja) stażu
pracy męża V2 (0,001)**

``` r
m4 <- (sum((dane$V2 - mean(dane$V2))^4)/n)/(Sd)^4
m4
```

odp: 2.842 \< 3

\#zad 7

**Pytanie 7. Staż pracy męża V2 ma rozkład (wg miar klasycznych):**

`lewostronnie asym. ponieważ nasz trzeci moment jest mniejszy od 0`

**\#zad8 Pytanie 8. Staż pracy męża V2 ma rozkład (wg miar
klasycznych):**

`paltokurtyczny ponieważ nasz czwarty moment jest mniejszy od 3`

\#zad 9

**Pytanie 9. Współczynnik korelacji liniowej pomiędzy stażem pracy męża
V2 a wynagrodzeniem męża V4 (0,001)**

``` r
wsp_kor_lin <- cor(x =dane$V2 , y=dane$V4)
wsp_kor_lin
```

odp: 0.617

\#zad 10

**Pytanie 10. Cząstkowy współczynnik korelacji liniowej pomiędzy stażem
pracy męża V2 a wynagrodzeniem męża V4 przy uwzględnieniu wpływy stażu
pracy żony V1 i wynagrodzenia żony V3 (0,001**)

``` r
cza_wsp_kor_lin <- pcor.test(x =dane$V2 , y=dane$V4, dane[,c("V1","V3")])
cza_wsp_kor_lin$estimate
```

odp: 0.419

\#zad 11

**Pytanie 11. Współczynnik korelacji rang pomiędzy stażem pracy męża V2
a wynagrodzeniem męża V4 (0,001)**

``` r
rangi <- cor(x =dane$V2 , y=dane$V4, method = "spearman") #zawsze uzywac spearmana!!!!!!!
rangi
```

odp: 0.596

\#zad 12

**Pytanie 12. Współczynnik V-Cramera pomiędzy poziomem wykształcenia
żony V5 a poziomem wykształcenia męża V6 (0,001)**

``` r
Vcramer <- DescTools::CramerV(x=dane$V5,y=dane$V6)
Vcramer
```

odp: 0.056

\#zad 13

**Pytanie 13. Współczynnik kierunkowy w modelu regresji liniowej, w
którym zmienną objaśnianą jest wynagrodzenie żony V3 a zmienną
objaśniającą staż pracy żony V1 (0,01)**

``` r
model<- lm(V3~V1, data = dane)
wsp_kier <- model$coef[2]
wsp_kier
```

opdp:11.31

\#zad 14

**Pytanie 14. Wyraz wolny w modelu regresji liniowej, w którym zmienną
objaśnianą jest wynagrodzenie męża V4 a zmienną objaśniającą staż pracy
męża V2 (0,01)**

``` r
model2 <- lm(V4~V2, data = dane)
b0 <- model2$coefficients[1]
b0
```

odp;699.03

\#zad 15

**Pytanie 15. Odchylenie standardowe składnika resztowego modelu
regresji liniowej, w którym zmienną objaśnianą jest wynagrodzenie żony
V3 a zmienną objaśniającą staż pracy żony V1 (0,01)\***

``` r
model3 <- lm(V3~V1, data = dane)
model3_sum <- summary(model3)
Se <- model3_sum$sigma
Se
```

odp: 745.87

\#zad 16

**Pytanie 16. Współczynnik determinacji modelu regresji liniowej, w
którym zmienną objaśnianą jest wynagrodzenie żony V3 a zmienną
objaśniającą staż pracy żony V1 (0,001); UWAGA: 0,123 - dobrze,
12,345% - źle**

``` r
R2 <- model3_sum$r.squared
R2
```

odp: 0.394

\#zad 17

**Pytanie 17. Najsłabsza korelacja liniowa występuje pomiędzy:**

``` r
wyn17 <- cor(dane[,c("V1","V2","V3","V4")], use = "complete.obs")
wyn17
```

V3-V4 , znajdz w tabelce najmniejsza liczbę

\#zad 18

**Zaznacz przedział, w którym znajduje się procent objaśnionej
zmienności wynagrodzenia męża V4, przez model regresji liniowej, w
którym w roli zmiennej objaśniającej jest staż pracy mężą V2**

``` r
model4 <- lm(V4~V2, data = dane)
model4_sum <- summary(model4)
R24<- model4_sum$r.squared
proc_zmienn <- R2 * 100
```

odp : 39.158 \[25,50)
