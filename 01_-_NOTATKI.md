# Analiza Struktury — Notatki na Kolokwium
Notatki własne
2026-06-16

------------------------------------------------------------------------

# Wprowadzenie — Analiza Struktury

**Analiza struktury** polega na opisie zbiorowości statystycznej ze
względu na badane cechy zmienne. Dla cech ilościowych składa się z
czterech filarów:

| Filar | Co bada |
|----|----|
| **Tendencja centralna** | Wartości typowe, wokół których skupiają się dane |
| **Rozproszenie (dyspersja)** | Stopień zróżnicowania wartości wokół wartości typowych |
| **Asymetria (skośność)** | Kierunek zróżnicowania wartości względem średniej |
| **Koncentracja** | Stopień skupienia wokół wartości centralnych / intensywność wartości skrajnych |

## Miary klasyczne vs. pozycyjne

|  | Miary klasyczne | Miary pozycyjne |
|----|----|----|
| **Podstawa obliczeń** | Wszystkie obserwacje | Wybrane obserwacje (np. kwartyle) |
| **Kiedy stosować** | Rozkłady jednorodne, jednomodalne, symetryczne lub umiarkowanie asymetryczne | Rozkłady niejednorodne, silnie asymetryczne, wielomodalne |
| **Wrażliwość na outliers** | ⚠️ Tak — bardzo wrażliwe | ✅ Nie — odporne |

> **Miary klasyczne stosujemy** gdy rozkład jest: (1) jednomodalny, (2)
> symetryczny lub umiarkowanie asymetryczny, (3) o umiarkowanym
> zróżnicowaniu i koncentracji.

------------------------------------------------------------------------

# Typy szeregów statystycznych

Zanim przejdziemy do wzorów — musisz wiedzieć, z jakim typem danych
pracujesz, bo wzory różnią się zależnie od szeregu.

## Szereg szczegółowy

Surowe, posortowane rosnąco obserwacje: $x_1, x_2, \ldots, x_n$.

## Szereg strukturalny punktowy

Tabela częstości dla cechy skokowej: poziomy $x_i$ z liczebnościami
$n_i$, gdzie $\sum n_i = n$.

## Szereg strukturalny przedziałowy

Tabela z przedziałami klasowymi $[x_{i-1}, x_i)$ i liczebnościami
$n_i$.  
Środek $i$-tego przedziału: $x'_i = \frac{x_i + x_{i-1}}{2}$

> **W praktyce (plik .xlsx):** Masz kolumnę z wartościami → szereg
> szczegółowy. Masz kolumnę z przedziałami i kolumnę z liczebnościami →
> szereg przedziałowy. R obsługuje obie sytuacje.

------------------------------------------------------------------------

# Tendencja Centralna

## Średnia arytmetyczna $\bar{x}$

### Pojęcie

Suma wszystkich wartości podzielona przez liczebność próby. Wskazuje
“centrum ciężkości” rozkładu. **Podstawowa miara klasyczna tendencji
centralnej.**

### Wzór

**Szereg szczegółowy:**

```math
\bar{x} = \frac{\sum_{i=1}^{n} x_i}{n}
```

**Szereg strukturalny punktowy:**
```math
\bar{x} = \frac{\sum_{i=1}^{k} x_i n_i}{n}
```

**Szereg strukturalny przedziałowy:**
```math
\bar{x} = \frac{\sum_{i=1}^{k} x'_i n_i}{n}
```

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx` (np. kolumna `wynagrodzenie`, `wiek`,
`stopa_zwrotu`).

### Kod R

``` r
# ── ŚREDNIA ARYTMETYCZNA ──────────────────────────────────────────────

# Wczytaj dane z pliku xlsx (odkomentuj i podaj ścieżkę):
# library(readxl)
# df <- read_excel("dane.xlsx")
# x <- df$nazwa_kolumny

# Przykładowe dane (szereg szczegółowy)
x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

# Obliczenie średniej
srednia <- mean(x)

