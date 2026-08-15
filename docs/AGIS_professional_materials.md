# AGIS — lokalny system agentowy do monitorowania i automatyzacji

## Opis projektu

AGIS to projekt lokalnego systemu agentowego dla środowiska Windows. Jego zadaniem jest łączenie obserwacji środowiska, analizy danych, planowania działań oraz kontrolowanej automatyzacji. System ma działać przede wszystkim jako narzędzie wspierające operatora, a nie jako niezależny podmiot podejmujący nieograniczone decyzje.

Projekt należy traktować jako **prototyp badawczo-inżynieryjny**. Zakres autonomii powinien być zwiększany stopniowo, po potwierdzeniu skuteczności testami, audytem i pomiarem ryzyka.

## Główne cele

AGIS ma wspierać cztery obszary:

1. **Monitorowanie** — obserwacja wybranych źródeł danych, stanu systemu i interfejsu użytkownika.
2. **Analizę** — porządkowanie danych, wykrywanie zdarzeń i przygotowywanie kontekstu dla modelu językowego.
3. **Planowanie** — tworzenie planów działań z określeniem warunków wstępnych, ryzyka i oczekiwanego wyniku.
4. **Kontrolowaną automatyzację** — wykonywanie wyłącznie zatwierdzonych operacji w ograniczonym środowisku.

## Architektura funkcjonalna

| Komponent | Odpowiedzialność | Status i ograniczenia |
|---|---|---|
| `perception.py` | Pobieranie obrazu, dźwięku i danych z wybranych źródeł | Wyniki percepcji muszą zawierać poziom pewności oraz znacznik czasu. |
| `automation.py` | Wykonywanie zatwierdzonych operacji na komputerze | Wyłącznie lista dozwolonych aplikacji i operacji niskiego ryzyka. |
| `simulation.py` | Testowanie planu bez wpływu na środowisko produkcyjne | Symulacja nie zastępuje walidacji w środowisku kontrolowanym. |
| `orchestrator.py` | Koordynowanie obserwacji, planowania, kontroli i wykonania | Każda decyzja musi mieć identyfikator, status i możliwość anulowania. |
| `security.py` | Egzekwowanie polityk, izolacja, kontrola uprawnień i audyt | Analiza AST jest tylko jednym z filtrów, nie pełną piaskownicą. |
| `web_ingestion.py` | Pobieranie i normalizacja danych ze źródeł internetowych | Dostęp sieciowy wyłącznie do zatwierdzonych domen i usług. |
| `memory.py` | Przechowywanie i odtwarzanie wybranych informacji | Wymagane zasady retencji, usuwania i ochrony danych. |
| `monitoring.py` | Metryki, logi, alerty i stan usług | Rejestrowanie awarii, opóźnień, decyzji i wykonanych operacji. |
| `operator_ui.html` | Panel operatora | Ma prezentować stan, ryzyko, dowody i wymagane potwierdzenia. |

## Przepływ działania

```text
Źródła danych
    ↓
Percepcja i normalizacja
    ↓
Kontekst oraz pamięć
    ↓
Plan działania
    ↓
Ocena polityk bezpieczeństwa
    ↓
Symulacja lub dry-run
    ↓
Autoryzacja operatora, jeśli wymagana
    ↓
Ograniczone wykonanie
    ↓
Weryfikacja wyniku i pełny audyt
```

## Zasady bezpieczeństwa

AGIS powinien działać zgodnie z zasadą **minimalnych uprawnień**. Moduł nie może uzyskiwać dostępu do plików, sieci, procesów ani urządzeń, jeżeli nie jest to niezbędne do konkretnego zadania.

Operacje o podwyższonym ryzyku — usuwanie danych, zmiana konfiguracji, wysyłanie wiadomości, dostęp do danych wrażliwych, działania finansowe oraz instalacja oprogramowania — wymagają jednoznacznego potwierdzenia operatora.

Każda operacja powinna mieć: opis celu, listę kroków, zakres uprawnień, przewidywany skutek, limit czasu, limit prób, warunek przerwania i zapis wyniku. W razie niepewności system powinien się zatrzymać i poprosić o decyzję, a nie kontynuować na podstawie zgadywania.

## Samorozbudowa narzędzi

Generowanie kodu przez model może służyć do przygotowywania narzędzi pomocniczych, ale nie powinno automatycznie rozszerzać uprawnień systemu. Proponowany cykl wdrożenia narzędzia jest następujący:

```text
Wygenerowanie → testy → analiza bezpieczeństwa → izolowane uruchomienie
→ przegląd operatora → wdrożenie wersji roboczej → monitoring → ewentualny rollback
```

Każde narzędzie musi być wersjonowane i posiadać opis wymaganych uprawnień. Przejście testu funkcjonalnego nie jest równoznaczne z uznaniem kodu za bezpieczny.

