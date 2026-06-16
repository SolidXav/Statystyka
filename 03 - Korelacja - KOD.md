# Kolokwium
Xav

Wczytaj pakiety

``` r
library(quantmod) # getSymbols
library(dplyr)
library(readxl)
library(ppcor, include.only = "pcor")
```

Usuń wszystkie obiekty

``` r
remove(list = ls())
```

Pobierz ceny złota dla roku 2025

``` r
getSymbols("GC=F", src = "yahoo", from = "2025-11-01", to = "2025-12-31", auto.assign = FALSE)
```

Pobierz ceny zamknięcia najważniejszych surowców od
01.11.2025-31.12.2025. Jako źródło wykorzystajcie “yahoo”. Wynikiem ma
być ramka danych, gdzie w kolumnach będą poszczególne surowce.

``` r
symbole <- c("GC=F", "SI=F", "CL=F", "BZ=F", "NG=F", "HG=F", "PL=F", "PA=F", "ZW=F", "ZC=F", "ZS=F", "CC=F", "KC=F", "SB=F")

nazwy <- c("Zloto", "Srebro", "Ropa_WTI", "Ropa_Brent", "Gaz_ziemny", "Miedz", "Platyna", "Pallad", "Pszenica", "Kukurydza", "Soja", "Kakao", "Kawa", "Cukier")

close_list <- 
  lapply(symbole, \(x) {
    dane_x <- getSymbols(x, src = "yahoo", from = "2025-11-01", to = "2025-12-31", auto.assign = FALSE)
    dane_x <- as.data.frame(dane_x)
    dane_x$Date <- rownames(dane_x)
    rownames(dane_x) <- NULL
    dane_x <- dane_x[, c("Date", paste0(x, ".Close"))]
    dane_x
})

close_df <- Reduce(merge , close_list)
colnames(close_df)[-1] <- nazwy

close_df
```

Zbadaj współzależność pomiędzy cenami złota i srebra

``` r
plot(x = close_df$Zloto, y = close_df$Srebro)
```

``` r
# współczynnik korelacji liniowej Pearsona
cor(x = close_df$Zloto, y = close_df$Srebro)
```

Komentarz: Pomiędzy cenami złota a cenami srebra obserwujemy bardzo
silną dodatnią korelację liniową

``` r
# współczynnik korelacji rang Spearmana
cor(x = close_df$Zloto, y = close_df$Srebro, method = "spearman")
cor(x = rank(close_df$Zloto), y = rank(close_df$Srebro))
```

``` r
# funkcja rank
close_df$Zloto
rank(close_df$Zloto)
```

Eksperyment 1: Dodaj do zestawu cen złota i srebra wartości odpowiednio
8000 i 20.

``` r
zloto_outlier <- c(close_df$Zloto, 8000)
srebro_outlier <- c(close_df$Srebro, 20)

plot(zloto_outlier, srebro_outlier)

cor(zloto_outlier, srebro_outlier)
cor(zloto_outlier, srebro_outlier, method = "spearman")
```

Eksperyment 2: Wygeneruj dwa wektory z rozkładu normalnego N(1000, 10).
Wyznacz współczynnik między tymi wektorami. Następnie dodaj wartość
odstającą i ponownie wyznacz współczynniki korelacji.

``` r
set.seed(1234)

x <- rnorm(50, 1000, 10)
y <- rnorm(50, 1000, 10)

plot(x, y)
cor(x, y)
cor(x, y, method = "spearman")

x <- c(x, 100)
y <- c(y, 100)

plot(x, y)
cor(x, y)
cor(x, y, method = "spearman")
```

Zadanie: - A Pomiędz którą parą surowców występuje nasilniejsza
współzależność liniowa - B Pomiędz którą parą surowców występuje
nasłabsza współzależność liniowa - C Pomiędz którą parą surowców
występuje nasilniejsza dodatnia współzależność liniowa - D Pomiędz którą
parą surowców występuje nasilniejsza ujemna współzależność liniowa - E Z
jakim surowcem jest najsilniej skorelowane złoto - F Z jakim surowcem
jest najsłabiej skorelowane złoto

``` r
macierz_korelacji <- cor(close_df[,-1])
diag(macierz_korelacji) <- NA
macierz_korelacji
```

``` r
arrayInd(which.max(abs(macierz_korelacji)), .dim = dim(macierz_korelacji))
```

Odpowiedź: Najsilniejsza współzależność występuje między Ropa_WTI a
Ropa_Brent

B

``` r
arrayInd(which.min(abs(macierz_korelacji)), .dim = dim(macierz_korelacji))
```

Odpowiedź: Najsłabsza współzależność występuje między Pszenicą a Kakao

D

``` r
macierz_korelacji_ujemne <- macierz_korelacji
macierz_korelacji_ujemne[macierz_korelacji_ujemne >= 0] <- NA
macierz_korelacji_ujemne

arrayInd(which.max(abs(macierz_korelacji_ujemne)), .dim = dim(macierz_korelacji))
```

Odpowiedź: Najsilniejsza ujemna współzależność występuje między Kawą a
Palladem

Zadanie: Wczytaj dane z pliku samochody.xlsx. Pozostaw tylko samochody
osobowe. Pozostaw tylko cechy ilościowe

``` r
samochody <- 
  read_xlsx("../dane-surowe/samochody.xlsx") |>
  filter(Typ == "osobowy") |>
  select_if(is.numeric)

samochody
```

Zadanie: Oceń współzależność pomiędzy pojemnością silnika a zużyciem
paliwa uwzględniają wpływ wagi samochodu.

``` r
cor(samochody$Pojemnosc_silnika, samochody$Zuzycie_paliwa)
```

``` r
pcor(x = select(samochody, Pojemnosc_silnika, Zuzycie_paliwa, Waga))
```

Uwzględniając wagę samochodu (przyjmując, że wszystkie samochody mają
dokładnie taką samą wagę) korelacja pomiędzy pojemnością silnika a
zużyciem paliwa jest słaba (cząstkowy współczynnik korelacji liniowej
wynosi ok. 0,08).
