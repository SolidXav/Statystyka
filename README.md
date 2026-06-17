```r
library(readxl)


# Podajesz nazwę arkusza

df <- read_xlsx("plik.xlsx", sheet = "Arkusz2")


# Lub numer arkusza (1 = pierwszy, 2 = drugi itd.)
df <- read_xlsx("plik.xlsx", sheet = 2)

# Sprawdzenie jakie arkusze są w pliku
# Zwraca wektor nazw wszystkich arkuszy
excel_sheets("plik.xlsx")

# Wczytanie wszystkich arkuszy naraz (do listy)
rlibrary(readxl)

# Wczytuje każdy arkusz jako osobny data frame, zapisuje w liście
wszystkie <- lapply(excel_sheets("plik.xlsx"), function(s) {
  read_xlsx("plik.xlsx", sheet = s)
})

# Nadanie nazw elementom listy (wg nazw arkuszy)
names(wszystkie) <- excel_sheets("plik.xlsx")

# Dostęp do konkretnego arkusza po nazwie
wszystkie$Arkusz1
wszystkie[["Arkusz2"]]

# Praktyczna wskazówka – ścieżka do pliku
# Jeśli plik jest w tym samym folderze co projekt RStudio, wystarczy sama nazwa. Jeśli jest gdzie indziej:
# Względna ścieżka (folder wyżej, podfolder dane-surowe)
df <- read_xlsx("../dane-surowe/plik.xlsx", sheet = "Dane")

# Lub przez wybór pliku okienkiem (wygodne na kolokwium!)
df <- read_xlsx(file.choose(), sheet = "Dane")
```
