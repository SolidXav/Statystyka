# 📊 Regresja Liniowa – Notatki do Kolokwium

> **Cel pliku:** Pełny tutorial z teorią, wzorami i kodem R. Każde pojęcie ma przykład działający na data frame'ie z kolumnami.

---

## 0. Przygotowanie środowiska

```r
library(readxl)
library(dplyr)

# Wczytanie danych z pliku Excel
df <- read_xlsx("sciezka/do/pliku.xlsx")

# Podgląd danych
head(df)
str(df)

# Filtrowanie wierszy (np. tylko samochody osobowe)
df <- df |> filter(Typ == "osobowy")
```

> **Dane wejściowe:** `read_xlsx()` potrzebuje ścieżki do pliku `.xlsx`.
> `filter()` z pakietu `dplyr` filtruje wiersze na podstawie warunku w kolumnie.

---

## 1. Współzależność liniowa (korelacja)

### 1.1 Wykres rozrzutu

```r
# Schemat ogólny: plot(x = df$KolumnaX, y = df$KolumnaY)
plot(x = df$Moc, y = df$Cena,
     xlab = "Moc (KM)", ylab = "Cena (tys. USD)",
     main = "Zależność ceny od mocy")
```

> **Dane wejściowe:** dwie kolumny numeryczne z data frame'u.
> Wykres pozwala ocenić **czy zależność jest liniowa** (punkty układają się wzdłuż linii prostej) czy nieliniowa (krzywa).

---

### 1.2 Współczynnik korelacji Pearsona

**Wzór:**

$$r_{xy} = \frac{\frac{1}{n}\sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y})}{s_x \cdot s_y}$$

```r
# Schemat ogólny
cor(x = df$KolumnaX, y = df$KolumnaY)

# Przykład
cor(x = df$Moc, y = df$Cena)
```

> **Dane wejściowe:** dwie kolumny numeryczne.
> **Wynik:** liczba z przedziału [-1, 1].

**Interpretacja:**
| Wartość `r` | Interpretacja |
|-------------|---------------|
| blisko **+1** | silna dodatnia współzależność liniowa |
| blisko **-1** | silna ujemna współzależność liniowa |
| blisko **0** | brak współzależności liniowej |
| > 0.7 lub < -0.7 | silna współzależność |
| 0.3–0.7 lub -0.7–(-0.3) | umiarkowana współzależność |
| < 0.3 | słaba lub brak współzależności |

**Przykładowy komentarz:** *„Pomiędzy mocą a ceną obserwujemy silną dodatnią współzależność liniową (r ≈ 0.85)."*

---

### 1.3 Współczynnik korelacji rang Spearmana

Stosowany gdy dane **nie są normalne** lub mają charakter porządkowy.

```r
# Schemat ogólny
cor(x = df$KolumnaX, y = df$KolumnaY, method = "spearman")

# Przykład: korelacja rang między kolumną V1 a V3
cor(x = df$V1, y = df$V3, method = "spearman")
```

> **Dane wejściowe:** dwie kolumny numeryczne (lub porządkowe).
> **Interpretacja** taka sama jak dla Pearsona – oceniamy siłę i kierunek zależności monotonicznej.

---

## 2. Model regresji liniowej

### 2.1 Równanie modelu

$$y_i = a_0 + a_1 x_i + e_i$$

$$\hat{y}_i = a_0 + a_1 x_i$$

gdzie:
- $y_i$ – wartości **empiryczne** (obserwowane) zmiennej objaśnianej (Y, zależnej)
- $x_i$ – wartości empiryczne zmiennej objaśniającej (X, niezależnej)
- $\hat{y}_i$ – wartości **teoretyczne** (dopasowane przez model)
- $e_i = y_i - \hat{y}_i$ – **reszty** (błędy modelu)
- $a_1$ – **współczynnik kierunkowy**
- $a_0$ – **wyraz wolny**

---

### 2.2 Estymacja metodą najmniejszych kwadratów (MNK)