# Dla szeregu przedziałowego (ręcznie):
# Definiujesz środki przedziałów x_prime i liczebności n_i
x_prime <- c(5, 15, 25, 35, 45)   # środki przedziałów
n_i     <- c(10, 20, 40, 20, 10)  # liczebności
n       <- sum(n_i)

srednia_przedzialowa <- sum(x_prime * n_i) / n
```

### Co ten wynik oznacza

Średnia $\bar{x}$ to typowa wartość cechy w zbiorowości. **Interpretacja
bezpośrednia** — wyrażona w jednostkach badanej cechy (np. PLN, lata,
%).

> ⚠️ Jeśli rozkład jest silnie asymetryczny lub zawiera wartości
> odstające, średnia jest **myląca** — lepiej użyć mediany.

------------------------------------------------------------------------

## Mediana $Me$ (Kwartyl 2.)

### Pojęcie

Wartość środkowa — **50% obserwacji leży poniżej, 50% powyżej** mediany.
Miara pozycyjna, odporna na wartości odstające.

### Wzór

**Szereg szczegółowy / punktowy:**
```math
Me = \begin{cases} x_{\lfloor n/2 \rfloor + 1} & \text{gdy } n \text{ nieparzyste} \\ \dfrac{x_{n/2} + x_{n/2+1}}{2} & \text{gdy } n \text{ parzyste} \end{cases}
```

**Szereg strukturalny przedziałowy (interpolacja):**
```math
Me = x_{Me} + \frac{\dfrac{1}{2}n - \sum_{i=1}^{k_{Me}-1} n_i}{n_{Me}} \cdot c_{Me}
```

gdzie:

- $x_{Me}$ — początek przedziału medianego
- $\sum_{i=1}^{k_{Me}-1} n_i$ — suma liczebności przed przedziałem
  medianym
- $n_{Me}$ — liczebność przedziału medianego
- $c_{Me}$ — szerokość przedziału medianego
- Przedział mediany: pierwszy, dla którego dystrybuanta empiryczna
  $\geq 0{,}5$

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx`. Dla szeregu przedziałowego: kolumna
z przedziałami + kolumna z liczebnościami.

### Kod R

``` r
# ── MEDIANA ───────────────────────────────────────────────────────────

x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

mediana <- median(x)
cat("Mediana:", mediana, "\n")

# Dla szeregu przedziałowego — ręczna interpolacja:
granice <- c(0, 10, 20, 30, 40, 50)   # granice przedziałów
n_i     <- c(5, 15, 30, 25, 15, 10)   # liczebności
n       <- sum(n_i)

# Dystrybuanta empiryczna (skumulowane częstości)
kum <- cumsum(n_i) / n
cat("Dystrybuanta:", round(kum, 3), "\n")

# Przedział mediany: pierwszy, dla którego dystrybuanta >= 0.5
k_Me <- which(kum >= 0.5)[1]

x_Me  <- granice[k_Me]               # początek przedziału
sum_przed <- ifelse(k_Me > 1, sum(n_i[1:(k_Me-1)]), 0)
n_Me  <- n_i[k_Me]
c_Me  <- granice[k_Me + 1] - granice[k_Me]

Me_przedzialowa <- x_Me + (0.5 * n - sum_przed) / n_Me * c_Me
cat("Mediana (szereg przedziałowy):", Me_przedzialowa, "\n")
```

### Co ten wynik oznacza

Co najmniej 50% jednostek ma wartość $\leq Me$. **Mediana jest
odporniejsza na wartości ekstremalne niż średnia.**

> Jeśli $\bar{x} \approx Me$ → rozkład symetryczny.  
> Jeśli $\bar{x} > Me$ → rozkład prawostronnie asymetryczny.  
> Jeśli $\bar{x} < Me$ → rozkład lewostronnie asymetryczny.

------------------------------------------------------------------------

## Dominanta $D$ (Moda)

### Pojęcie

Wartość najczęściej występująca w zbiorze. Dla szeregu przedziałowego —
środek najliczniejszego przedziału, wyznaczany interpolacją.

