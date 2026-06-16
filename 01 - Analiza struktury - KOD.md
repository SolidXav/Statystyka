# Kolokwium
Xav

Wczytaj potrzebne pakiety

``` r
library(dplyr)
library(tidyr)
```

# Przygotowanie danych

**Zadanie** Wczytaj dane z pliku rossmann-train.csv

``` r
rossmann <- read.csv("../dane-surowe/rossmann-train.csv")
head(rossmann)
```

**Zadanie** Sprawdź strukturę wczytanego zbioru

``` r
str(rossmann)
```

``` r
ncol(rossmann)
nrow(rossmann)
```

``` r
summary(rossmann)
```

``` r
table(sapply(rossmann, class))
rossmann |> sapply(class) |> table() # to samo co wyżej, ale w przetwarzaniu potokowym
```

**Zadanie** Wyznacz łączną sprzedaż poszczególnych sklepów w roku 2014.
Ogranicz zbiór do tych sklepów, które były obserwowane przez cały rok.

``` r
rossmann_sale <- 
  rossmann |>
  filter(substr(Date, 1, 4) == "2014") |>
  summarise(
    Sales = sum(Sales), 
    FirstDate = min(Date), 
    LastDate = max(Date), 
    .by = Store
  ) |>
  filter(FirstDate == "2014-01-01", LastDate == "2014-12-31") |>
  select(-FirstDate, -LastDate) |>
  arrange(Sales)

rossmann_sale
```

**Zadanie** Zapisz wartości rocznej sprzedaży do wektora sales

``` r
sales <- rossmann_sale$Sales
head(sales)
tail(sales)
```

# Prezentacja danych

**Zadanie** Zaprezentuj roczną sprzedaż za pomocą szeregu strukturalnego
przedziałowego

``` r
table(cut(sales, breaks = seq(from = 500000, to = 8000000, by = 500000)))
```

**Zadanie** Przedstaw rozkład rocznej sprzedaży za pomocą histogramu

``` r
hist(sales/1000000, breaks = seq(0.5, 8, 0.5), xlab = "Roczna sprzedaż (mln. euro)", ylab = "Liczba sklepów", main = "Rozkład rocznej sprzedaży sklepów sieci Rossmann")
```

**Komentarz** Po prawej stronie rozkładu obserwowane są wartości
odstające, co może świadczyć o asymetrii prawostronnej.

# Analiza struktury

## Miary klasyczne

**Zadanie** Wyznacz średnią arytmetyczną

``` r
# ze wzoru

sr <- sum(sales)/length(sales)
sr
```

``` r
sr <- mean(sales)
sr
```

**Interpretacja** Roczna sprzedaż badanych sklepów wynosi przeciętnie
2145342 euro.

**Zadanie** Wyznacz wariancję (z populacji)

$$S^2 = \frac{1}{n}\sum_{i=1}^n(x_i-\overline{x})^2$$

``` r
S2 <- 1/length(sales)*sum((sales-mean(sales))^2)
S2
```

``` r
var(sales) # wariancja z próby
var(sales)*(length(sales)-1)/length(sales) # wariancja z populacji
```

**Interpretacja** Nie interpretujemy, ze względu na to, że wariancja
wyrażona jest w euro do kwadratu.

**Zadanie**Wyznacz odchylenie standardowe (z populacji)

``` r
S <- sqrt(S2)
S
```

``` r
sd(sales) # odchylenie standardowe z próby
```

**Interpretacja** Wartości rocznej sprzedaży odchylają się od średniej
przeciętnie o 766840 euro

**Zadanie** Wyznacz współczynnik zmienności

``` r
V <- S/sr
V
```

**Interpretacja** Roczna sprzedaż charakteryzuje się umiarkowanym
zróżnicowaniem (według przyjętych progów)

**Zadanie** Napisz funkcję, która będzie wyznaczała k-ty moment cetralny
i k-ty moment centralny standaryzowany
$$m_k(x) = \frac{1}{n}\sum_{i=1}^n (x_i - \overline{x})^k$$
$$\alpha_k(x) = \frac{m_k(x)}{S^k(x)}$$

``` r
m <- function(x, k) {
  n <- length(x)
  sr <- mean(x)
  1/n*sum((x-sr)^k)
}

alfa <- function(x, k) {
  m(x, k)/sqrt(m(x, 2))^k
}
```

**Zadanie** Wyznacz współczynnik skośności i kurtozę rocznej sprzedaży

``` r
alfa3 <- alfa(sales, 3)
alfa3
```

**Interpretacja** Sprzedaż badanych sklepów charakteryzuje się silną
asymetrią prawostronną (po prawej stronie rozkładu obserwuje wartości
odstające)

``` r
alfa4 <- alfa(sales, 4)
alfa4
```