MNK minimalizuje sumę kwadratów reszt: $\min \sum_{i=1}^{n} e_i^2$

**Wzory na parametry:**

$$a_1 = \frac{\frac{1}{n}\sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y})}{s_x^2}$$

$$a_0 = \bar{y} - a_1 \bar{x}$$

```r
# Budowa modelu regresji: lm(Y ~ X, data = df)
# Y = kolumna objaśniana (zależna)
# X = kolumna objaśniająca (niezależna)

model <- lm(Cena ~ Moc, data = df)
model

# Podgląd współczynników
model$coefficients        # oba parametry: a0 (Intercept) i a1
model$coefficients[1]     # wyraz wolny a0
model$coefficients[2]     # współczynnik kierunkowy a1

# Wartości teoretyczne (ŷ)
model$fitted.values

# Reszty (e_i = y_i - ŷ_i)
model$residuals
head(model$residuals)     # kilka pierwszych reszt
```

> **Dane wejściowe:** `lm()` potrzebuje formuły `Y ~ X` oraz data frame'u ze wskazanymi kolumnami.

---

### 2.3 Interpretacja parametrów

#### Współczynnik kierunkowy $a_1$

> „Wraz ze wzrostem **[X]** o **1 [jednostka X]**, **[Y]** zmienia się przeciętnie o **$a_1$ [jednostka Y]**."

```r
# Odczyt wartości a1
a1 <- model$coefficients[2]
cat("Współczynnik kierunkowy a1 =", a1)

# Przykład interpretacji:
# a1 = 0.214 → "Wraz ze wzrostem mocy o 1 KM, cena wzrasta przeciętnie o 0.214 tys. USD"
```

#### Wyraz wolny $a_0$

> „Gdy **[X] = 0**, to **[Y]** wynosi teoretycznie **$a_0$ [jednostka Y]**."
> ⚠️ Ta interpretacja **często nie ma sensu ekonomicznego/technicznego** i w praktyce się jej nie stosuje.

```r
a0 <- model$coefficients[1]
cat("Wyraz wolny a0 =", a0)
```

---

## 3. Ocena dopasowania modelu

```r
# Pełne podsumowanie modelu
model_sum <- summary(model)
model_sum
```

---

### 3.1 Odchylenie standardowe składnika resztowego $s_e$

**Wzór:**

$$s_e = \sqrt{\frac{\sum_{i=1}^{n}(y_i - \hat{y}_i)^2}{n - 2}}$$

```r
# Odczyt z podsumowania modelu
se <- model_sum$sigma
cat("Odchylenie standardowe reszt se =", se)
```

> **Interpretacja:** „Teoretyczne wartości **[Y]** odchylają się od obserwowanych przeciętnie o **$s_e$ [jednostka Y]**."

---

### 3.2 Współczynnik zmienności składnika resztowego $V_e$

**Wzór:**

$$V_e = \frac{s_e}{\bar{y}} \cdot 100\%$$

```r
# Schemat ogólny
se <- model_sum$sigma
srednia_y <- mean(df$KolumnaY)   # <-- zmień na swoją kolumnę Y

Ve <- se / srednia_y
cat("Współczynnik zmienności reszt Ve =", Ve * 100, "%")
```

> **Interpretacja:** „Odchylenia teoretycznych wartości **[Y]** od obserwowanych stanowią przeciętnie **$V_e \cdot 100\%$** przeciętnego poziomu **[Y]**."
>
> 💡 Zasada: im mniejszy $V_e$, tym lepsze dopasowanie modelu.
> Umownie: $V_e < 10\%$ – dobre dopasowanie; $V_e > 30\%$ – słabe dopasowanie.

---

### 3.3 Współczynnik determinacji $R^2$

**Wzór:**

$$R^2 = 1 - \frac{\sum_{i=1}^{n}(y_i - \hat{y}_i)^2}{\sum_{i=1}^{n}(y_i - \bar{y})^2} = r_{xy}^2$$