### Wzór

**Szereg strukturalny przedziałowy:**
```math
D = x_D + \frac{n_D - n_{D-1}}{2n_D - n_{D-1} - n_{D+1}} \cdot c_D
```

gdzie:

- $x_D$ — początek najliczniejszego przedziału
- $n_D$ — liczebność najliczniejszego przedziału
- $n_{D-1}$, $n_{D+1}$ — liczebności sąsiednich przedziałów
- $c_D$ — szerokość przedziału dominanty

> ⚠️ **Nie wyznaczamy dominanty**, gdy najliczniejszy jest pierwszy lub
> ostatni przedział klasowy.

### Typ danych wejściowych

Dla szeregu punktowego: kolumna kategoryczna/numeryczna. Dla
przedziałowego: tabela z liczebnościami.

### Kod R

``` r
# ── DOMINANTA ─────────────────────────────────────────────────────────

# Szereg szczegółowy — dominanta jako najczęstsza wartość
x <- c(12, 15, 14, 10, 13, 14, 11, 16, 14, 17)

dominanta_szcz <- as.numeric(names(sort(table(x), decreasing = TRUE)[1]))
cat("Dominanta (szereg szczegółowy):", dominanta_szcz, "\n")

# Szereg przedziałowy — interpolacja
granice <- c(0, 10, 20, 30, 40, 50)
n_i     <- c(5, 15, 30, 25, 15, 10)

# Znajdź najliczniejszy przedział (indeks w n_i)
k_D <- which.max(n_i)

# Sprawdź, czy dominanta nie jest w pierwszym lub ostatnim przedziale
if (k_D == 1 || k_D == length(n_i)) {
  cat("⚠️ Dominanta nie jest wyznaczana — najliczniejszy jest skrajny przedział.\n")
} else {
  x_D   <- granice[k_D]
  n_D   <- n_i[k_D]
  n_Dm1 <- n_i[k_D - 1]
  n_Dp1 <- n_i[k_D + 1]
  c_D   <- granice[k_D + 1] - granice[k_D]
  
  D <- x_D + (n_D - n_Dm1) / (2 * n_D - n_Dm1 - n_Dp1) * c_D
  cat("Dominanta (szereg przedziałowy):", D, "\n")
}
```

### Co ten wynik oznacza

Dominanta to wartość/przedział o największej częstości. Wskazuje, gdzie
“skupia się masa” rozkładu.

> **Związek z asymetrią:**  
> Rozkład lewostronny: $\bar{x} < Me < D$  
> Rozkład symetryczny: $\bar{x} = Me = D$  
> Rozkład prawostronny: $D < Me < \bar{x}$

------------------------------------------------------------------------

# Rozproszenie (Zmienność)

## Wariancja $S^2$ / $s^2$

### Pojęcie

Przeciętne kwadratowe odchylenie wartości cechy od średniej. **Wyrażona
w kwadracie jednostek** → nie interpretujemy bezpośrednio — służy
głównie jako krok pośredni do odchylenia standardowego.

### Wzór

**Populacja — szereg szczegółowy:**
```math
S^2 = \frac{\sum_{i=1}^{n}(x_i - \bar{x})^2}{n}
```

**Populacja — szereg punktowy:**
```math
S^2 = \frac{\sum_{i=1}^{k}(x_i - \bar{x})^2 n_i}{n}
```

**Populacja — szereg przedziałowy:**
```math
S^2 = \frac{\sum_{i=1}^{k}(x'_i - \bar{x})^2 n_i}{n}
```

**Próba** (korekta Bessela — dzielnik $n-1$ zamiast $n$):
```math
s^2 = \frac{\sum_{i=1}^{n}(x_i - \bar{x})^2}{n-1}
```

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx`.

### Kod R

``` r
# ── WARIANCJA ─────────────────────────────────────────────────────────

x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

# Wariancja próby (domyślna w R) — dzielnik (n-1)
wariancja_proba <- var(x)

