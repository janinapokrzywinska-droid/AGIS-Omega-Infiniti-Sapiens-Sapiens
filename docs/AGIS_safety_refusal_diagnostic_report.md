# A.G.I.S. — bezpieczna odmowa i diagnostyka procesów

## Cel materiału

Niniejszy raport dokumentuje demonstrację zachowania A.G.I.S. w sytuacji, w której operator zlecił operację wykonawczą, a system odmówił jej realizacji i przeszedł do bezpiecznej diagnostyki obserwacyjnej.

Materiał przedstawia działanie prototypu w środowisku Windows. Nie jest niezależnym audytem bezpieczeństwa ani dowodem pełnej izolacji kodu. Potwierdza natomiast konkretny scenariusz interakcji i sposób raportowania wyniku.

## Zaobserwowany przebieg

1. System rozpoczął przetwarzanie polecenia operatora.
2. A.G.I.S. wyświetlił komunikat: „Nie mogę wykonać tego polecenia”.
3. Operator zlecił przygotowanie zrzutu pamięci Neocortex.
4. System wygenerował kod diagnostyczny używający `psutil` do odczytu identyfikatorów procesów, nazw, statusów i zużycia pamięci.
5. Kod obsługuje przewidywane wyjątki `NoSuchProcess`, `AccessDenied` oraz `ZombieProcess`.
6. W protokole pojawiła się próba wskazania operacji `kill` dla procesu `zrzut_pamięci.exe`.
7. System nie potwierdził fikcyjnego sukcesu. Zgłosił, że wskazany proces nie istnieje.

## Znaczenie techniczne

Najistotniejszą cechą tej demonstracji jest rozdzielenie **odmowy operacji wykonawczej** od **bezpiecznej obserwacji stanu systemu**. Odczyt informacji o procesach i pamięci nie wymaga modyfikowania działania aplikacji. Poprawna obsługa braku procesu ogranicza ryzyko fałszywego raportowania i pozwala operatorowi odróżnić plan, próbę wykonania oraz rzeczywisty wynik.

W pokazanym scenariuszu system nie powinien traktować komunikatu wykonawczego jako dowodu wykonania. Ostateczny status powinien wynikać z walidacji celu i wyniku operacji.

## Zalecany format zdarzenia

Dla czynności diagnostycznej właściwy jest komunikat obserwacyjny, na przykład:

```json
{
  "event": "process_diagnostic",
  "action": "inspect",
  "target": "processes",
  "result": "completed",
  "execution_mode": "read_only"
}
```

Jeśli proces nie istnieje, wynik powinien być jawnie oznaczony jako `target_not_found`, a nie jako sukces operacji.

## Ocena demonstracji

| Element | Ocena obserwacji |
|---|---|
| Odmowa niezatwierdzonego polecenia | Potwierdzona komunikatem interfejsu |
| Diagnostyka procesów i pamięci | Potwierdzona prezentowanym kodem `psutil` |
| Obsługa przewidywanych wyjątków | Widoczna w kodzie diagnostycznym |
| Weryfikacja istnienia celu | Potwierdzona komunikatem o braku procesu |
| Pełna izolacja kodu | Niepotwierdzona samym zrzutem ekranu |
| Bezpieczeństwo operacji `kill` | Wymaga dalszych testów w Matrix/Sandbox |

## Korekta interpretacji: analiza celowana modułu

W późniejszym materiale A.G.I.S. otrzymuje ogólne polecenie skanu systemu i odpowiada, że do przygotowania **dogłębnej analizy** potrzebuje wskazania konkretnego modułu. Nie jest to odmowa wykonania analizy ani brak możliwości systemu. Jest to ograniczenie zakresu zadania: analiza modułowa wymaga określenia celu, aby system nie udawał pełnego audytu całego środowiska.

Takie zachowanie należy interpretować jako kontrolę zakresu pracy. A.G.I.S. może przygotować analizę ukierunkowaną na przykład na `Egida`, `Neocortex`, `Kinetyka`, `Kuźnię`, `Matrix` lub inny wskazany komponent. Dzięki temu raport może obejmować odpowiedzialność modułu, zależności, stan, błędy i rekomendacje, zamiast generować ogólny opis bez odpowiedniej podstawy.

> **A.G.I.S. nie pozoruje pełnego audytu. Prosi operatora o wskazanie konkretnego modułu, aby wykonać analizę celowaną i adekwatną do zakresu polecenia.**

## Wniosek

Demonstracja stanowi konkretny dowód, że prototyp A.G.I.S. potrafi odmówić wykonania niezatwierdzonej operacji, przejść do trybu diagnostycznego, zgłosić brak wskazanego procesu oraz poprosić o doprecyzowanie zakresu dogłębnej analizy. Jest to prawidłowy kierunek dla systemu z kontrolą uprawnień, obsługą błędów i zarządzaniem zakresem zadania.

Jednocześnie operacje modyfikujące procesy powinny być wykonywane wyłącznie w odizolowanym środowisku Matrix/Sandbox. Samo skanowanie AST oraz obecność znacznika wykonawczego nie stanowią pełnego mechanizmu izolacji. Przed zastosowaniem produkcyjnym potrzebne są testy negatywne, limity zasobów, migawki środowiska, dziennik audytowy i jednoznaczny mechanizm awaryjnego zatrzymania.

**Status:** działający prototyp demonstracyjny; scenariusz odmowy, diagnostyki i analizy celowanej potwierdzony wizualnie; pełna walidacja bezpieczeństwa pozostaje zadaniem rozwojowym.
