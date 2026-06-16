# Kolokwium
Xav

Zad. Wczytaj pakiety

``` r
library(readxl)
library(dplyr)
```

Zad. Wczytaj dane z pliku samochody.xlsx. Pozostaw tylko samochody
osobowe.

``` r
samochody <- 
  read_xlsx("../dane-surowe/samochody.xlsx") |>
  filter(Typ == "osobowy")

samochody
```

Zad. Zbadaj współzależność pomiędzy mocą a ceną samochodu.

``` r
plot(x = samochody$Moc, y = samochody$Cena)
```

``` r
cor(x = samochody$Moc, y = samochody$Cena)
```

Komentarz: Pomiędzy mocą a ceną obserwujemy silną dodatnią
współzależność liniową.

Zad. Zbuduj model regresji liniowej, w którym cena samochodów jest
objaśniana za pomocą mocy samochodów. Zinterpretuj parametry modelu.

``` r
# Y - cena
# X - moc 

model <- lm(Cena ~ Moc, data = samochody)
model
```

Interpretacja:

współczynnik kierunkowy: Wraz ze wzrostem mocy samochodu o 1 KM cena
samochodu wzrasta przeciętnie o 0.214 tys. dolarów

wyraz wolny: samochodów o zerowej mocy kosztuje teoretycznie -11.994
tys. dolarów (ta interpretacja nie ma sensu ekonomicznego/techniczego; w
praktyce tego parametru się nie interpretuje)

``` r
model$coefficients[2] # współczynnik kierunkowy
head(model$residuals) # kilka pierwszych reszt z modelu
```

Zad. Oceń dopasowanie modelu

``` r
model_sum <- summary(model)
model_sum
```

``` r
# odchylenie standardowe składnika resztowego

model_sum$sigma

# współczynnik zmienności składnika resztowego

model_sum$sigma/mean(samochody$Cena)
```

Interpretacja:

Teoretyczne wartości cen odchylają się od obserwowanych przeciętnie o
8.178 tys. dolarów.

Odchylenia teoretycznych wartości cen od obserwowanych stanowią
przeciętnie 29,5% przeciętnego poziomu cen.

``` r
# współczynnik determinacji R-kwadrat

model_sum$r.squared

# współczynnik zbieżności (indeterminacji)

1 - model_sum$r.squared
```

Interpretacja:

Model tłumaczy 72,4% zmienności cen samochodów. Pozostałe 27,6% nie
zostało wytłumaczonych.

W praktyce wykorzystuje się tylko R^2

``` r
# błędy oszacowania parametrów modelu

model_sum$coefficients[, 2]

# względne błędy oszacowań parametrów

model_sum$coefficients[, 2]/abs(model_sum$coefficients[, 1]) 
```

Interpretacja:

V_a1 \< 0.5, stąd współczynnik kierunkowy istotnie różni się od zera
(jest istotny statytycznie), moc istotnie wpływa na cenę samochodu

V_a0 \< 0.5, stąd wyraz wolny jest istotny statystycznie; w praktyce
ocenę istotności wyrazu wolnego raczej się pomija

``` r
# wartości p (p-value) testu istotności parametrów

model_sum$coefficients[,4]
```

Komentarz: W praktyce do oceny istotności parametrów wykorzystuje się
wartości p, parametr jako istotny uznaje się, gdy p-value jest mniejsze
niż 0.05 (poziom istotności)

Zad. Dokonaj prognozy/interpolacji ceny samochodu o mocy 500 KM

``` r
model$coefficients[1] + model$coefficients[2]*500
```

``` r
# błąd prnogzy

se <- model_sum$sigma
n <- nrow(samochody)
sr_x <- mean(samochody$Moc)
s2_x <- var(samochody$Moc)

se*sqrt(1+1/n+(500-sr_x)^2/(s2_x*(n-1)))
```

Interpretacja: Cena samochodu o mocy 500 KM teoretycznie wyniosłaby
95.005 (+/- 9.087) tys. dolarów
