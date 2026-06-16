# Analiza współzależności cech jakościowych – Notatki do kolokwium
Statystyka Opisowa

``` r
library(readxl)
library(DescTools)
```

``` r
remove(list = ls())
```

``` r
# Wczytanie danych
bael <- read_xlsx("../dane-surowe/bael.xlsx")

# Przypisanie zmiennych (cecha X i cecha Y)
x <- bael$WYKSZ   # cecha X – poziom wykształcenia (w wariantów)
y <- bael$KAT     # cecha Y – status na rynku pracy (k wariantów)
```

------------------------------------------------------------------------

# Tabela kontyngencji

### Pojęcie

Tabela kontyngencji (tabela krzyżowa) to dwuwymiarowe zestawienie
częstości, które pozwala zbadać łączny rozkład dwóch cech jakościowych
**X** i **Y**. Cecha **X** przyjmuje **w** wariantów, cecha **Y**
przyjmuje **k** wariantów.

- $n_{ij}$ – liczba jednostek posiadających jednocześnie wariant $X_i$
  cechy X i wariant $Y_j$ cechy Y (liczebności **empiryczne**)
- $n_{i\cdot}$ – suma $i$-tego wiersza; liczba jednostek przyjmujących
  wariant $X_i$
- $n_{\cdot j}$ – suma $j$-tej kolumny; liczba jednostek przyjmujących
  wariant $Y_j$
- $n$ – łączna liczebność zbiorowości

### Wzór

$$n = \sum_{i=1}^{w}\sum_{j=1}^{k} n_{ij}
= \sum_{i=1}^{w} n_{i\cdot}
= \sum_{j=1}^{k} n_{\cdot j}$$

### Typ danych wejściowych

Dwa wektory kategoryczne (jakościowe): `x` i `y` o tej samej długości,
gdzie każda obserwacja opisuje jedną jednostkę statystyczną.

### Kod w R

``` r
# Marginalne rozkłady brzegowe
table(x)           # liczebności wariantów cechy X
prop.table(table(x))  # udziały procentowe cechy X

table(y)           # liczebności wariantów cechy Y
prop.table(table(y))  # udziały procentowe cechy Y
```

``` r
# Tabela kontyngencji – łączny rozkład X i Y
table(x, y)                          # liczebności empiryczne nij
prop.table(table(x, y))              # udziały w całej zbiorowości
prop.table(table(x, y), margin = 1)  # udziały w wierszach (dla każdego Xi)
prop.table(table(x, y), margin = 2)  # udziały w kolumnach (dla każdego Yj)
```

``` r
# Wizualizacja – wykres mozaikowy
mosaicplot(table(x, y), color = c("green", "red", "blue"),
           main = "Wykres mozaikowy cech X i Y")
```

### Co ten wynik oznacza

- `table(x, y)` zwraca macierz liczebności empirycznych $n_{ij}$ – każda
  komórka to liczba obserwacji z daną kombinacją wariantów.
- `prop.table(..., margin = 1)` pokazuje strukturę cechy Y **wewnątrz**
  każdego wariantu cechy X – jeśli wiersze są podobne do siebie, cechy
  są niezależne.
- `prop.table(..., margin = 2)` – analogicznie dla kolumn.
- Wykres mozaikowy: szerokości kolumn odpowiadają rozkładowi X,
  wysokości segmentów – rozkładowi Y wewnątrz X. Niejednolite wysokości
  sygnalizują zależność.

------------------------------------------------------------------------

# Statystyka $\chi^2$ (chi-kwadrat)

### Pojęcie

Statystyka $\chi^2$ mierzy odchylenie obserwowanych liczebności
empirycznych od liczebności **teoretycznych** (oczekiwanych przy
założeniu braku zależności między cechami). Im większa wartość $\chi^2$,
tym silniejsza zależność.

Liczebności teoretyczne $\hat{n}_{ij}$ to wartości, jakie wystąpiłyby,
gdyby cechy X i Y były **całkowicie niezależne**.

### Wzór

**Wzór ogólny:**

