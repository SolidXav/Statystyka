# 📊 Analiza Współzależności Cech Ilościowych — Notatki na Kolokwium

> **Temat:** Korelacja | **Kurs:** Analiza Danych w R | Uniwersytet Ekonomiczny w Poznaniu

---

## Spis treści

1. [Wymagane pakiety](#1-wymagane-pakiety)
2. [Wykres rozrzutu](#2-wykres-rozrzutu)
3. [Współczynnik korelacji Pearsona](#3-współczynnik-korelacji-pearsona)
4. [Macierz korelacji](#4-macierz-korelacji)
5. [Współczynnik korelacji Spearmana](#5-współczynnik-korelacji-spearmana)
6. [Wpływ obserwacji odstających](#6-wpływ-obserwacji-odstających)
7. [Korelacja cząstkowa](#7-korelacja-cząstkowa)
8. [Korelacja wieloraka](#8-korelacja-wieloraka)
9. [Pobieranie danych finansowych (quantmod)](#9-pobieranie-danych-finansowych-quantmod)
10. [Wzory — ściągawka](#10-wzory--ściągawka)
11. [Typowe zadania kolokwialne](#11-typowe-zadania-kolokwialne)

---

## 1. Wymagane pakiety

```r
library(quantmod)  # getSymbols() — pobieranie danych giełdowych z Yahoo Finance
library(dplyr)     # filter(), select(), select_if(), pipe |>
library(readxl)    # read_xlsx() — wczytywanie plików Excel
library(ppcor, include.only = "pcor")  # pcor() — korelacja cząstkowa

# Wyczyść środowisko przed analizą
remove(list = ls())
```

---

## 2. Wykres rozrzutu

**Do czego:** Pierwsza, wizualna ocena współzależności między dwiema cechami ilościowymi.

**Dane wejściowe:** Dwa wektory numeryczne (lub dwie kolumny ramki danych) tej samej długości.

```r
# Podstawowy wykres rozrzutu
plot(x = wektor_x, y = wektor_y)

# Przykład z ramką danych:
plot(x = close_df$Zloto, y = close_df$Srebro)
```

**Co obserwować na wykresie:**

| Kształt chmury punktów | Interpretacja |
|---|---|
| Punkty układają się wzdłuż linii rosnącej | Korelacja **dodatnia** |
| Punkty układają się wzdłuż linii malejącej | Korelacja **ujemna** |
| Punkty bardzo blisko linii prostej | Korelacja **silna** |
| Punkty rozrzucone szeroko | Korelacja **słaba lub brak** |
| Punkty układają się w krzywą (np. parabolę) | Zależność **nieliniowa** (Pearson zawiedzie!) |

> ⚠️ **Zawsze rysuj wykres rozrzutu przed obliczeniem r!** Wysoka wartość Pearsona nie musi oznaczać związku liniowego — może być artefaktem outlierów lub zależności nieliniowej.

---

## 3. Współczynnik korelacji Pearsona

### Wzór

$$r_{XY} = \frac{Cov(X,Y)}{S_X \cdot S_Y}$$

gdzie kowariancja:

$$Cov(X,Y) = \frac{1}{n} \sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y})$$

### Interpretacja

**Kierunek:**
- $r_{XY} > 0$ → korelacja **dodatnia** (obie cechy rosną razem)
- $r_{XY} < 0$ → korelacja **ujemna** (jedna rośnie, druga maleje)

**Siła:**

| Wartość $|r_{XY}|$ | Interpretacja |
|---|---|
| $= 0$ | **Brak** korelacji |
| $(0;\; 0{,}2]$ | **Bardzo słaba** |
| $(0{,}2;\; 0{,}4]$ | **Słaba** |
| $(0{,}4;\; 0{,}7]$ | **Umiarkowana** |
| $(0{,}7;\; 0{,}9]$ | **Silna** |
| $(0{,}9;\; 1{,}0)$ | **Bardzo silna** |
| $= 1$ | **Idealna** (wszystkie punkty na jednej prostej) |

### Kod R

**Dane wejściowe:** dwa wektory numeryczne `x` i `y` tej samej długości, bez wartości NA (lub z `use = "complete.obs"`).

```r
# Podstawowe użycie — dwa wektory
cor(x = wektor_x, y = wektor_y)

# Przykład:
cor(x = close_df$Zloto, y = close_df$Srebro)

# Jeśli są NA:
cor(x, y, use = "complete.obs")

# Dla całej ramki danych (wszystkie pary kolumn naraz):
cor(ramka_danych[, -1])   # -1 jeśli pierwsza kolumna to np. daty
```

**Wynik:** liczba z przedziału $[-1, 1]$.

**Przykładowy komentarz do wyników:**
> "Pomiędzy cenami złota a cenami srebra obserwujemy **bardzo silną dodatnią** korelację liniową (r ≈ 0,97)."

---

## 4. Macierz korelacji

### Do czego

Gdy masz więcej niż dwie cechy i chcesz zbadać **wszystkie pary** jednocześnie.

### Kod R

**Dane wejściowe:** ramka danych lub macierz zawierająca **wyłącznie kolumny numeryczne**.

```r
# Oblicz macierz korelacji
macierz_korelacji <- cor(ramka_danych[, -1])  # usuń kolumny nienumeryczne

# Ustaw przekątną na NA (korelacja cechy z samą sobą = 1, nie interesuje nas)
diag(macierz_korelacji) <- NA

# Wyświetl
macierz_korelacji
```

### Znajdowanie ekstremalnych par

```r
# A) Najsilniejsza współzależność (najwyższe |r|, niezależnie od znaku)
arrayInd(which.max(abs(macierz_korelacji)), .dim = dim(macierz_korelacji))

# B) Najsłabsza współzależność (najniższe |r|)
arrayInd(which.min(abs(macierz_korelacji)), .dim = dim(macierz_korelacji))

# C) Najsilniejsza DODATNIA współzależność
macierz_dodatnie <- macierz_korelacji
macierz_dodatnie[macierz_dodatnie <= 0] <- NA
arrayInd(which.max(macierz_dodatnie), .dim = dim(macierz_korelacji))

# D) Najsilniejsza UJEMNA współzależność
macierz_ujemne <- macierz_korelacji
macierz_ujemne[macierz_ujemne >= 0] <- NA
arrayInd(which.max(abs(macierz_ujemne)), .dim = dim(macierz_korelacji))

# E/F) Z jakim surowcem najsilniej/najsłabiej skorelowane jest złoto?
# Zakładając, że złoto to kolumna "Zloto":
zloto_row <- macierz_korelacji["Zloto", ]
names(which.max(abs(zloto_row)))   # najsilniej
names(which.min(abs(zloto_row)))   # najsłabiej
```

> ⚠️ `arrayInd()` zwraca numery wierszy i kolumn — żeby uzyskać nazwy, użyj:
> ```r
> idx <- arrayInd(which.max(abs(macierz_korelacji)), .dim = dim(macierz_korelacji))
> rownames(macierz_korelacji)[idx[1]]
> colnames(macierz_korelacji)[idx[2]]
> ```

**Macierz korelacji (notacja matematyczna):**

$$P = \begin{pmatrix} 1 & r_{XY} & r_{XZ} \\ r_{YX} & 1 & r_{YZ} \\ r_{ZX} & r_{ZY} & 1 \end{pmatrix}$$

Macierz jest symetryczna: $r_{XY} = r_{YX}$.

---

## 5. Współczynnik korelacji Spearmana

### Wzór

$$r_S = \frac{Cov(R_X, R_Y)}{S_{R_X} \cdot S_{R_Y}}$$

gdzie $R_X$, $R_Y$ to **rangi** (pozycje po sortowaniu rosnącym) odpowiednio cech X i Y.

**Uproszczony wzór (bez rang wiązanych):**

$$r_S = 1 - \frac{6 \sum_{i=1}^{n}(r_{x,i} - r_{y,i})^2}{n(n^2-1)}$$

### Nadawanie rang

Rangi przypisujemy przez posortowanie wartości rosnąco i nadanie pozycji 1, 2, 3, ...

- Jeśli wartości się **powtarzają** → ranga wiązana = średnia arytmetyczna pozycji, na których ta wartość wystąpiła.

```r
# Jak działa rank():
x <- c(10, 30, 20, 30, 10)
rank(x)
# [1] 1.5 4.5 3.0 4.5 1.5   ← rangi wiązane dla powtórzeń
```

### Kiedy używać Spearmana zamiast Pearsona?

| Sytuacja | Użyj |
|---|---|
| Dane mają outliery | **Spearman** (bardziej odporny) |
| Zależność jest monotonична, ale nieliniowa | **Spearman** |
| Dane na skali porządkowej (np. rankingi) | **Spearman** |
| Dane normalne, bez outlierów, zależność liniowa | **Pearson** |

### Kod R

**Dane wejściowe:** identyczne jak dla Pearsona — dwa wektory numeryczne.

```r
# Spearman
cor(x = wektor_x, y = wektor_y, method = "spearman")

# Równoważnie (obliczony ręcznie przez rangi):
cor(x = rank(wektor_x), y = rank(wektor_y))

# Przykład:
cor(x = close_df$Zloto, y = close_df$Srebro, method = "spearman")
```

**Interpretacja siły i kierunku** — taka sama skala jak dla Pearsona.

---

## 6. Wpływ obserwacji odstających

### Eksperyment 1 — outlier dodany do danych

```r
# Dodanie wartości odstającej do wektorów
zloto_outlier <- c(close_df$Zloto, 8000)
srebro_outlier <- c(close_df$Srebro, 20)

plot(zloto_outlier, srebro_outlier)  # widać wyraźnie punkt odstający

cor(zloto_outlier, srebro_outlier)                    # Pearson — wrażliwy!
cor(zloto_outlier, srebro_outlier, method = "spearman")  # Spearman — odporny
```

### Eksperyment 2 — outliery w danych losowych

```r
set.seed(1234)  # dla powtarzalności wyników

# Bez outlieru — dwa nieskorelowane wektory
x <- rnorm(50, 1000, 10)
y <- rnorm(50, 1000, 10)

plot(x, y)
cor(x, y)              # powinno być bliskie 0
cor(x, y, method = "spearman")

# Po dodaniu outlieru
x <- c(x, 100)
y <- c(y, 100)

plot(x, y)
cor(x, y)              # Pearson mocno wzrośnie przez outlier!
cor(x, y, method = "spearman")  # Spearman zmieni się mało
```

**Wniosek:** Pearson jest czuły na outliery — jeden punkt może drastycznie zmienić wynik. Spearman jest **odporny**, bo liczy tylko rangi (kolejność), nie wartości bezwzględne.

---

## 7. Korelacja cząstkowa

### Idea

Mierzy związek **między dwiema cechami** przy jednoczesnym **wyeliminowaniu wpływu** jednej (lub więcej) innych cech.

**Przykład z wykładu:** Liczba strażaków i poziom zniszczeń są pozornie dodatnio skorelowane — ale obie te wielkości zależą od **wielkości pożaru**. Po uwzględnieniu wielkości pożaru (korelacja cząstkowa), korelacja między liczbą strażaków a zniszczeniami **zanika**.

### Wzór (dla 3 cech X, Y, Z)

$$r_{XY \cdot Z} = \frac{r_{XY} - r_{XZ} \cdot r_{YZ}}{\sqrt{(1 - r_{XZ}^2)(1 - r_{YZ}^2)}}$$

Ogólnie przez dopełnienia algebraiczne macierzy korelacji:

$$r_{XY \cdot Z} = \frac{-P_{12}}{\sqrt{P_{11} \cdot P_{22}}}$$

gdzie $P_{ij}$ to dopełnienie algebraiczne macierzy korelacji $P$.

### Kod R

**Dane wejściowe:** `pcor()` przyjmuje **ramkę danych** (lub macierz) z **dokładnie tymi kolumnami**, które chcemy uwzględnić — zmienna kontrolowana musi być **trzecią kolumną**.

```r
library(ppcor, include.only = "pcor")

# Składnia: pcor(x) gdzie x to ramka danych z kolumnami:
# [zmienna 1, zmienna 2, ..., zmienne kontrolowane]

# Przykład: korelacja Pojemnosc_silnika ~ Zuzycie_paliwa,
# kontrolując wpływ Wagi:
pcor(x = select(samochody, Pojemnosc_silnika, Zuzycie_paliwa, Waga))
```

**Co zwraca `pcor()`:**

```
$estimate       # macierz cząstkowych współczynników korelacji
$p.value        # macierz p-wartości (test istotności)
$statistic      # statystyki testowe
$n              # liczba obserwacji
$gp             # liczba kontrolowanych zmiennych
$method         # "pearson" (domyślnie)
```

Interesuje Cię `$estimate[1,2]` — cząstkowy współczynnik korelacji między pierwszą a drugą zmienną, z wyeliminowaniem wpływu pozostałych.

```r
wynik <- pcor(x = select(samochody, Pojemnosc_silnika, Zuzycie_paliwa, Waga))
wynik$estimate[1, 2]  # sam współczynnik
wynik$p.value[1, 2]   # p-wartość
```

**Interpretacja:**
> "Uwzględniając wagę samochodu (przy założeniu, że wszystkie samochody mają taką samą wagę), korelacja między pojemnością silnika a zużyciem paliwa jest **słaba** (r_cząstkowe ≈ 0,08). Wcześniejsza wysoka korelacja całkowita była spowodowana wpływem wagi — cięższe auta mają większe silniki i zużywają więcej paliwa."

---

## 8. Korelacja wieloraka

### Idea

Mierzy współzależność **jednej cechy ze wszystkimi pozostałymi łącznie**.

### Wzór

$$r_{X \cdot YZ} = \sqrt{1 - \frac{|P|}{P_{11}}}$$

Rozwinięcie:

$$r_{X \cdot YZ} = \sqrt{\frac{r_{XY}^2 + r_{XZ}^2 - 2 r_{XY} r_{XZ} r_{YZ}}{1 - r_{YZ}^2}}$$

gdzie:
- $|P|$ — wyznacznik macierzy korelacji
- $P_{11}$ — dopełnienie algebraiczne elementu $(1,1)$

**Własności:**
- Wartości z przedziału $[0, 1]$ (zawsze nieujemna)
- Im bliżej 1, tym silniejsza zależność wybranej cechy z pozostałymi łącznie

---

## 9. Pobieranie danych finansowych (quantmod)

### Pobieranie pojedynczego instrumentu

```r
# Dane wejściowe:
# - symbol: string z tickerem Yahoo Finance (np. "GC=F" = złoto)
# - src: źródło danych (tutaj zawsze "yahoo")
# - from / to: daty w formacie "RRRR-MM-DD"
# - auto.assign = FALSE: zwróć obiekt zamiast przypisywać do globalnej zmiennej

dane_zloto <- getSymbols("GC=F",
                          src = "yahoo",
                          from = "2025-11-01",
                          to = "2025-12-31",
                          auto.assign = FALSE)
```

### Wybrane tickery surowców na Yahoo Finance

| Symbol | Surowiec |
|---|---|
| `GC=F` | Złoto |
| `SI=F` | Srebro |
| `CL=F` | Ropa WTI |
| `BZ=F` | Ropa Brent |
| `NG=F` | Gaz ziemny |
| `HG=F` | Miedź |
| `PL=F` | Platyna |
| `PA=F` | Pallad |
| `ZW=F` | Pszenica |
| `ZC=F` | Kukurydza |
| `ZS=F` | Soja |
| `CC=F` | Kakao |
| `KC=F` | Kawa |
| `SB=F` | Cukier |

### Pobieranie wielu instrumentów i łączenie w ramkę danych

```r
symbole <- c("GC=F", "SI=F", "CL=F", "BZ=F", "NG=F",
             "HG=F", "PL=F", "PA=F", "ZW=F", "ZC=F",
             "ZS=F", "CC=F", "KC=F", "SB=F")

nazwy <- c("Zloto", "Srebro", "Ropa_WTI", "Ropa_Brent", "Gaz_ziemny",
           "Miedz", "Platyna", "Pallad", "Pszenica", "Kukurydza",
           "Soja", "Kakao", "Kawa", "Cukier")

# Pobierz ceny zamknięcia dla każdego symbolu
close_list <-
  lapply(symbole, \(x) {
    dane_x <- getSymbols(x, src = "yahoo",
                         from = "2025-11-01", to = "2025-12-31",
                         auto.assign = FALSE)
    dane_x <- as.data.frame(dane_x)
    dane_x$Date <- rownames(dane_x)
    rownames(dane_x) <- NULL
    dane_x <- dane_x[, c("Date", paste0(x, ".Close"))]  # zostaw tylko datę i cenę zamknięcia
    dane_x
  })

# Złącz wszystkie ramki po kolumnie Date (intersection dat)
close_df <- Reduce(merge, close_list)

# Nadaj czytelne nazwy kolumnom (poza Date)
colnames(close_df)[-1] <- nazwy

head(close_df)
```

### Wczytywanie danych z Excela

```r
samochody <- read_xlsx("../dane-surowe/samochody.xlsx")

# Filtrowanie i selekcja
samochody <- samochody |>
  filter(Typ == "osobowy") |>      # zostaw tylko samochody osobowe
  select_if(is.numeric)            # zostaw tylko kolumny numeryczne (usuwa np. Typ, Marka)

head(samochody)
```

---

## 10. Wzory — ściągawka

### Pearson

$$r_{XY} = \frac{\sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y})}{\sqrt{\sum_{i=1}^{n}(x_i-\bar{x})^2 \cdot \sum_{i=1}^{n}(y_i-\bar{y})^2}}$$

### Spearman (bez rang wiązanych)

$$r_S = 1 - \frac{6 \sum_{i=1}^{n} d_i^2}{n(n^2 - 1)}, \quad d_i = r_{x,i} - r_{y,i}$$

### Korelacja cząstkowa (3 zmienne)

$$r_{XY \cdot Z} = \frac{r_{XY} - r_{XZ} \cdot r_{YZ}}{\sqrt{(1 - r_{XZ}^2)(1 - r_{YZ}^2)}}$$

### Korelacja wieloraka

$$r_{X \cdot YZ} = \sqrt{\frac{r_{XY}^2 + r_{XZ}^2 - 2 r_{XY} r_{XZ} r_{YZ}}{1 - r_{YZ}^2}}$$

---

## 11. Typowe zadania kolokwialne

### Zadanie A–F: Analiza macierzy korelacji surowców

```r
# Przygotowanie
macierz_korelacji <- cor(close_df[, -1])
diag(macierz_korelacji) <- NA

# A: Najsilniejsza współzależność (absolutna, bez względu na znak)
idx_A <- arrayInd(which.max(abs(macierz_korelacji)), .dim = dim(macierz_korelacji))
cat("A:", rownames(macierz_korelacji)[idx_A[1]], "—", colnames(macierz_korelacji)[idx_A[2]], "\n")

# B: Najsłabsza współzależność
idx_B <- arrayInd(which.min(abs(macierz_korelacji)), .dim = dim(macierz_korelacji))
cat("B:", rownames(macierz_korelacji)[idx_B[1]], "—", colnames(macierz_korelacji)[idx_B[2]], "\n")

# C: Najsilniejsza DODATNIA
m_pos <- macierz_korelacji
m_pos[m_pos <= 0] <- NA
idx_C <- arrayInd(which.max(m_pos), .dim = dim(macierz_korelacji))
cat("C:", rownames(macierz_korelacji)[idx_C[1]], "—", colnames(macierz_korelacji)[idx_C[2]], "\n")

# D: Najsilniejsza UJEMNA
m_neg <- macierz_korelacji
m_neg[m_neg >= 0] <- NA
idx_D <- arrayInd(which.max(abs(m_neg)), .dim = dim(macierz_korelacji))
cat("D:", rownames(macierz_korelacji)[idx_D[1]], "—", colnames(macierz_korelacji)[idx_D[2]], "\n")

# E: Najsilniej skorelowany surowiec ze złotem
zloto_row <- macierz_korelacji["Zloto", ]
cat("E (najsilniej):", names(which.max(abs(zloto_row))),
    "r =", round(max(abs(zloto_row), na.rm = TRUE), 3), "\n")

# F: Najsłabiej skorelowany surowiec ze złotem
cat("F (najsłabiej):", names(which.min(abs(zloto_row))),
    "r =", round(min(abs(zloto_row), na.rm = TRUE), 3), "\n")
```

### Zadanie: Korelacja cząstkowa z kontrolą zmiennej zakłócającej

```r
library(ppcor, include.only = "pcor")

# Scenariusz: czy A jest skorelowane z B, jeśli wyeliminujemy wpływ C?
wynik <- pcor(select(dane, A, B, C))

r_calkowita  <- cor(dane$A, dane$B)
r_czastkowa  <- wynik$estimate[1, 2]
p_wartosc    <- wynik$p.value[1, 2]

cat("Korelacja całkowita A~B:", round(r_calkowita, 3), "\n")
cat("Korelacja cząstkowa A~B|C:", round(r_czastkowa, 3), "\n")
cat("p-wartość:", round(p_wartosc, 4), "\n")

# Interpretacja:
# Jeśli r_calkowita >> r_czastkowa → C "odpowiada" za dużą część korelacji
# Jeśli r_czastkowa ≈ r_calkowita  → C nie wpływa na związek A~B
```

### Zadanie: Porównanie Pearson vs Spearman z outlierami

```r
set.seed(42)
x <- rnorm(50, mean = 100, sd = 10)
y <- rnorm(50, mean = 100, sd = 10)

# Wyniki bez outlieru
r_pearson_bez  <- cor(x, y)
r_spearman_bez <- cor(x, y, method = "spearman")

# Dodaj outlier
x_out <- c(x, 500)
y_out <- c(y, 500)

# Wyniki z outlierem
r_pearson_z  <- cor(x_out, y_out)
r_spearman_z <- cor(x_out, y_out, method = "spearman")

cat("Pearson  bez outlieru:", round(r_pearson_bez, 3),  "| z outlierem:", round(r_pearson_z, 3),  "\n")
cat("Spearman bez outlieru:", round(r_spearman_bez, 3), "| z outlierem:", round(r_spearman_z, 3), "\n")
# Wniosek: Pearson silnie zmienił wartość, Spearman pozostał stabilny
```

---

## Szybkie przypomnienie — co kiedy używać

```
Mam dwie cechy ilościowe → plot() + cor()
  ├─ Dane bez outlierów, zależność liniowa → Pearson (domyślne)
  ├─ Są outliery / zależność monotonická nieliniowa → Spearman (method = "spearman")
  └─ Chcę wykluczyć wpływ trzeciej zmiennej → pcor() z ppcor

Mam wiele cech → cor(ramka) + diag() <- NA
  └─ Szukam par ekstremalnych → which.max/min(abs()) + arrayInd()
```

---

*Notatki przygotowane na podstawie wykładu i kodu laboratoryjnego — Analiza Współzależności Cech Ilościowych*