## Ograniczenia techniczne

Dokładność systemu zależy od jakości modeli, danych wejściowych, dostępnych zasobów sprzętowych oraz stabilności środowiska Windows. Rozpoznawanie elementów interfejsu na podstawie obrazu może być zawodne, a zmiana układu aplikacji może spowodować wykonanie niewłaściwej operacji.

Pamięć wektorowa może wspierać odtwarzanie kontekstu, ale nie zapewnia sama w sobie poprawności pamięci ani pełnej ciągłości procesu po awarii. Mechanizm watchdog może wykrywać i restartować usługi, lecz samonaprawa wymaga dodatkowych procedur diagnostycznych, testów i bezpiecznego wycofania zmian.

## Plan realizacji wersji MVP

| Etap | Zakres | Kryterium zakończenia |
|---|---|---|
| 1 | Definicja przypadków użycia i kontraktów API | Każdy moduł ma jasno opisane wejścia, wyjścia i błędy. |
| 2 | Monitorowanie i analiza bez automatycznego wykonania | System poprawnie rejestruje zdarzenia i przygotowuje propozycje działań. |
| 3 | Kontrolowana automatyzacja niskiego ryzyka | Operacje są ograniczone do zatwierdzonych aplikacji i scenariuszy. |
| 4 | Symulacja oraz testy awarii | System zatrzymuje się bezpiecznie po błędzie i nie traci dziennika audytowego. |
| 5 | Polityki bezpieczeństwa i izolacja | Dostęp do plików, sieci i procesów jest jawnie ograniczony. |
| 6 | Monitoring operacyjny | Dostępne są metryki, alerty, logi i procedury wycofania. |
| 7 | Testy integracyjne i bezpieczeństwa | Wyniki testów są powtarzalne i udokumentowane. |
| 8 | Stopniowe zwiększanie autonomii | Każdy nowy poziom autonomii ma osobne kryteria akceptacji. |

## Wersja do filmu — 27 sekund

**0–4 s — tytuł**

> AGIS
> Lokalny system agentowy do monitorowania i kontrolowanej automatyzacji

**4–9 s — problem i zakres**

> Jedna warstwa koordynuje dane, analizę, planowanie i wykonanie.
> Domyślnie system działa w trybie nadzorowanym.

**9–15 s — moduły**

> Percepcja środowiska
> Analiza danych i pamięć kontekstowa
> Planowanie działań
> Kontrola uprawnień i audyt
> Symulacja przed wykonaniem

**15–21 s — bezpieczeństwo**

> Minimalne uprawnienia
> Lista dozwolonych operacji
> Potwierdzenie działań wysokiego ryzyka
> Pełny dziennik zdarzeń

**21–27 s — zakończenie**

> Cel projektu: powtarzalna, mierzalna i bezpieczna automatyzacja.
> AGIS — prototyp badawczo-inżynieryjny.

## Zalecenia dotyczące materiału wideo

Należy usunąć widok czatu mobilnego, baner promocyjny, nieczytelne przewijanie, zniekształcony szum oraz nazwy sugerujące fikcyjny system wszechmocny. Zamiast tego film powinien używać spokojnej planszy 9:16, prostego diagramu przepływu, krótkich podpisów i neutralnej palety: grafit, biel, granat oraz jeden kolor akcentowy.

Należy zastąpić określenia „Omega Infiniti”, „Rada Trzech”, „Rój”, „Kuźnia”, „Egida”, „Architekt”, „Matrix” i „samonaprawiająca się twierdza” nazwami opisowymi: **koordynator**, **moduł bezpieczeństwa**, **generator narzędzi**, **symulator**, **operator** i **mechanizm odzyskiwania po awarii**. Nazwa AGIS może pozostać jako nazwa projektu, ale powinna być opisana jako nazwa robocza prototypu.

Materiał powinien jasno odróżniać funkcje już zaimplementowane, funkcje testowane oraz elementy planowane. To jednoznacznie zwiększy wiarygodność bardziej niż rozbudowane deklaracje o pełnej autonomii.

## Podsumowanie

AGIS ma sens jako modułowa platforma do lokalnego monitorowania i kontrolowanej automatyzacji. Najbardziej wiarygodna wersja projektu nie obiecuje sztucznej ogólnej inteligencji ani pełnej niezależności. Pokazuje natomiast konkretną architekturę, ograniczenia, sposób testowania i mierzalny plan wdrożenia.

**Najważniejsza zmiana komunikacyjna:** zamiast prezentować system jako gotowego, autonomicznego nadzorcę, należy przedstawiać go jako rozwijany prototyp, którego autonomia jest stopniowo zwiększana po spełnieniu kryteriów bezpieczeństwa i jakości.