$$\chi^2 = \sum_{i=1}^{w}\sum_{j=1}^{k} \frac{(n_{ij} - \hat{n}_{ij})^2}{\hat{n}_{ij}},
\qquad \text{gdzie} \quad \hat{n}_{ij} = \frac{n_{i\cdot} \cdot n_{\cdot j}}{n}$$

**Wzór skrócony dla tabeli 2×2** (cechy dwuwariantowe):

$$\chi^2 = \frac{n(ad - bc)^2}{(a+b)(c+d)(a+c)(b+d)}$$

gdzie $a, b, c, d$ to komórki tabeli 2×2.

### Typ danych wejściowych

Dwa wektory kategoryczne `x` i `y` (jak w tabeli kontyngencji).

### Kod w R

``` r
# Funkcja uniwersalna obliczająca chi-kwadrat ze wzoru
chi2 <- function(x, y) {
  nij     <- table(x, y)              # liczebności empiryczne
  ni.     <- table(x)                 # sumy wierszy
  n.j     <- table(y)                 # sumy kolumn
  n       <- sum(nij)                 # liczebność ogółem
  nij_hat <- (ni. %*% t(n.j)) / n    # liczebności teoretyczne
  sum((nij - nij_hat)^2 / nij_hat)   # statystyka chi-kwadrat
}

chi2(x, y)
```

``` r
# Alternatywnie – gotowa funkcja R (wynik identyczny)
test <- chisq.test(x = x, y = y)
test$statistic   # wartość chi-kwadrat
test$observed    # liczebności empiryczne nij
test$expected    # liczebności teoretyczne nij_hat
```

``` r
# Weryfikacja własności liczebności teoretycznych
rowSums(test$expected)  # sumy wierszy = brzegowe ni.
colSums(test$expected)  # sumy kolumn  = brzegowe n.j
# Wnioski: sumy brzegowe teoretycznych = sumy brzegowe empirycznych
```

### Co ten wynik oznacza

- $\chi^2 = 0$ → brak zależności (liczebności empiryczne = teoretyczne).
- Im większa wartość $\chi^2$, tym bardziej cechy są od siebie zależne.
- Sama wartość $\chi^2$ zależy od liczebności próby $n$ i liczby
  kategorii – **nie służy do porównań między zbiorami**. Do oceny siły
  zależności używa się miar znormalizowanych (T, V, C).
- Liczebności teoretyczne powinny być ≥ 5; jeśli nie – stosuje się
  poprawkę Yatesa.

------------------------------------------------------------------------

# Statystyka $\chi^2$ z poprawką Yatesa

### Pojęcie

Poprawka Yatesa jest modyfikacją wzoru na $\chi^2$, stosowaną gdy **co
najmniej jedna liczebność teoretyczna jest mniejsza niż 5** (dotyczy
głównie tabel 2×2). Poprawka zmniejsza wartość statystyki, korygując
przybliżenie rozkładu.

### Wzór

$$\chi^2_{\text{Yates}} = \sum_{i=1}^{2}\sum_{j=1}^{2}
\frac{\left(|n_{ij} - \hat{n}_{ij}| - 0{,}5\right)^2}{\hat{n}_{ij}}$$

### Typ danych wejściowych

Tabela kontyngencji **2×2** z małymi liczebnościami (min. jedna
$\hat{n}_{ij} < 5$).

### Kod w R

``` r
# Sprawdzenie, czy liczebności teoretyczne są wystarczające
test$expected
# Jeśli któraś wartość < 5, stosuj poprawkę Yatesa

# Poprawka Yatesa stosowana automatycznie przez chisq.test dla tabel 2x2
test_yates <- chisq.test(x = x, y = y, correct = TRUE)
test_yates$statistic
```

``` r
# Funkcja ręczna – poprawka Yatesa (dla tabeli 2x2)
chi2_yates <- function(x, y) {
  nij     <- table(x, y)
  ni.     <- table(x)
  n.j     <- table(y)
  n       <- sum(nij)
  nij_hat <- (ni. %*% t(n.j)) / n
  sum((abs(nij - nij_hat) - 0.5)^2 / nij_hat)
}

chi2_yates(x, y)
```

### Co ten wynik oznacza