# Wariancja populacji — dzielnik n
n <- length(x)
wariancja_pop <- var(x) * (n - 1) / n
```

### Co ten wynik oznacza

Wariancja jest **nieinterpretowalną miarą pośrednią** — jej jednostka to
kwadrat jednostki cechy (np. PLN²). Używamy jej do obliczenia odchylenia
standardowego.

> ✅ Im większa wariancja → tym bardziej “rozsiane” dane.  
> ⚠️ Nie porównuj wariancji między zmiennymi o różnych jednostkach.

------------------------------------------------------------------------

## Odchylenie standardowe $S$ / $s$

### Pojęcie

Pierwiastek kwadratowy z wariancji. **Podstawowa miara zróżnicowania** —
wyrażona w tych samych jednostkach co badana cecha. Interpretacja:
przeciętne odchylenie wartości od średniej.

### Wzór

```math
S = \sqrt{S^2} \qquad \text{(populacja)}
```
```math
s = \sqrt{s^2} \qquad \text{(próba)}
```

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx`.

### Kod R

``` r
# ── ODCHYLENIE STANDARDOWE ────────────────────────────────────────────

x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

# Odchylenie standardowe próby (n-1) — domyślna w R
odch_std_proba <- sd(x)

# Odchylenie standardowe populacji (n)
n <- length(x)
odch_std_pop <- sd(x) * sqrt((n - 1) / n)
```

### Co ten wynik oznacza

Odchylenie standardowe $S$ mówi, o ile **przeciętnie** wartości cechy
odchylają się od średniej $\bar{x}$.

> **Przykład interpretacji:**  
> Jeśli $\bar{x} = 50$ PLN i $S = 10$ PLN → wartości odchylają się
> przeciętnie o 10 PLN od średniej.
>
> **Reguła 3σ** (dla rozkładu normalnego):  
> - $(\bar{x} \pm \sigma)$ obejmuje ≈ 68,2% obserwacji  
> - $(\bar{x} \pm 2\sigma)$ obejmuje ≈ 95,4% obserwacji  
> - $(\bar{x} \pm 3\sigma)$ obejmuje ≈ 99,7% obserwacji

------------------------------------------------------------------------

## Współczynnik zmienności $V$

### Pojęcie

Względna (bezjednostkowa) miara rozproszenia. Pozwala **porównywać
zmienność** między różnymi zbiorami danych lub zmiennymi o różnych
jednostkach.

### Wzór

```math
V = \frac{S}{\bar{x}} \cdot 100\%
```

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx` (potrzebujemy $S$ i $\bar{x}$).

### Kod R

``` r
# ── WSPÓŁCZYNNIK ZMIENNOŚCI ────────────────────────────────────────────

x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

srednia <- mean(x)
odch_std <- sd(x)

V <- (odch_std / srednia) * 100

# Interpretacja automatyczna
if (V <= 20) {
  cat("Interpretacja: SŁABE zróżnicowanie (V ≤ 20%)\n")
} else if (V <= 40) {
  cat("Interpretacja: UMIARKOWANE zróżnicowanie (20% < V ≤ 40%)\n")
} else if (V <= 70) {
  cat("Interpretacja: SILNE zróżnicowanie (40% < V ≤ 70%)\n")
} else {
  cat("Interpretacja: BARDZO SILNE zróżnicowanie (V > 70%)\n")
}
```

### Co ten wynik oznacza

| Przedział $V$           | Ocena zróżnicowania                  |
|-------------------------|--------------------------------------|
| $V \in (0, 20\%]$       | 🟢 Słabe — dane jednorodne           |
| $V \in (20\%, 40\%]$    | 🟡 Umiarkowane                       |
| $V \in (40\%, 70\%]$    | 🟠 Silne                             |
| $V \in (70\%, +\infty)$ | 🔴 Bardzo silne — dane niejednorodne |

> ⚠️ Jeśli $V > 40\%$, rozważ stosowanie miar pozycyjnych zamiast
> klasycznych.

------------------------------------------------------------------------

## Odchylenie ćwiartkowe $Q$

### Pojęcie

Pozycyjna miara zmienności oparta na kwartalach. Interpretacja:
przeciętne odchylenie wartości od mediany (dla środkowych 50%
obserwacji).

### Wzór

```math
Q = \frac{Q_3 - Q_1}{2}
```

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx`.

