# 📊 Analiza Współzależności Cech Ilościowych — Notatki na Kolokwium

---

## Spis treści

1. [Wymagane pakiety](#1-wymagane-pakiety)
2. [Wczytywanie i przygotowanie danych](#2-wczytywanie-i-przygotowanie-danych)
3. [Wykres rozrzutu](#3-wykres-rozrzutu)
4. [Współczynnik korelacji Pearsona](#4-współczynnik-korelacji-pearsona)
5. [Macierz korelacji](#5-macierz-korelacji)
6. [Współczynnik korelacji Spearmana](#6-współczynnik-korelacji-spearmana)
7. [Wpływ obserwacji odstających](#7-wpływ-obserwacji-odstających)
8. [Korelacja cząstkowa](#8-korelacja-cząstkowa)
9. [Korelacja wieloraka](#9-korelacja-wieloraka)
10. [Pobieranie danych finansowych (quantmod)](#10-pobieranie-danych-finansowych-quantmod)
11. [Wzory — ściągawka z kodem](#11-wzory--ściągawka-z-kodem)
12. [Typowe zadania kolokwialne — gotowy kod](#12-typowe-zadania-kolokwialne--gotowy-kod)

---

## 1. Wymagane pakiety

```r
library(quantmod)                     # getSymbols() — pobieranie danych giełdowych
library(dplyr)                        # filter(), select(), select_if(), pipe |>
library(readxl)                       # read_xlsx() — wczytywanie plików Excel
library(ppcor, include.only = "pcor") # pcor() — korelacja cząstkowa

# Wyczyść środowisko przed analizą
remove(list = ls())
```

---

## 2. Wczytywanie i przygotowanie danych

### Z pliku Excel

```r
# Wczytaj cały plik
df <- read_xlsx("sciezka/do/pliku.xlsx")

# Filtruj wiersze + zostaw tylko kolumny numeryczne
df <- df |>
  filter(Kolumna_kategorialna == "wartość") |>  # np. filter(Typ == "osobowy")
  select_if(is.numeric)                          # usuwa kolumny tekstowe/kategorialne

# Sprawdź strukturę
head(df)
str(df)
colnames(df)  # nazwy kolumn — ważne do dalszych operacji
```

### Jak się odwołać do kolumny data frame?

```r
# Trzy równoważne sposoby — wszystkie dają ten sam wektor wartości:
df$NazwaKolumny          # operator $  — najczęściej używany
df[["NazwaKolumny"]]     # operator [[]] — gdy nazwa jest w zmiennej
df[, "NazwaKolumny"]     # indeksowanie — daje data.frame; użyj drop=TRUE dla wektora

# Przykład z kolokwium:
samochody$Pojemnosc_silnika
samochody[["Zuzycie_paliwa"]]
```

---

## 3. Wykres rozrzutu

**Do czego:** Wizualna ocena współzależności — zawsze rób **przed** liczeniem korelacji.

**Dane wejściowe:** dwie kolumny numeryczne z tej samej ramki danych (lub dwa wektory tej samej długości).

```r
# Składnia podstawowa:
plot(x = df$KolumnaX, y = df$KolumnaY)

# Przykład — złoto vs srebro:
plot(x = close_df$Zloto, y = close_df$Srebro)

# Przykład — pojemność silnika vs zużycie paliwa:
plot(x = samochody$Pojemnosc_silnika, y = samochody$Zuzycie_paliwa)
```

**Co obserwować na wykresie:**

| Kształt chmury punktów | Interpretacja |
|---|---|
| Punkty układają się wzdłuż linii **rosnącej** | Korelacja **dodatnia** |
| Punkty układają się wzdłuż linii **malejącej** | Korelacja **ujemna** |
| Punkty **bardzo blisko** linii prostej | Korelacja **silna** |
| Punkty **rozrzucone** szeroko | Korelacja **słaba lub brak** |
| Punkty układają się w **krzywą** (np. parabolę) | Zależność **nieliniowa** — Pearson zawiedzie! |
| Jeden punkt daleko od reszty | **Obserwacja odstająca** — użyj Spearmana |

> ⚠️ **Zawsze rysuj wykres rozrzutu przed obliczeniem r!** Wartość r ≈ 0 nie musi oznaczać braku związku — może być zależność nieliniowa (np. U-kształtna).

---

## 4. Współczynnik korelacji Pearsona

### Co mierzy

Siłę i kierunek **liniowej** zależności między dwiema cechami ilościowymi.

### Wzór matematyczny

$$r_{XY} = \frac{Cov(X,Y)}{S_X \cdot S_Y} = \frac{\frac{1}{n}\sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y})}{\sqrt{\frac{1}{n}\sum(x_i-\bar{x})^2} \cdot \sqrt{\frac{1}{n}\sum(y_i-\bar{y})^2}}$$

### Wzór jako kod R — krok po kroku na kolumnach data frame

```r
# Zakładamy: df to ramka danych, V1 i V3 to nazwy kolumn

# Krok 1: wyciągnij kolumny jako wektory
x <- df$V1
y <- df$V3

# Krok 2: oblicz składniki wzoru ręcznie (żeby rozumieć co liczy R)
n       <- length(x)
x_sr    <- mean(x)               # x̄
y_sr    <- mean(y)               # ȳ
cov_xy  <- mean((x - x_sr) * (y - y_sr))  # Cov(X,Y) = (1/n) * Σ(xi - x̄)(yi - ȳ)
sd_x    <- sqrt(mean((x - x_sr)^2))        # Sx — odchylenie standardowe populacyjne
sd_y    <- sqrt(mean((y - y_sr)^2))        # Sy

r_reczny <- cov_xy / (sd_x * sd_y)
r_reczny

# Krok 3: to samo jedną funkcją — wynik identyczny
r <- cor(x = df$V1, y = df$V3)  # domyślnie method = "pearson"
r

# Jeśli są braki danych (NA):
r <- cor(x = df$V1, y = df$V3, use = "complete.obs")
```

### Interpretacja wyniku

**Kierunek (znak):**
- $r > 0$ → korelacja **dodatnia** — gdy V1 rośnie, V3 też rośnie
- $r < 0$ → korelacja **ujemna** — gdy V1 rośnie, V3 maleje
- $r = 0$ → **brak** korelacji liniowej

**Siła (wartość bezwzględna):**

| $|r|$ | Słowna interpretacja |
|---|---|
| $= 0$ | **Brak** korelacji |
| $(0;\; 0{,}2]$ | **Bardzo słaba** |
| $(0{,}2;\; 0{,}4]$ | **Słaba** |
| $(0{,}4;\; 0{,}7]$ | **Umiarkowana** |
| $(0{,}7;\; 0{,}9]$ | **Silna** |
| $(0{,}9;\; 1{,}0)$ | **Bardzo silna** |
| $= 1$ | **Idealna** (wszystkie punkty na jednej prostej) |

```r
# Automatyczna interpretacja siły — funkcja pomocnicza
interpretuj_r <- function(r) {
  r_abs <- abs(r)
  kierunek <- ifelse(r > 0, "dodatnia", ifelse(r < 0, "ujemna", "brak kierunku"))
  sila <- dplyr::case_when(
    r_abs == 0              ~ "brak",
    r_abs <= 0.2            ~ "bardzo słaba",
    r_abs <= 0.4            ~ "słaba",
    r_abs <= 0.7            ~ "umiarkowana",
    r_abs <= 0.9            ~ "silna",
    r_abs < 1               ~ "bardzo silna",
    r_abs == 1              ~ "idealna"
  )
  cat("r =", round(r, 4), "→", sila, kierunek, "korelacja liniowa\n")
}

r <- cor(df$V1, df$V3)
interpretuj_r(r)
# Przykładowy wynik: r = 0.8732 → silna dodatnia korelacja liniowa
```

**Przykładowy komentarz na kolokwium:**
> „Pomiędzy V1 a V3 obserwujemy **silną dodatnią** korelację liniową (r = 0,87). Wraz ze wzrostem wartości V1 rosną również wartości V3."

---

## 5. Macierz korelacji

### Co to jest

Tablica wszystkich dwustronnych współczynników korelacji Pearsona dla każdej pary kolumn w data frame. Macierz jest **symetryczna** ($r_{XY} = r_{YX}$), przekątna wynosi 1 (cecha skorelowana z samą sobą).

$$P = \begin{pmatrix} 1 & r_{V1,V2} & r_{V1,V3} \\ r_{V2,V1} & 1 & r_{V2,V3} \\ r_{V3,V1} & r_{V3,V2} & 1 \end{pmatrix}$$

### Dane wejściowe

`cor()` na ramce danych wymaga, żeby **wszystkie kolumny były numeryczne**. Jeśli masz kolumnę z datami lub tekstem — usuń ją przed obliczeniem.

```r
# Wariant 1: ramka ma tylko kolumny numeryczne
macierz_kor <- cor(df)

# Wariant 2: pierwsza kolumna to Date/tekst — pomiń ją
macierz_kor <- cor(df[, -1])             # usuń pierwszą kolumnę
macierz_kor <- cor(df[, c("V1","V2","V3")])  # wybierz konkretne kolumny

# Wariant 3: są NA
macierz_kor <- cor(df[, -1], use = "complete.obs")

# Usuń przekątną (jedynki) — żeby nie zaburzały szukania ekstremum
diag(macierz_kor) <- NA

# Wyświetl z zaokrągleniem
round(macierz_kor, 3)
```

### Znajdowanie ekstremalnych par — gotowy kod

```r
macierz_kor <- cor(df[, -1])   # df[,-1] jeśli pierwsza kolumna to np. Date
diag(macierz_kor) <- NA

# ── A) Najsilniejsza współzależność (najwyższe |r|, bez względu na znak) ──
idx <- arrayInd(which.max(abs(macierz_kor)), .dim = dim(macierz_kor))
cat("Najsilniejsza:", rownames(macierz_kor)[idx[1]], "—", colnames(macierz_kor)[idx[2]],
    "| r =", round(macierz_kor[idx[1], idx[2]], 3), "\n")

# ── B) Najsłabsza współzależność (najniższe |r|) ──
idx <- arrayInd(which.min(abs(macierz_kor)), .dim = dim(macierz_kor))
cat("Najsłabsza:", rownames(macierz_kor)[idx[1]], "—", colnames(macierz_kor)[idx[2]],
    "| r =", round(macierz_kor[idx[1], idx[2]], 3), "\n")

# ── C) Najsilniejsza DODATNIA współzależność ──
m_pos <- macierz_kor
m_pos[m_pos <= 0] <- NA   # wyzeruj ujemne i zera
idx <- arrayInd(which.max(m_pos), .dim = dim(macierz_kor))
cat("Najsilniejsza dodatnia:", rownames(macierz_kor)[idx[1]], "—", colnames(macierz_kor)[idx[2]],
    "| r =", round(macierz_kor[idx[1], idx[2]], 3), "\n")

# ── D) Najsilniejsza UJEMNA współzależność ──
m_neg <- macierz_kor
m_neg[m_neg >= 0] <- NA   # wyzeruj dodatnie i zera
idx <- arrayInd(which.max(abs(m_neg)), .dim = dim(macierz_kor))
cat("Najsilniejsza ujemna:", rownames(macierz_kor)[idx[1]], "—", colnames(macierz_kor)[idx[2]],
    "| r =", round(macierz_kor[idx[1], idx[2]], 3), "\n")

# ── E) Z jaką kolumną najsilniej skorelowana jest kolumna "V1"? ──
wiersz_V1 <- macierz_kor["V1", ]               # wyciągnij cały wiersz dla V1
idx_max <- which.max(abs(wiersz_V1))
cat("V1 najsilniej z:", names(idx_max),
    "| r =", round(wiersz_V1[idx_max], 3), "\n")

# ── F) Z jaką kolumną najsłabiej skorelowana jest kolumna "V1"? ──
idx_min <- which.min(abs(wiersz_V1))
cat("V1 najsłabiej z:", names(idx_min),
    "| r =", round(wiersz_V1[idx_min], 3), "\n")
```

> ⚠️ `arrayInd(which.max(...), .dim = dim(macierz))` — zwraca numer wiersza i kolumny. Żeby dostać **nazwy**, użyj `rownames()[idx[1]]` i `colnames()[idx[2]]`.

---

## 6. Współczynnik korelacji Spearmana

### Co mierzy

Siłę i kierunek **monotonicznej** zależności (niekoniecznie liniowej) — oparty na **rangach**, nie wartościach bezwzględnych.

### Wzór matematyczny

$$r_S = \frac{Cov(R_X, R_Y)}{S_{R_X} \cdot S_{R_Y}}$$

gdzie $R_X$, $R_Y$ to rangi wartości kolumn X i Y.

**Uproszczony wzór (gdy nie ma rang wiązanych):**

$$r_S = 1 - \frac{6 \sum_{i=1}^{n} d_i^2}{n(n^2-1)}, \quad d_i = \text{ranga}(x_i) - \text{ranga}(y_i)$$

### Nadawanie rang — jak działa `rank()`

Rangi: posortuj wartości rosnąco, nadaj pozycje 1, 2, 3, ... Przy powtórzeniach → **ranga wiązana** = średnia arytmetyczna pozycji.

```r
# Przykład działania rank() na kolumnie data frame:
df$V1
rank(df$V1)
# Wartość   10  30  20  30  10
# Ranga    1.5 4.5 3.0 4.5 1.5   ← 10 jest na pozycji 1 i 2, więc ranga = 1.5

# Podgląd rang dla konkretnej kolumny:
data.frame(
  V1       = df$V1,
  ranga_V1 = rank(df$V1)
)
```

### Wzór Spearmana jako kod R — krok po kroku na kolumnach

```r
# Metoda 1: wzór ręczny przez rangi (= Pearson obliczony na rangach)
x_rangi <- rank(df$V1)
y_rangi <- rank(df$V3)
r_spearman_reczny <- cor(x_rangi, y_rangi)  # Pearson na rangach = Spearman
r_spearman_reczny

# Metoda 2: uproszczony wzór z d²
d  <- rank(df$V1) - rank(df$V3)      # różnice rang
n  <- nrow(df)
r_spearman_wzor <- 1 - (6 * sum(d^2)) / (n * (n^2 - 1))
r_spearman_wzor

# Metoda 3: wbudowana funkcja — najszybsza i poprawna nawet z rangami wiązanymi
r_spearman <- cor(x = df$V1, y = df$V3, method = "spearman")
r_spearman
```

> ⚠️ Metoda 3 (`method = "spearman"`) jest zawsze poprawna. Metoda 2 (wzór z $d^2$) **jest dokładna tylko bez rang wiązanych** — jeśli w danych są powtórzenia, wyniki się różnią. Na kolokwium zawsze używaj metody 3.

### Interpretacja

Skala siły i kierunku — **identyczna jak dla Pearsona** (ta sama tabela).

```r
# Spearman dla pary kolumn:
cor(x = df$V1, y = df$V3, method = "spearman")

# Spearman dla całej macierzy:
cor(df[, -1], method = "spearman")
```

### Kiedy Spearman, kiedy Pearson?

| Sytuacja | Użyj |
|---|---|
| Dane normalne, bez outlierów, zależność liniowa | **Pearson** |
| Są outliery w kolumnach | **Spearman** (odporny na outliery) |
| Zależność monotonics ale nieliniowa (widać na wykresie) | **Spearman** |
| Dane na skali porządkowej (np. oceny, rankingi) | **Spearman** |

---

## 7. Wpływ obserwacji odstających

### Dlaczego to ważne

Jedna ekstremalna wartość w kolumnie może całkowicie zmienić wartość r Pearsona, a prawie nie wpłynąć na r Spearmana (bo outlier dostaje tylko skrajną rangę, a nie drastycznie inną wartość).

### Eksperyment 1 — dodanie outlieru do kolumn

```r
# Sytuacja: masz df z kolumnami Zloto i Srebro
# Dodaj do kolumn ekstremalną wartość

zloto_out <- c(close_df$Zloto, 8000)  # 8000 — dużo wyżej niż normalne ceny
srebro_out <- c(close_df$Srebro, 20)  # 20 — dużo niżej niż normalne ceny

# Porównaj wykresy
par(mfrow = c(1, 2))
plot(close_df$Zloto, close_df$Srebro, main = "Bez outlieru")
plot(zloto_out, srebro_out, main = "Z outlierem")
par(mfrow = c(1, 1))

# Pearson vs Spearman — bez outlieru
cor(close_df$Zloto, close_df$Srebro)
cor(close_df$Zloto, close_df$Srebro, method = "spearman")

# Pearson vs Spearman — z outlierem
cor(zloto_out, srebro_out)
cor(zloto_out, srebro_out, method = "spearman")

# Spodziewany efekt: Pearson bardzo się zmienia, Spearman prawie nie
```

### Eksperyment 2 — losowe dane + outlier

```r
set.seed(1234)

# Dwie niezależne kolumny z rozkładu normalnego (r ≈ 0)
x <- rnorm(50, 1000, 10)
y <- rnorm(50, 1000, 10)

# Umieść w data frame — tak jak na kolokwium
df_exp <- data.frame(x = x, y = y)

plot(df_exp$x, df_exp$y, main = "Bez outlieru")
cat("Pearson bez outlieru: ", round(cor(df_exp$x, df_exp$y), 3), "\n")
cat("Spearman bez outlieru:", round(cor(df_exp$x, df_exp$y, method = "spearman"), 3), "\n")

# Dodaj outlier — dodaj wiersz do data frame
df_exp_out <- rbind(df_exp, data.frame(x = 100, y = 100))

plot(df_exp_out$x, df_exp_out$y, main = "Z outlierem")
cat("Pearson z outlierem: ", round(cor(df_exp_out$x, df_exp_out$y), 3), "\n")
cat("Spearman z outlierem:", round(cor(df_exp_out$x, df_exp_out$y, method = "spearman"), 3), "\n")
```

**Wniosek do komentarza:**
> „Po dodaniu obserwacji odstającej współczynnik Pearsona zmienił się znacząco (z ≈ 0,05 do ≈ 0,72), natomiast współczynnik Spearmana pozostał stabilny (z ≈ 0,04 do ≈ 0,07), co potwierdza jego odporność na obserwacje odstające."

---

## 8. Korelacja cząstkowa

### Co mierzy

Współzależność między dwiema kolumnami **po wyeliminowaniu wpływu** pozostałych kolumn (zmiennych kontrolnych). Odpowiada na pytanie: „Czy X i Y są skorelowane, gdyby wszystkie jednostki miały **tę samą wartość** Z?"

### Wzór matematyczny (3 zmienne: X, Y, Z)

$$r_{XY \cdot Z} = \frac{r_{XY} - r_{XZ} \cdot r_{YZ}}{\sqrt{(1 - r_{XZ}^2)(1 - r_{YZ}^2)}}$$

gdzie:
- $r_{XY}$ = korelacja całkowita między X i Y (bez kontroli)
- $r_{XZ}$ = korelacja X z kontrolowaną zmienną Z
- $r_{YZ}$ = korelacja Y z kontrolowaną zmienną Z

### Wzór jako kod R — krok po kroku na kolumnach

```r
# Mamy df z kolumnami: Pojemnosc_silnika, Zuzycie_paliwa, Waga
# Pytanie: czy pojemność i zużycie są skorelowane, jeśli wyeliminujemy wpływ wagi?

r_XY <- cor(df$Pojemnosc_silnika, df$Zuzycie_paliwa)   # korelacja całkowita X~Y
r_XZ <- cor(df$Pojemnosc_silnika, df$Waga)              # X~Z (kontrolowana)
r_YZ <- cor(df$Zuzycie_paliwa,    df$Waga)              # Y~Z (kontrolowana)

# Wzór ręczny:
r_czastkowa_reczna <- (r_XY - r_XZ * r_YZ) / sqrt((1 - r_XZ^2) * (1 - r_YZ^2))
cat("Korelacja cząstkowa (ręcznie):", round(r_czastkowa_reczna, 4), "\n")

# Funkcja pcor() — robi to samo dla wielu par naraz:
# ⚠️ WAŻNE: kolejność kolumn w select() ma znaczenie!
#   - kolumna 1: zmienna X
#   - kolumna 2: zmienna Y
#   - kolumna 3+: zmienne kontrolowane (eliminowane)
wynik_pcor <- pcor(x = select(df, Pojemnosc_silnika, Zuzycie_paliwa, Waga))

# Wyciągnij cząstkowy r między kolumną 1 a kolumną 2:
r_czastkowa <- wynik_pcor$estimate[1, 2]
p_wartosc   <- wynik_pcor$p.value[1, 2]

cat("Korelacja całkowita X~Y:  ", round(r_XY, 4), "\n")
cat("Korelacja cząstkowa X~Y|Z:", round(r_czastkowa, 4), "\n")
cat("p-wartość:                 ", round(p_wartosc, 4), "\n")
```

### Co zwraca `pcor()`

```r
wynik_pcor$estimate    # macierz cząstkowych współczynników korelacji
                       #   estimate[1,2] = r_cząstkowe między kolumną 1 a 2
wynik_pcor$p.value     # p-wartości dla każdej pary
wynik_pcor$statistic   # statystyki testowe
wynik_pcor$n           # liczba obserwacji
wynik_pcor$gp          # liczba kontrolowanych zmiennych
wynik_pcor$method      # domyślnie "pearson"
```

### Interpretacja i komentarz

```r
# Pełne porównanie:
cat("=== Korelacja cząstkowa — podsumowanie ===\n")
cat("Korelacja całkowita (bez kontroli): ", round(r_XY, 3), "—", interpretuj_r(r_XY), "\n")
cat("Korelacja cząstkowa (kontrolując Z):", round(r_czastkowa, 3), "—", interpretuj_r(r_czastkowa), "\n")

if (abs(r_XY) - abs(r_czastkowa) > 0.2) {
  cat("→ Zmienna Z wyjaśnia dużą część związku X~Y (korelacja pozorna lub zakłócona)\n")
} else {
  cat("→ Zmienna Z nie wpływa istotnie na związek X~Y\n")
}
```

**Klasyczny przykład do komentarza:**
> „Korelacja całkowita między pojemnością silnika a zużyciem paliwa wynosi r = 0,82 (silna dodatnia). Jednak po uwzględnieniu wagi samochodu (kontrolując wpływ masy), cząstkowy współczynnik korelacji wynosi zaledwie r = 0,08 (bardzo słaba). Oznacza to, że waga samochodu jest zmienną zakłócającą — zarówno duże silniki, jak i wysokie zużycie paliwa są w dużej mierze konsekwencją ciężkiego auta."

---

## 9. Korelacja wieloraka

### Co mierzy

Współzależność **jednej wybranej kolumny** ze **wszystkimi pozostałymi kolumnami łącznie**. Im wyższy wynik, tym lepiej pozostałe cechy wyjaśniają tę jedną kolumnę.

### Wzór matematyczny (X zależna, Y i Z objaśniające)

$$r_{X \cdot YZ} = \sqrt{\frac{r_{XY}^2 + r_{XZ}^2 - 2\, r_{XY}\, r_{XZ}\, r_{YZ}}{1 - r_{YZ}^2}}$$

Ogólny wzór przez macierz:

$$r_{X \cdot YZ} = \sqrt{1 - \frac{|P|}{P_{11}}}$$

gdzie $|P|$ to wyznacznik macierzy korelacji, $P_{11}$ to dopełnienie algebraiczne elementu (1,1).

### Wzór jako kod R — krok po kroku na kolumnach

```r
# Mamy df z kolumnami V1, V2, V3
# Pytanie: jak silnie V1 jest powiązana łącznie z V2 i V3?

# Krok 1: oblicz potrzebne korelacje dwustronne
r_12 <- cor(df$V1, df$V2)   # r_XY
r_13 <- cor(df$V1, df$V3)   # r_XZ
r_23 <- cor(df$V2, df$V3)   # r_YZ

# Krok 2: podstaw do wzoru
r_wieloraka <- sqrt(
  (r_12^2 + r_13^2 - 2 * r_12 * r_13 * r_23) /
  (1 - r_23^2)
)
cat("Korelacja wieloraka V1~(V2,V3):", round(r_wieloraka, 4), "\n")

# ─── Alternatywnie: przez macierz korelacji i wyznacznik ───

P <- cor(df[, c("V1", "V2", "V3")])  # macierz korelacji 3x3
# Wyznacznik macierzy:
det_P  <- det(P)
# Dopełnienie algebraiczne P11 = wyznacznik podmacierzy bez wiersza 1 i kolumny 1:
P11    <- det(P[-1, -1])
r_wieloraka_macierz <- sqrt(1 - det_P / P11)
cat("Korelacja wieloraka (przez macierz):", round(r_wieloraka_macierz, 4), "\n")
```

### Interpretacja

```r
# Własności:
# - Wartości z przedziału [0, 1] — zawsze nieujemna!
# - Im bliżej 1, tym silniejsza łączna współzależność
# - Użyj tej samej skali słownej co dla Pearsona (ale bez kierunku — zawsze >= 0)

cat("r_wieloraka =", round(r_wieloraka, 3), "\n")
# Przykład: 0.91 → bardzo silna współzależność V1 z V2 i V3 łącznie
```

**Komentarz do wyników:**
> „Współczynnik korelacji wielorakiej V1 z V2 i V3 wynosi r = 0,91, co oznacza **bardzo silną** łączną współzależność. Cechy V2 i V3 razem w dużym stopniu wyjaśniają zmienność cechy V1."

---

## 10. Pobieranie danych finansowych (quantmod)

### Pobieranie pojedynczego instrumentu

```r
# Dane wejściowe:
# - symbol: ticker Yahoo Finance jako string (np. "GC=F" = złoto)
# - src:    źródło danych — zawsze "yahoo"
# - from / to: daty w formacie "RRRR-MM-DD"
# - auto.assign = FALSE: WAŻNE — zwróć obiekt, nie przypisuj globalnie

dane <- getSymbols("GC=F",
                    src         = "yahoo",
                    from        = "2025-11-01",
                    to          = "2025-12-31",
                    auto.assign = FALSE)

# Konwersja do zwykłej ramki danych:
dane_df           <- as.data.frame(dane)
dane_df$Date      <- rownames(dane_df)
rownames(dane_df) <- NULL
head(dane_df)
# Kolumny: GC=F.Open, GC=F.High, GC=F.Low, GC=F.Close, GC=F.Volume, GC=F.Adjusted, Date
```

### Pobieranie wielu instrumentów naraz

```r
symbole <- c("GC=F", "SI=F", "CL=F", "BZ=F", "NG=F",
             "HG=F", "PL=F", "PA=F", "ZW=F", "ZC=F",
             "ZS=F", "CC=F", "KC=F", "SB=F")

nazwy <- c("Zloto", "Srebro", "Ropa_WTI", "Ropa_Brent", "Gaz_ziemny",
           "Miedz", "Platyna", "Pallad", "Pszenica", "Kukurydza",
           "Soja", "Kakao", "Kawa", "Cukier")

# Dla każdego symbolu: pobierz dane i wyciągnij tylko cenę zamknięcia
close_list <- lapply(symbole, \(x) {
  dane_x           <- getSymbols(x, src = "yahoo",
                                  from = "2025-11-01", to = "2025-12-31",
                                  auto.assign = FALSE)
  dane_x           <- as.data.frame(dane_x)
  dane_x$Date      <- rownames(dane_x)
  rownames(dane_x) <- NULL
  # Zostaw tylko: Date i kolumnę z ceną zamknięcia (kończy się na ".Close")
  dane_x[, c("Date", paste0(x, ".Close"))]
})

# Połącz wszystkie ramki po kolumnie Date
close_df              <- Reduce(merge, close_list)
colnames(close_df)[-1] <- nazwy  # nadaj czytelne nazwy (bez Date)

head(close_df)
# Wynik: ramka danych gdzie każda kolumna to ceny zamknięcia jednego surowca
```

### Tabela tickerów surowców

| Symbol | Surowiec | Symbol | Surowiec |
|---|---|---|---|
| `GC=F` | Złoto | `ZW=F` | Pszenica |
| `SI=F` | Srebro | `ZC=F` | Kukurydza |
| `CL=F` | Ropa WTI | `ZS=F` | Soja |
| `BZ=F` | Ropa Brent | `CC=F` | Kakao |
| `NG=F` | Gaz ziemny | `KC=F` | Kawa |
| `HG=F` | Miedź | `SB=F` | Cukier |
| `PL=F` | Platyna | `PA=F` | Pallad |

---

## 11. Wzory — ściągawka z kodem

### Pearson

$$r_{XY} = \frac{Cov(X,Y)}{S_X \cdot S_Y}$$

```r
cor(df$KolumnaX, df$KolumnaY)                        # domyślnie Pearson
cor(df$KolumnaX, df$KolumnaY, use = "complete.obs")  # jeśli są NA
```

### Spearman

$$r_S = 1 - \frac{6\sum d_i^2}{n(n^2-1)}, \quad d_i = \text{ranga}(x_i) - \text{ranga}(y_i)$$

```r
cor(df$KolumnaX, df$KolumnaY, method = "spearman")   # wbudowana funkcja
cor(rank(df$KolumnaX), rank(df$KolumnaY))             # równoważne (Pearson na rangach)
```

### Korelacja cząstkowa (eliminacja zmiennej Z)

$$r_{XY \cdot Z} = \frac{r_{XY} - r_{XZ} \cdot r_{YZ}}{\sqrt{(1 - r_{XZ}^2)(1 - r_{YZ}^2)}}$$

```r
# X = kolumna 1, Y = kolumna 2, Z = kolumna kontrolna (3+)
wynik <- pcor(select(df, KolumnaX, KolumnaY, KolumnaZ))
wynik$estimate[1, 2]   # cząstkowy r między X a Y
wynik$p.value[1, 2]    # p-wartość
```

### Korelacja wieloraka (X wyjaśniana przez Y i Z)

$$r_{X \cdot YZ} = \sqrt{\frac{r_{XY}^2 + r_{XZ}^2 - 2\,r_{XY}\,r_{XZ}\,r_{YZ}}{1 - r_{YZ}^2}}$$

```r
r_12 <- cor(df$V1, df$V2)
r_13 <- cor(df$V1, df$V3)
r_23 <- cor(df$V2, df$V3)
sqrt((r_12^2 + r_13^2 - 2*r_12*r_13*r_23) / (1 - r_23^2))
```

---

## 12. Typowe zadania kolokwialne — gotowy kod

### „Wyznacz współczynnik korelacji między kolumną V1 a V3"

```r
# Pearson (domyślny):
cor(df$V1, df$V3)

# Spearman:
cor(df$V1, df$V3, method = "spearman")

# Oba naraz z interpretacją:
r_p <- cor(df$V1, df$V3)
r_s <- cor(df$V1, df$V3, method = "spearman")
cat("Pearson: ", round(r_p, 4), "\n")
cat("Spearman:", round(r_s, 4), "\n")
```

### „Zbadaj współzależność między wszystkimi parami cech ilościowych"

```r
# Usuń kolumny nienumeryczne
df_num <- df |> select_if(is.numeric)

# Macierz korelacji
macierz_kor <- cor(df_num)
diag(macierz_kor) <- NA
round(macierz_kor, 3)
```

### „Znajdź parę cech o najsilniejszej/najsłabszej korelacji"

```r
macierz_kor <- cor(df |> select_if(is.numeric))
diag(macierz_kor) <- NA

znajdz_pare <- function(macierz, typ = "max_abs") {
  idx <- switch(typ,
    "max_abs" = arrayInd(which.max(abs(macierz)),           .dim = dim(macierz)),
    "min_abs" = arrayInd(which.min(abs(macierz)),           .dim = dim(macierz)),
    "max_pos" = { m <- macierz; m[m<=0] <- NA
                  arrayInd(which.max(m),          .dim = dim(macierz)) },
    "max_neg" = { m <- macierz; m[m>=0] <- NA
                  arrayInd(which.max(abs(m)),     .dim = dim(macierz)) }
  )
  cat(rownames(macierz)[idx[1]], "—", colnames(macierz)[idx[2]],
      "| r =", round(macierz[idx[1], idx[2]], 3), "\n")
}

znajdz_pare(macierz_kor, "max_abs")  # A: najsilniejsza
znajdz_pare(macierz_kor, "min_abs")  # B: najsłabsza
znajdz_pare(macierz_kor, "max_pos")  # C: najsilniejsza dodatnia
znajdz_pare(macierz_kor, "max_neg")  # D: najsilniejsza ujemna
```

### „Oceń wpływ zmiennej zakłócającej — korelacja cząstkowa"

```r
# Scenariusz: czy V1 i V2 są skorelowane, jeśli wyeliminujemy wpływ V3?

r_calkowita <- cor(df$V1, df$V2)

wynik       <- pcor(select(df, V1, V2, V3))
r_czastkowa <- wynik$estimate[1, 2]
p_wartosc   <- wynik$p.value[1, 2]

cat("Korelacja całkowita V1~V2:     ", round(r_calkowita, 3), "\n")
cat("Korelacja cząstkowa V1~V2 | V3:", round(r_czastkowa, 3), "\n")
cat("p-wartość:                      ", round(p_wartosc, 4), "\n")

# Jeśli p-wartość < 0.05: korelacja cząstkowa jest istotna statystycznie
```

### „Porównaj Pearson i Spearman przed i po dodaniu outlieru"

```r
# Bez outlieru
r_p_bez <- cor(df$V1, df$V3)
r_s_bez <- cor(df$V1, df$V3, method = "spearman")

# Dodaj wiersz-outlier do data frame
df_out <- rbind(df, setNames(data.frame(t(rep(NA, ncol(df)))), colnames(df)))
df_out[nrow(df_out), "V1"] <- 99999   # ekstremalna wartość
df_out[nrow(df_out), "V3"] <- 1

r_p_out <- cor(df_out$V1, df_out$V3, use = "complete.obs")
r_s_out <- cor(df_out$V1, df_out$V3, method = "spearman", use = "complete.obs")

cat("Pearson  bez / z outlierem:", round(r_p_bez,3), "/", round(r_p_out,3), "\n")
cat("Spearman bez / z outlierem:", round(r_s_bez,3), "/", round(r_s_out,3), "\n")
```

---

## Szybkie drzewko decyzyjne

```
Mam zadanie z korelacją...
│
├─ Dwie kolumny numeryczne?
│   ├─ Zawsze: plot(df$X, df$Y)  ← najpierw wykres!
│   ├─ Zależność liniowa, bez outlierów → cor(df$X, df$Y)              [Pearson]
│   ├─ Outliery / nieliniowe / porządkowe → cor(df$X, df$Y, method="spearman")
│   └─ Chcę usunąć wpływ trzeciej kolumny Z → pcor(select(df, X, Y, Z))
│
├─ Wiele kolumn naraz?
│   ├─ Macierz: cor(df[,-1])  +  diag() <- NA
│   └─ Szukanie par: which.max/min(abs()) + arrayInd() → nazwy kolumn
│
└─ Korelacja wieloraka (X z Y i Z łącznie)?
    └─ Wzór ręczny ze składników cor() lub det() macierzy
```

---

*Notatki przygotowane na podstawie wykładu i kodu laboratoryjnego — Analiza Współzależności Cech Ilościowych*