- Stosuj gdy: tabela **2×2** i któraś $\hat{n}_{ij} < 5$.
- Wartość $\chi^2$ z poprawką jest **mniejsza** niż bez poprawki – jest
  to celowe, aby nie zawyżać siły zależności przy małych próbach.
- Jeśli wszystkie liczebności teoretyczne ≥ 5, poprawka nie jest
  wymagana – użyj standardowego wzoru.

------------------------------------------------------------------------

# Miary współzależności cech jakościowych

Miary zbudowane na bazie $\chi^2$ są znormalizowane do przedziału
$[0, 1]$, dzięki czemu umożliwiają ocenę **siły** zależności niezależnie
od $n$ i liczby kategorii.

**Progi interpretacyjne (umowne):**

| Wartość            | Interpretacja                        |
|--------------------|--------------------------------------|
| $(0{,}0;\, 0{,}2)$ | Słaby związek                        |
| $[0{,}2;\, 0{,}5)$ | Umiarkowany związek                  |
| $[0{,}5;\, 1{,}0]$ | Silny związek                        |
| $= 0$              | Brak zależności                      |
| $= 1$              | Zależność idealna (deterministyczna) |

------------------------------------------------------------------------

## Współczynnik T-Czuprowa

### Pojęcie

Współczynnik T-Czuprowa mierzy siłę zależności między dwiema cechami
jakościowymi. Uwzględnia liczbę kategorii obu cech poprzez człon
$\sqrt{(w-1)(k-1)}$.

### Wzór

$$T = \sqrt{\frac{\chi^2}{n \cdot \sqrt{(w-1)(k-1)}}}$$

### Typ danych wejściowych

Wektory `x`, `y`; obliczona wcześniej wartość `chi2_value`, `n`, `w`,
`k`.

### Kod w R

``` r
chi2_value <- chi2(x, y)
n <- length(x)
w <- length(unique(x))  # liczba wariantów cechy X
k <- length(unique(y))  # liczba wariantów cechy Y

T_Czuprow <- sqrt(chi2_value / (n * sqrt((w - 1) * (k - 1))))
T_Czuprow

# Alternatywnie – pakiet DescTools
TschuprowT(x = x, y = y)
```

### Co ten wynik oznacza

- $T \in [0, 1]$; $T = 0$ → brak zależności, $T = 1$ → zależność
  deterministyczna.
- Stosuj progi: słaby / umiarkowany / silny (tabela powyżej).
- Gdy $w = k$ (tyle samo kategorii), $T = V$ (Cramer).
- Interpretacja przykładowa: $T = 0{,}35$ → **umiarkowany** związek
  między cechami X i Y.

------------------------------------------------------------------------

## Współczynnik V-Cramera

### Pojęcie

Współczynnik V-Cramera to najpopularniejsza miara zależności dla tabel
kontyngencji. Normalizuje $\chi^2$ przez $n$ i **mniejszą** z liczb
kategorii, co czyni go porównywalnym dla różnych rozmiarów tabel.

### Wzór

$$V = \sqrt{\frac{\chi^2}{n \cdot \min\{w-1,\; k-1\}}}$$

### Typ danych wejściowych

Wektory `x`, `y`; obliczone wartości `chi2_value`, `n`, `w`, `k`.

### Kod w R

``` r
V_Cramer <- sqrt(chi2_value / (n * min(w - 1, k - 1)))
V_Cramer

# Alternatywnie – pakiet DescTools
CramerV(x = x, y = y)
```

### Co ten wynik oznacza

- $V \in [0, 1]$; interpretacja identyczna jak dla T.
- Gdy $w = k$: $V = T$.
- Gdy $w \neq k$: $V \geq T$ (V jest zawsze co najmniej tak duże jak T).
- Interpretacja przykładowa: $V = 0{,}35$ → **umiarkowany** związek.

------------------------------------------------------------------------

## Współczynnik C-Pearsona i skorygowany $C_{\text{kor}}$

### Pojęcie

Współczynnik kontyngencji C-Pearsona mierzy zależność, ale jego
maksymalna wartość **nie osiąga 1** i zależy od liczby kategorii $w$ i
$k$ – dlatego nie nadaje się do porównań między tabelami różnych
rozmiarów. Rozwiązaniem jest wersja **skorygowana** $C_{\text{kor}}$,
która dzieli $C$ przez jego teoretyczne maksimum $C_{\max}$, unormowując
wynik do $[0, 1]$.