### Kod R

``` r
# ── ODCHYLENIE ĆWIARTKOWE ─────────────────────────────────────────────

x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

# Kwartyle algorytmem n-1 (type=2 w R odpowiada zbliżonemu podejściu)
# Wykładowca używa algorytmu n-1 → type=2
Q1 <- quantile(x, 0.25, type = 2)
Q3 <- quantile(x, 0.75, type = 2)

Q <- (Q3 - Q1) / 2
cat("Q1 =", Q1, "| Q3 =", Q3, "\n")
cat("Odchylenie ćwiartkowe Q =", Q, "\n")
```

### Co ten wynik oznacza

Odchylenie ćwiartkowe $Q$ to przeciętne odchylenie środkowych 50%
obserwacji od mediany. Im mniejsze $Q$, tym bardziej skupione są dane
centralne.

------------------------------------------------------------------------

## Pozycyjny współczynnik zmienności $V_Q$

### Pojęcie

Względna (bezjednostkowa) miara zmienności oparta na miarach
pozycyjnych. Analogia do $V$, ale odporna na wartości odstające.

### Wzór

```math
V_Q = \frac{Q}{Me} \cdot 100\%
```

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx`.

### Kod R

``` r
# ── POZYCYJNY WSPÓŁCZYNNIK ZMIENNOŚCI ─────────────────────────────────

x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

Q1 <- quantile(x, 0.25, type = 2)
Q3 <- quantile(x, 0.75, type = 2)
Me <- median(x)
Q  <- (Q3 - Q1) / 2

VQ <- (Q / Me) * 100