```r
# Odczyt R² z modelu
R2 <- model_sum$r.squared
cat("R² =", R2)
```

> **Interpretacja:** „Model tłumaczy **$R^2 \cdot 100\%$** zmienności **[Y]**. Pozostałe **$(1-R^2) \cdot 100\%$** nie zostało wyjaśnione przez model."

---

### 3.4 Współczynnik zbieżności (indeterminacji) $\phi^2$

**Wzór:**

$$\phi^2 = 1 - R^2$$

```r
phi2 <- 1 - model_sum$r.squared
cat("Współczynnik zbieżności phi² =", phi2)
```

> **Interpretacja:** Odsetek zmienności **[Y]** niewyjaśniony przez model.
> W praktyce podaje się głównie $R^2$, nie $\phi^2$.

---

### 3.5 Skorygowany $R^2$ (adjusted $R^2$)

**Wzór:**

$$R^2_{adj} = 1 - \frac{(1-R^2)(N-1)}{N-p-1}$$

gdzie $p$ = liczba zmiennych objaśniających.

```r
R2_adj <- model_sum$adj.r.squared
cat("Skorygowany R² =", R2_adj)
```

> Stosowany przy **porównaniu modeli z różną liczbą zmiennych** – karze za dodawanie nieistotnych predyktorów.

---

## 4. Istotność parametrów modelu

### 4.1 Błędy szacunku parametrów

**Wzory:**

$$s_{a_1} = \frac{s_e}{\sqrt{n} \cdot s_x}$$

$$s_{a_0} = \frac{s_e \cdot \sqrt{\sum_{i=1}^{n} x_i^2}}{n \cdot s_x}$$

```r
# Błędy szacunku obu parametrów (a0 i a1)
bledy <- model_sum$coefficients[, 2]
bledy

# Błąd a0 (wyrazu wolnego)
blad_a0 <- model_sum$coefficients[1, 2]

# Błąd a1 (współczynnika kierunkowego)
blad_a1 <- model_sum$coefficients[2, 2]
```

---

### 4.2 Względne błędy szacunku $V_{a_0}$, $V_{a_1}$

**Wzory:**

$$V_{a_1} = \frac{s_{a_1}}{|a_1|}$$

$$V_{a_0} = \frac{s_{a_0}}{|a_0|}$$

```r
# Względne błędy szacunku (proporcja błędu do wartości parametru)
wzgledne_bledy <- model_sum$coefficients[, 2] / abs(model_sum$coefficients[, 1])
wzgledne_bledy

# Osobno dla a1:
Va1 <- model_sum$coefficients[2, 2] / abs(model_sum$coefficients[2, 1])
cat("Względny błąd a1 (Va1) =", Va1)

# Osobno dla a0:
Va0 <- model_sum$coefficients[1, 2] / abs(model_sum$coefficients[1, 1])
cat("Względny błąd a0 (Va0) =", Va0)
```

> **Interpretacja:**
> - $V_{a_1} < 0.5$ → współczynnik kierunkowy **jest istotny statystycznie** → zmienna X **istotnie wpływa** na Y
> - $V_{a_1} \geq 0.5$ → współczynnik kierunkowy **nie jest istotny** → brak istotnego wpływu X na Y
> - Interpretację $V_{a_0}$ w praktyce pomija się

---

### 4.3 Wartości p (p-value) – preferowana metoda oceny istotności

```r
# Wartości p dla obu parametrów
pvalues <- model_sum$coefficients[, 4]
pvalues

# p-value dla a1 (współczynnik kierunkowy)
p_a1 <- model_sum$coefficients[2, 4]
cat("p-value dla a1 =", p_a1)
```

> **Interpretacja (poziom istotności α = 0.05):**
> - `p-value < 0.05` → parametr **jest istotny statystycznie** ✅
> - `p-value ≥ 0.05` → parametr **nie jest istotny statystycznie** ❌
>
> Przykład: *„p-value = 0.0001 < 0.05, zatem moc istotnie wpływa na cenę samochodu."*