### Wzór

$$C = \sqrt{\frac{\chi^2}{\chi^2 + n}}$$

$$C_{\max} = \frac{\sqrt{\dfrac{w-1}{w}} + \sqrt{\dfrac{k-1}{k}}}{2}$$

$$C_{\text{kor}} = \frac{C}{C_{\max}}$$

### Typ danych wejściowych

Wektory `x`, `y`; obliczone wartości `chi2_value`, `n`, `w`, `k`.

### Kod w R

``` r
C_Pearson <- sqrt(chi2_value / (chi2_value + n))
C_Pearson

C_max <- (sqrt((w - 1) / w) + sqrt((k - 1) / k)) / 2
C_max

C_kor <- C_Pearson / C_max
C_kor

# Alternatywnie – pakiet DescTools
ContCoef(x = x, y = y)               # C-Pearson (nieskorygowany)
ContCoef(x = x, y = y, correct = TRUE) # C-Pearson skorygowany
```

### Co ten wynik oznacza

- $C \in [0, C_{\max})$; $C$ **nigdy nie osiąga 1** przy skończonej
  liczbie kategorii → nie porównuj go wprost z T i V.
- $C_{\text{kor}} \in [0, 1]$ → w pełni porównywalny z T i V.
- $C_{\max} < 1$ zawsze; im więcej kategorii, tym $C_{\max}$ bliższe 1.
- Stosuj te same progi interpretacyjne co dla T i V (na podstawie
  $C_{\text{kor}}$).
- Interpretacja przykładowa: $C_{\text{kor}} = 0{,}38$ → **umiarkowany**
  związek między cechami X i Y.

------------------------------------------------------------------------

# Zestawienie wszystkich miar – podsumowanie

### Kod w R

``` r
# Wszystkie miary współzależności w jednym bloku
chi2_value <- chi2(x, y)
n  <- length(x)
w  <- length(unique(x))
k  <- length(unique(y))

T_Czuprow <- sqrt(chi2_value / (n * sqrt((w - 1) * (k - 1))))
V_Cramer  <- sqrt(chi2_value / (n * min(w - 1, k - 1)))
C_Pearson <- sqrt(chi2_value / (chi2_value + n))
C_max     <- (sqrt((w - 1) / w) + sqrt((k - 1) / k)) / 2
C_kor     <- C_Pearson / C_max

cat("T-Czuprow  :", round(T_Czuprow, 4), "\n")
cat("V-Cramer   :", round(V_Cramer,  4), "\n")
cat("C-Pearson  :", round(C_Pearson, 4), "\n")
cat("C_max      :", round(C_max,     4), "\n")
cat("C_kor      :", round(C_kor,     4), "\n")

# Pomiędzy badanymi cechami występuje umiarkowana współzależność
```

### Co ten wynik oznacza

| Miara | Wzór normalizujący | Zakres | Uwagi |
|----|----|----|----|
| T-Czuprow | $\sqrt{(w-1)(k-1)}$ | $[0,1]$ | Dokładniejszy przy $w \neq k$ |
| V-Cramer | $\min(w-1, k-1)$ | $[0,1]$ | Najpopularniejszy |
| C-Pearson | – | $[0, C_{\max})$ | Nie używać samodzielnie |
| $C_{\text{kor}}$ | $C_{\max}$ | $[0,1]$ | Porównywalny z T i V |

- Gdy wyniki T, V, $C_{\text{kor}}$ są zbliżone → interpretacja jest
  spójna i wiarygodna.
- Gdy $w = k$: T = V (możesz to sprawdzić jako weryfikację obliczeń).
- Ostateczna interpretacja: wskaż wartość wybranej miary, określ siłę
  związku (słaby / umiarkowany / silny) i sformułuj wniosek w kontekście
  badanych cech, np.: *„Pomiędzy poziomem wykształcenia a statusem na
  rynku pracy występuje umiarkowana współzależność ($V = 0{,}35$).”*