# Interpretacja: identyczna jak dla V (progi 20%, 40%, 70%)
```

### Co ten wynik oznacza

Interpretacja identyczna jak dla $V$ (progi 20%, 40%, 70%). Używamy, gdy
stosujemy miary pozycyjne (rozkład asymetryczny, outliery).

------------------------------------------------------------------------

# Asymetria (Skośność)

## Współczynnik asymetrii $\alpha_3$

### Pojęcie

Trzeci moment centralny standaryzowany. Mierzy kierunek i siłę asymetrii
rozkładu względem średniej.

### Wzór

**Populacja — szereg szczegółowy:**
```math
\alpha_3 = \frac{\dfrac{1}{n}\sum_{i=1}^{n}(x_i - \bar{x})^3}{S^3}
```

**Populacja — szereg punktowy:**
```math
\alpha_3 = \frac{\dfrac{1}{n}\sum_{i=1}^{k}(x_i - \bar{x})^3 n_i}{S^3}
```

**Populacja — szereg przedziałowy:**
```math
\alpha_3 = \frac{\dfrac{1}{n}\sum_{i=1}^{k}(x'_i - \bar{x})^3 n_i}{S^3}
```

**Próba** (z korektą):
```math
\alpha_3 = \frac{n}{(n-1)(n-2)} \cdot \frac{\sum_{i=1}^{n}(x_i - \bar{x})^3}{S^3}
```

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx`.

### Kod R

``` r
# ── WSPÓŁCZYNNIK ASYMETRII / SKOŚNOŚCI ────────────────────────────────
# Wymagany pakiet: e1071 (lub moments)
# install.packages("e1071")
library(e1071)

x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

# Skośność próby (typ 2 — zgodny z Excelem: SKOŚNOŚĆ)
alpha3_proba <- skewness(x, type = 2)
cat("Współczynnik asymetrii (próba):", round(alpha3_proba, 4), "\n")

# Skośność populacji (typ 1)
alpha3_pop <- skewness(x, type = 1)
cat("Współczynnik asymetrii (populacja):", round(alpha3_pop, 4), "\n")

# Excel: SKOŚNOŚĆ → typ 2 (próba), SKOŚNOŚĆ.P → typ 1 (populacja)

# Automatyczna interpretacja
interpretuj_asym <- function(a3) {
  kierunek <- if (a3 < 0) "LEWOSTRONNIE asymetryczny" else
              if (a3 > 0) "PRAWOSTRONNIE asymetryczny" else "SYMETRYCZNY"
  sila <- if (abs(a3) < 0.25) "słaba" else
          if (abs(a3) < 0.5)  "umiarkowana" else "silna"
  cat("Kierunek:", kierunek, "\n")
  if (a3 != 0) cat("Siła asymetrii:", sila, "\n")
}

interpretuj_asym(alpha3_proba)
```

### Co ten wynik oznacza

**Kierunek asymetrii:**

| Wartość $\alpha_3$ | Kierunek               | Relacja miar centralnych |
|--------------------|------------------------|--------------------------|
| $\alpha_3 < 0$     | Lewostronny (ujemny)   | $\bar{x} < Me < D$       |
| $\alpha_3 = 0$     | Symetryczny            | $\bar{x} = Me = D$       |
| $\alpha_3 > 0$     | Prawostronny (dodatni) | $D < Me < \bar{x}$       |

**Siła asymetrii:**

| $|\alpha_3|$         | Ocena          |
|----------------------|----------------|
| $(0;\; 0{,}25)$      | 🟢 Słaba       |
| $[0{,}25;\; 0{,}5)$  | 🟡 Umiarkowana |
| $[0{,}5;\; +\infty)$ | 🔴 Silna       |

> **Lewostronny:** większość jednostek ma wartość **większą** niż
> średnia (ogon w lewo).  
> **Prawostronny:** większość jednostek ma wartość **mniejszą** niż
> średnia (ogon w prawo).

------------------------------------------------------------------------

## Pozycyjny współczynnik asymetrii $A_Q$

### Pojęcie

Miara asymetrii oparta na kwartalach — odporna na wartości odstające.
Dotyczy asymetrii w środkowych 50% obserwacji.

### Wzór

```math
A_Q = \frac{Q_1 + Q_3 - 2 \cdot Me}{2Q}
```

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx`.

### Kod R

``` r
# ── POZYCYJNY WSPÓŁCZYNNIK ASYMETRII ──────────────────────────────────

x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

Q1 <- quantile(x, 0.25, type = 2)
Q3 <- quantile(x, 0.75, type = 2)
Me <- median(x)
Q  <- (Q3 - Q1) / 2

AQ <- (Q1 + Q3 - 2 * Me) / (2 * Q)
cat("Pozycyjny współczynnik asymetrii AQ =", round(AQ, 4), "\n")

# Interpretacja: identyczna jak dla alpha3
# AQ < 0 → lewostronny, AQ = 0 → symetryczny, AQ > 0 → prawostronny
# |AQ| < 0.25 → słaba, [0.25, 0.5) → umiarkowana, >= 0.5 → silna
```

### Co ten wynik oznacza

Interpretacja taka sama jak $\alpha_3$, ale dotyczy **środkowych 50%
obserwacji** (między Q1 a Q3). Używamy przy rozkładach asymetrycznych
lub z outliersami.

------------------------------------------------------------------------

## Klasyczno-pozycyjny współczynnik asymetrii $As$

### Pojęcie

Miara asymetrii łącząca elementy klasyczne ($\bar{x}$, $S$) i pozycyjne
($D$). Wskazuje, jak daleko średnia leży od dominanty w stosunku do
odchylenia standardowego.

### Wzór

```math
As = \frac{\bar{x} - D}{S}
```

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx`.

### Kod R

``` r
# ── KLASYCZNO-POZYCYJNY WSPÓŁCZYNNIK ASYMETRII ────────────────────────

x <- c(12, 15, 14, 10, 13, 14, 11, 16, 14, 17)

# Wyznacz składniki
xbar <- mean(x)
S    <- sd(x)     # odchylenie standardowe próby
D    <- as.numeric(names(sort(table(x), decreasing = TRUE)[1]))  # dominanta

As <- (xbar - D) / S
cat("x̄ =", xbar, "| D =", D, "| S =", round(S, 4), "\n")
cat("Klasyczno-pozycyjny współczynnik asymetrii As =", round(As, 4), "\n")

# Interpretacja taka sama jak alpha3
```

### Co ten wynik oznacza

Interpretacja identyczna z $\alpha_3$. $As > 0$ → rozkład prawostronny;
$As < 0$ → lewostronny.

------------------------------------------------------------------------

# Koncentracja

## Kurtoza $K$ i Współczynnik ekscesu $K_0$

### Pojęcie

**Kurtoza** (czwarty moment centralny standaryzowany) mierzy
“wysmukłość” lub “spłaszczenie” rozkładu względem rozkładu normalnego.
Wskazuje, jak silna jest koncentracja wokół wartości centralnych i jak
intensywne są wartości skrajne (ogony).

**Współczynnik ekscesu** $K_0$ to kurtoza pomniejszona o 3 (wartość
kurtozy dla rozkładu normalnego) — ułatwia porównanie z normalnym.

### Wzór

**Populacja:**
```math
K = \alpha_4 = \frac{\dfrac{1}{n}\sum_{i=1}^{n}(x_i - \bar{x})^4}{S^4}
```

```math
K_0 = K - 3
```

**Próba** (z korektą):
```math
K = \alpha_4 = \frac{n(n+1)}{(n-1)(n-2)(n-3)} \cdot \frac{\sum_{i=1}^{n}(x_i - \bar{x})^4}{S^4}
```

```math
K_0 = K - \frac{3(n-1)^2}{(n-2)(n-3)}
```

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx`.

### Kod R

``` r
# ── KURTOZA I WSPÓŁCZYNNIK EKSCESU ────────────────────────────────────
library(e1071)

x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

# W R: kurtosis() z e1071 zwraca WSPÓŁCZYNNIK EKSCESU (K0 = K - 3)
współczynnik_eksces_populacja <- kurtosis(x, type = 1)
współczynnik_eksces_proba <- kurtosis(x, type = 2)

# Kurtoza = eksces + 3
# Czwarty moment
kurtoza <- współczynnik_eksces_populacja + 3

# Interpretacja
if (współczynnik_ekscesu< 0) {
  cat("→ Rozkład PLATOKURTYCZNY (spłaszczony): słabsza koncentracja i słabsze ogony niż normalny\n")
} else if (współczynnik_ekscesu == 0) {
  cat("→ Rozkład MEZOKURTYCZNY: jak rozkład normalny\n")
} else {
  cat("→ Rozkład LEPTOKURTYCZNY (wysmukły): silniejsza koncentracja i grubsze ogony niż normalny\n")
}

# Excel: KURTOZA → K0 (eksces próby)
```

### Co ten wynik oznacza

| Wartość $K$ / $K_0$ | Typ rozkładu | Interpretacja |
|----|----|----|
| $K < 3$ / $K_0 < 0$ | 🔵 **Platokurtyczny** (spłaszczony) | Słabsza koncentracja wokół centrum, łagodniejsze ogony niż normalny |
| $K = 3$ / $K_0 = 0$ | ⚪ **Mezokurtyczny** | Jak rozkład normalny |
| $K > 3$ / $K_0 > 0$ | 🔴 **Leptokurtyczny** (wysmukły) | Silniejsza koncentracja, grubsze ogony — więcej wartości ekstremalnych |

> **Grubsze ogony** (leptokurtyczny) to ważne zjawisko w finansach —
> oznacza więcej ekstremalnych zwrotów niż przewiduje model normalny.

------------------------------------------------------------------------

# Miary Pozycyjne — Kwartyle

## Kwartyle $Q_1$, $Q_2$ ($Me$), $Q_3$

### Pojęcie

Kwartyle dzielą posortowany zbiór danych na cztery równe części (po
25%). Są pozycyjnymi miarami tendencji centralnej i zmienności.

- $Q_1$: co najmniej 25% obserwacji $\leq Q_1$
- $Q_2 = Me$: co najmniej 50% obserwacji $\leq Me$
- $Q_3$: co najmniej 75% obserwacji $\leq Q_3$

### Wzór

**Algorytm $n-1$ (algorytm wykładowcy):**

```math
q_1 = \frac{1}{4}(n-1) + 1, \quad Q_1 = (1-d) \cdot x_k + d \cdot x_{k+1}
```
```math
q_3 = \frac{3}{4}(n-1) + 1, \quad Q_3 = (1-d) \cdot x_k + d \cdot x_{k+1}
```

gdzie $k = \lfloor q \rfloor$, $d = q - k$ (część ułamkowa)

**Szereg strukturalny przedziałowy:**

```math
Q_1 = x_{Q_1} + \frac{\dfrac{1}{4}n - \sum_{i=1}^{k_{Q_1}-1} n_i}{n_{Q_1}} \cdot c_{Q_1}
```

```math
Q_3 = x_{Q_3} + \frac{\dfrac{3}{4}n - \sum_{i=1}^{k_{Q_3}-1} n_i}{n_{Q_3}} \cdot c_{Q_3}
```

Przedział $Q_1$: pierwszy, dla którego dystrybuanta $\geq 0{,}25$  
Przedział $Q_3$: pierwszy, dla którego dystrybuanta $\geq 0{,}75$

### Typ danych wejściowych

Kolumna numeryczna z pliku `.xlsx`. Dla szeregu przedziałowego: kolumna
z przedziałami + kolumna z liczebnościami.

### Kod R

``` r
# ── KWARTYLE ──────────────────────────────────────────────────────────

x <- c(12, 15, 14, 10, 13, 18, 11, 16, 14, 17)

# Algorytm n-1 → type=2 w R (KWARTYL.PRZEDZ.ZAMK w Excelu)
Q1 <- quantile(x, 0.25, type = 2)
Me <- quantile(x, 0.50, type = 2)   # lub median(x)
Q3 <- quantile(x, 0.75, type = 2)

cat("Q1 =", Q1, "\n")
cat("Me = Q2 =", Me, "\n")
cat("Q3 =", Q3, "\n")
cat("IQR (rozstęp ćwiartkowy) =", Q3 - Q1, "\n")

# Dla szeregu przedziałowego — funkcja ogólna:
kwartyl_przedzialowy <- function(p, granice, n_i) {
  n   <- sum(n_i)
  kum <- cumsum(n_i) / n
  k   <- which(kum >= p)[1]
  
  x_k       <- granice[k]
  sum_przed <- ifelse(k > 1, sum(n_i[1:(k-1)]), 0)
  n_k       <- n_i[k]
  c_k       <- granice[k + 1] - granice[k]
  
  Qp <- x_k + (p * n - sum_przed) / n_k * c_k
  return(Qp)
}

# Przykład użycia:
granice <- c(0, 10, 20, 30, 40, 50)
n_i     <- c(5, 15, 30, 25, 15, 10)

cat("\nSzerego przedziałowego:\n")
cat("Q1 =", kwartyl_przedzialowy(0.25, granice, n_i), "\n")
cat("Me =", kwartyl_przedzialowy(0.50, granice, n_i), "\n")
cat("Q3 =", kwartyl_przedzialowy(0.75, granice, n_i), "\n")

# Excel: KWARTYL.PRZEDZ.ZAMK(dane; 1/2/3) → algorytm n-1
```

### Co ten wynik oznacza

Kwartyle dzielą dane na ćwiartki:

\|—25%—\|—25%—\|—25%—\|—25%—\| min Q1 Me Q3 max

**IQR** (rozstęp ćwiartkowy) $= Q_3 - Q_1$ — obejmuje środkowe 50%
obserwacji. Używany do wykrywania wartości odstających (outliery:
$< Q_1 - 1{,}5 \cdot IQR$ lub $> Q_3 + 1{,}5 \cdot IQR$).