---

## 5. Prognoza / Interpolacja

### 5.1 Wartość prognozowana

**Wzór:**

$$\hat{y}^*(x^*) = a_0 + a_1 \cdot x^*$$

```r
# Schemat ogólny: podstawiamy dowolną wartość x* do modelu
x_new <- 500   # <-- wpisz interesującą Cię wartość X

# Metoda 1: ręczne obliczenie
prognoza <- model$coefficients[1] + model$coefficients[2] * x_new
cat("Prognozowana wartość Y dla X =", x_new, ":", prognoza)

# Metoda 2: funkcja predict() – wygodniejsza
nowe_dane <- data.frame(Moc = x_new)   # <-- nazwa kolumny musi być taka sama jak w modelu
predict(model, newdata = nowe_dane)
```

> **Dane wejściowe:** `predict()` potrzebuje modelu oraz data frame'u z tą samą nazwą kolumny X co przy budowie modelu.
> **Wynik:** prognozowana wartość zmiennej Y dla podanej wartości X.

---

### 5.2 Błąd prognozy

**Wzór:**

$$s(\hat{y}^*) = s_e \cdot \sqrt{1 + \frac{1}{n} + \frac{(x^* - \bar{x})^2}{s_x^2 \cdot n}}$$

```r
# Schemat ogólny – ręczne obliczenie błędu prognozy
se   <- model_sum$sigma           # odchylenie standardowe reszt
n    <- nrow(df)                  # liczba obserwacji
sr_x <- mean(df$KolumnaX)        # <-- zmień na kolumnę X
s2_x <- var(df$KolumnaX)         # <-- zmień na kolumnę X
x_new <- 500                      # <-- wartość, dla której liczymy prognozę

blad_prognozy <- se * sqrt(1 + 1/n + (x_new - sr_x)^2 / (s2_x * (n - 1)))
cat("Błąd prognozy =", blad_prognozy)

# Prognoza z przedziałem (metoda automatyczna)
predict(model, newdata = data.frame(Moc = x_new), interval = "prediction")
# zwraca: fit (prognoza), lwr (dolna granica), upr (górna granica)
```

> **Interpretacja:** „Cena samochodu o mocy 500 KM wyniosłaby teoretycznie **$\hat{y}^*$ ± błąd** tys. USD."
>
> Np.: *„Cena samochodu o mocy 500 KM wyniosłaby teoretycznie 95.005 (± 9.087) tys. USD."*

---

## 6. Pełny przykład – od A do Z na data frame'ie

```r
library(readxl)
library(dplyr)

# ── 1. Wczytanie i filtrowanie danych ──────────────────────────────────────────
df <- read_xlsx("samochody.xlsx") |>
  filter(Typ == "osobowy")

# ── 2. Analiza współzależności ─────────────────────────────────────────────────
plot(x = df$Moc, y = df$Cena, xlab = "Moc (KM)", ylab = "Cena (tys. USD)")
r <- cor(x = df$Moc, y = df$Cena)
cat("Współczynnik korelacji Pearsona r =", r, "\n")

# ── 3. Budowa modelu regresji ──────────────────────────────────────────────────
model <- lm(Cena ~ Moc, data = df)
cat("\nParametry modelu:\n")
print(model$coefficients)

# ── 4. Podsumowanie i miary dopasowania ───────────────────────────────────────
model_sum <- summary(model)

se  <- model_sum$sigma
Ve  <- se / mean(df$Cena)
R2  <- model_sum$r.squared

cat("\nOcena dopasowania:\n")
cat("  se (odchylenie reszt)           =", se, "\n")
cat("  Ve (wsp. zmienności reszt)      =", round(Ve * 100, 2), "%\n")
cat("  R² (wsp. determinacji)          =", round(R2, 4), "\n")
cat("  phi² (wsp. zbieżności)          =", round(1 - R2, 4), "\n")

# ── 5. Istotność parametrów ────────────────────────────────────────────────────
Va1 <- model_sum$coefficients[2, 2] / abs(model_sum$coefficients[2, 1])
p_a1 <- model_sum$coefficients[2, 4]

cat("\nIstotność współczynnika kierunkowego:\n")
cat("  Względny błąd Va1 =", round(Va1, 4), "\n")
cat("  p-value a1        =", p_a1, "\n")

# ── 6. Prognoza ────────────────────────────────────────────────────────────────
x_new <- 500

prognoza <- model$coefficients[1] + model$coefficients[2] * x_new

n    <- nrow(df)
sr_x <- mean(df$Moc)
s2_x <- var(df$Moc)
blad <- se * sqrt(1 + 1/n + (x_new - sr_x)^2 / (s2_x * (n - 1)))

cat("\nPrognoza dla X =", x_new, ":\n")
cat("  Wartość prognozowana =", round(prognoza, 3), "\n")
cat("  Błąd prognozy        =", round(blad, 3), "\n")
cat("  Przedział: [", round(prognoza - blad, 3), ";", round(prognoza + blad, 3), "]\n")
```