**Interpretacja** Sprzedaż charakteryzuje się rozkładem wysmukłym
(leptokurtycznym); silniejsza koncentracja wokół wartości centralnych
niż w rozkładzie normalnym; intensywność występowania wartośći
odstających jest większa niż w rozkładzie normalnym (grube ogony)

**Zadanie** Wyznacz medianę

``` r
Me <- median(sales)
Me
```

50% sklepów ma sprzedaż poniżej 2050574 euro.

**Zadanie** Napisz funkcję do wyznaczania kwantyli według algorytmu
“n-1”. Któremu typowi w funkcji quantile odpowiada ten algorytm?

``` r
kwantyl <- function(x, p) {
  n <- length(x)
  q <- p*(n-1) + 1
  k <- floor(q)
  d <- q - k
  Q <- (1-d)*x[k] + ifelse(k == n, 0, d*x[k+1]) # ifelse jest po to, aby p = 1 zwracało max jako wynik
  return(Q)
}

kwantyl(1:12, p = c(0.25, 0.5, 0.75))
quantile(1:12, probs = c(0.25, 0.5, 0.75), names = FALSE)
```

``` r
# przypomnienie działania lapply/sapply
lapply(1:9, sqrt)
lapply(1:9, \(i) 10^i)

sapply(1:9, sqrt)
sapply(1:9, \(i) 10^i)
```

``` r
kwantyl(1:11, p = c(0, 0.25, 0.5, 0.75, 1))
sapply(1:9, \(t) quantile(1:11, type = t)) |> t()
```

Algorytm 7 (domyślny) odpowiada algorytmowi “n-1”, który z kolei
odpowiada funkcji kwartyl.przedz.zamk w excelu.

**Zadanie** Wyznacz pozostałe kwartyle za pomocą algorytmu “n-1”.

``` r
Q1 <- quantile(sales, 0.25, names = FALSE)
Q3 <- quantile(sales, 0.75, names = FALSE)

Q1
Q3
```

25% sklepów ma sprzedaż poniżej 1622450 euro 75% sklepów ma sprzedaż
poniżej 2483658 euro

**Zadanie** Odchylenie ćwiartkowe i pozycyjny współczynnik zmienności

``` r
Q <- (Q3-Q1)/2
VQ <- Q/Me

Q
VQ
```

Sprzedaż sklepów odchyla się od mediany przeciętnie o 430604 euro
(połowa odchyla się o mniej, połowa o więcej).

Można ocenić zróznicowanie sprzedaży jako umiarkowane.

**Zadanie** Wyznacz pozycyjny współczynnik asymetrii.

``` r
AQ <- (Q1 + Q3 - 2*Me)/(2*Q)
AQ
```

Wśród 50% środkowych sklepów rozkład sprzedaży jest bliski symetrycznemu

**Zadanie** Czy wszystkie wartości są unikalne?

``` r
length(sales)
length(unique(sales))
```

**Przykład**

``` r
x <- c(rep(5, 11), 11:20)
x
```

``` r
quantile(x)
```

**Zadanie** Wczytaj plik wynagrodzenia.xlsx. Przygotuj zestawienie z
miarami analizy struktury (w wierszach przedsiębiorstwa, w kolumnach
miary).

średnia, odchylenie standardowe, współczynnik zmienności, współczynnik
asymetrii, kurtoza, kwartyl 1., mediana, kwartyl 3., odchylenie
ćwiartkowe, pozycyjny współczynnik zmienności, pozycyjny współczynnik
asymetrii

``` r
wynagrodzenia <- readxl::read_xlsx("../dane-surowe/wynagrodzenia.xlsx")
wynagrodzenia
```

``` r
lapply(wynagrodzenia, min)
```

``` r
analiza_struktury <- function(x) {
  
  xhat  <- mean(x)
  S  <- sqrt(m(x, 2))
  V <- S/xhat
  alfa3 <- alfa(x, 3)
  alfa4 <- alfa(x, 4)
  
  data.frame(xhat, S, V, alfa3, alfa4)
}

analiza_struktury(wynagrodzenia$`Wynagrodzenia-1`)
```

``` r
wynik <- lapply(wynagrodzenia, analiza_struktury)
wynik <- bind_rows(wynik, .id = "zmienna")
wynik

wynik <- lapply(wynagrodzenia, analiza_struktury)
wynik <- bind_rows(wynik)
rownames(wynik) <- colnames(wynagrodzenia)
wynik
```

``` r
wynagrodzenia |>
  pivot_longer(cols = everything(), names_to = "zmienna", values_to = "wartosc") |>
  summarise(
    avg = mean(wartosc), 
    S = sqrt(m(wartosc, 2)),
    alfa3 = alfa(wartosc, 3),
    alfa4 = alfa(wartosc, 4),
    .by = zmienna)

wynagrodzenia |>
  pivot_longer(cols = everything(), names_to = "zmienna", values_to = "wartosc") |>
  summarise(across(wartosc, list(avg = mean, S = \(x) sqrt(m(x, 2)))), .by = zmienna)
```