---

## 7. Ściągawka – funkcje i kolumny

| Co chcę policzyć | Kod R | Skąd pochodzi |
|---|---|---|
| Korelacja Pearsona | `cor(df$X, df$Y)` | base R |
| Korelacja Spearmana (rangi) | `cor(df$X, df$Y, method = "spearman")` | base R |
| Model regresji | `lm(Y ~ X, data = df)` | base R |
| Współczynniki $a_0$, $a_1$ | `model$coefficients` | model |
| Reszty $e_i$ | `model$residuals` | model |
| Wartości teoretyczne $\hat{y}_i$ | `model$fitted.values` | model |
| $s_e$ (odch. reszt) | `summary(model)$sigma` | summary |
| $V_e$ (wsp. zmienności reszt) | `summary(model)$sigma / mean(df$Y)` | summary + df |
| $R^2$ | `summary(model)$r.squared` | summary |
| $\phi^2$ | `1 - summary(model)$r.squared` | summary |
| $R^2_{adj}$ | `summary(model)$adj.r.squared` | summary |
| Błędy szacunku $s_{a_0}$, $s_{a_1}$ | `summary(model)$coefficients[, 2]` | summary |
| p-value $a_0$, $a_1$ | `summary(model)$coefficients[, 4]` | summary |
| Prognoza dla $x^*$ | `model$coef[1] + model$coef[2] * x_new` | model |
| Prognoza z przedziałem | `predict(model, data.frame(X = x_new), interval = "prediction")` | predict |

---

## 8. Słowniczek pojęć

| Pojęcie | Znaczenie |
|---|---|
| **Zmienna objaśniana (Y)** | zmienna, którą chcemy przewidzieć / wyjaśnić |
| **Zmienna objaśniająca (X)** | zmienna, która wyjaśnia/przewiduje Y |
| **Reszta $e_i$** | różnica między wartością obserwowaną a teoretyczną: $e_i = y_i - \hat{y}_i$ |
| **MNK** | Metoda Najmniejszych Kwadratów – minimalizuje $\sum e_i^2$ |
| **$R^2$** | ile % zmienności Y model wyjaśnia (0–1, im bliżej 1, tym lepiej) |
| **$s_e$** | przeciętne odchylenie prognoz od rzeczywistości (w jednostkach Y) |
| **$V_e$** | $s_e$ wyrażone jako % średniej Y (im mniej, tym lepiej) |
| **Interpolacja** | prognoza dla $x^*$ leżącego **wewnątrz** zakresu danych |
| **Ekstrapolacja** | prognoza dla $x^*$ leżącego **poza** zakresem danych (wyższy błąd!) |
| **p-value < 0.05** | parametr jest **istotny statystycznie** |
| **$V_{a_1} < 0.5$** | współczynnik kierunkowy jest istotny statystycznie |
