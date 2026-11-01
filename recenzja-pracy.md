# Recenzja pracy inżynierskiej: stan na 1 listopada 2026

**Oceniany tekst:** `Tomsz Nejman praca inż. przegląd literatury i rozwiązań.pdf` (strony 1–5) oraz jego kopia w `main.tex`.
**Kontekst:** historia gita i nagłówki repozytoriów `heroes3-with-ai`, `heroes3_with_ai_old`, `heroes3-with-ai-demo`.
**Co zmieniono w pracy:** nic. Ten plik jest jedynym efektem recenzji. Niczego nie usunięto, `main.tex` pozostał bez zmian.

---

## 0. Werdykt w skrócie

Obecny tekst to dobry szkic przeglądu, ale jeszcze nie praca. Ma około 1700 słów, czyli 5 stron. W gotowej pracy będzie to mniej niż 10% objętości. Największy problem nie leży w języku, tylko w tym, że **tekst nie mówi, co praca chce udowodnić, a repozytorium nie zawiera jeszcze tego, co tekst zapowiada**.

| Obszar | Ocena | Co o tym przesądza |
|---|---|---|
| Oś i sens pracy | **słabo** | Brak celu, tezy i pytania badawczego. Sekcje „Wstęp” i „Cel pracy” są puste. |
| Zgodność z projektem | **słabo** | Tekst mówi o ciągłym stanie i tanim kopiowaniu, a stan w repozytorium to graf obiektów na `shared_ptr`. Nie ma żadnych pomiarów. |
| Autorstwo | **ryzyko** | Bitwa, MCTS i pętla AI mają ślady autorstwa Tomasza Naszkowskiego. Tekst mówi „mój silnik”. |
| Poprawność faktów | **średnio** | Część rzeczy jest dobrze ujęta (Shannon, $M^N$, Gregory), ale jest kilka twardych błędów (rozdział 7). |
| Przegląd literatury | **średnio** | Dobry trop, lecz brak Stratega, vcmi-gym, podstaw Data-Oriented Design i wniosku końcowego. |
| Język | **średnio** | Czytelny, ale publicystyczny: pierwsza osoba, hiperbole, anglicyzmy, literówki składniowe. |

**Pięć rzeczy do zrobienia najpierw:**

1. Ustalić z promotorem temat i podział autorstwa z Tomaszem Naszkowskim (rozdział 4).
2. Zapisać cel i pytania badawcze (rozdział 2).
3. Wybrać wariant projektu i zmierzyć obecny `Battle::copy()` **przed** jakąkolwiek przebudową, bo to będzie punkt odniesienia (rozdział 3).
4. Poprawić błędy rzeczowe z rozdziału 7, zaczynając od: pamięci Atari 600XL, „deterministycznej” funkcji przejścia, StarCrafta II, opisu microRTS i architektury VCMI.
5. Domknąć przegląd porównaniem i wskazaniem luki (rozdział 8).

**Co jest naprawdę dobre i warto zachować:**

- pomysł otwarcia: zaczynamy od granicy między silnikiem a grą;
- ostatnie zdanie rozdziału 2: projektowanie architektury to „nieustanne zarządzanie kompromisem”. To najlepsza myśl w tekście;
- rozwinięcie Shannona w stronę gier 4X;
- spostrzeżenie, że `Forward Model` zależy od kosztu kopiowania stanu. To właściwy rdzeń pracy;
- świadome zawężenie przeglądu: nie renderowanie, tylko zarządzanie stanem;
- krytyka modelu pamięci w Javie jako kierunek rozumowania, po podparciu pomiarami.

---

## 1. Jak czytać tę recenzję

**Zakres odczytu repozytoriów.** Przeczytano historię commitów, drzewo plików, dokumentację i wybrane nagłówki. Implementacji (`.cc`) nie czytano. Wzorce w `.cc` jedynie zliczono wyszukiwaniem. Wnioski o kodzie dotyczą więc **deklaracji, nie działania**, i mogą wymagać korekty.

**Oznaczenia wiarygodności:**

| Znak | Znaczenie |
|---|---|
| ✅ | Potwierdzone w źródle pierwotnym, które zostało odczytane |
| 🟡 | Potwierdzone pośrednio: abstrakt, opis wtórny, wyszukiwarka |
| ❓ | Wiedza recenzenta bez odczytania źródła. Trzeba sprawdzić przed użyciem |
| ⚠️ | Twierdzenie częściowo poprawne albo źródło go nie dowodzi |
| ❌ | Twierdzenie błędne lub niepoparte |

**Czego nie udało się przeczytać:**

- pełnego tekstu artykułu o Tribes (AAAI blokuje automatyczny dostęp, a strony autorów nie mają pliku PDF);
- książki Gregory'ego (sprawdzono tylko wykład streszczający rozdział 1);
- strony AtariWiki z kodem PAC-MAN-a (błąd 502);
- książek Wolfa i Kenta.

---

## 2. O czym ma być ta praca

### 2.1. Diagnoza

Tekst krąży wokół trzech tematów: historii silników, algorytmów AI i kopiowania stanu. Nigdzie nie pada zdanie, które je łączy. Czytelnik dowiaduje się, że istnieje „mój silnik”, dopiero w środku rozdziału o istniejących rozwiązaniach, a jego cel trzeba odgadnąć z akapitu o zakresie przeglądu.

Oś, która wynika z tekstu, notatek i repozytorium, jest jedna i bardzo dobra:

> Algorytmy planowania są ograniczone liczbą symulacji w dostępnym czasie → liczbę symulacji ogranicza koszt kopiowania stanu i przejścia → ten koszt zależy od reprezentacji danych → dlatego projektujemy silnik z interfejsem Forward Model i sprawdzamy pomiarami, ile da się w ten sposób zyskać.

Każdy akapit pracy powinien przybliżać do tego łańcucha. Jeśli tego nie robi, trzeba go skrócić albo wyrzucić.

### 2.2. Propozycja celu

> **Celem pracy jest zaprojektowanie i implementacja silnika uproszczonej turowej gry strategicznej opartej na *Heroes of Might and Magic III*, udostępniającego agentom sztucznej inteligencji interfejs Forward Model, oraz zbadanie, jak reprezentacja stanu bitwy wpływa na koszt symulacji i przepustowość algorytmów przeszukiwania.**

**Zakres:** pełny Forward Model tylko dla bitwy. Mapa świata może pozostać zwykłą częścią gry.

**Pytania badawcze:**

- **P1.** Ile kosztuje kopia stanu i pojedyncze przejście w reprezentacji obiektowej, a ile w reprezentacji ciągłej?
- **P2.** Jaką część czasu przeszukiwania zajmuje obsługa stanu?
- **P3.** Czy szybszy Forward Model daje silniejszą grę przy stałym budżecie czasu?

P3 jest najciekawsze, bo odpowiada na zarzut „i co z tego, że szybciej?”. Może też dać wynik negatywny, co w pracy badawczej jest w pełni uczciwym rezultatem.

### 2.3. Propozycje tytułu

Tytuł musi zgadzać się z tematem zatwierdzonym na wydziale, więc poniższe to tylko punkt wyjścia do rozmowy:

1. *Silnik turowej gry strategicznej z interfejsem Forward Model dla agentów sztucznej inteligencji* / *A turn-based strategy game engine with a Forward Model interface for AI agents*
2. *Reprezentacja stanu gry a wydajność algorytmów planowania na przykładzie silnika gry typu Heroes of Might and Magic III* / *Game state representation and the performance of planning algorithms in a Heroes of Might and Magic III-like engine*

### 2.4. Pytania, które padną na obronie

1. Co dokładnie jest Pana/Pani wkładem, a co zrobił drugi autor projektu?
2. Czym ta praca różni się od VCMI i Stratega? Dlaczego nie wystarczyło rozszerzyć jednego z nich?
3. O ile szybszy jest nowy stan i jak to zmierzono?
4. Czy szybszy silnik faktycznie poprawia grę agenta?
5. Jak silnik radzi sobie z losowością, skoro obrażenia, szczęście i morale są losowe?
6. Dlaczego kopiowanie, a nie cofanie ruchów?
7. Które mechaniki HoMM III pominięto i czy to nie zniekształca wyników?
8. Co to znaczy, że silnik jest silnikiem, a nie po prostu grą?

Praca powinna odpowiadać na każde z tych pytań. Dziś nie odpowiada na żadne.

---

## 3. Tekst a stan projektu

### 3.1. Historia projektu to gotowa narracja

| Okres | Co się działo | Co daje pracy |
|---|---|---|
| 03–06.2025 | Projekt ZPR (zespół TNT): model obiektowy, bitwa, minimax z alfa-beta, wersja równoległa | Geneza problemu |
| 05–06.2025 | `Began changing Battle to save memory`, `Changed Artifact members to one ArtifactData...`, `added copping to most classes`, `Changed copying methods to return shared_ptr<T>` | Pierwsze zderzenie z kosztem kopiowania. Już wtedy pojawia się podział na dane wspólne i zmienne |
| 07.2026 | Nowe repozytorium, CI, flagi kompilatora, pokrycie testami | Inżynieria i jakość |
| 08.2026 | `separated rendering from game logic`, `Separated input gathering from game logic`, `removed global SFML::Graphics dependency`, `changed map to T from shared_ptr<T>`, `optimized enum sizes` | Dojście do granicy silnik/gra, czyli praktyka rozdziału 2 |
| 09.2026 | `restructure project; extract engine to module`, `Update copy for Character and all dependencies` | Wydzielenie silnika |
| 10.2026 | `fully moved to BattleCommand from Move`, `Created baseline UCT MCTS. Missing rollout policy`, `begin modularizing graphics` | Polecenia jako dane, początek MCTS |

To, czego w tekście najbardziej brakuje, już się wydarzyło. Pasuje szczególnie przejście z `shared_ptr<Move>` na małą strukturę `BattleCommand { action, destination }`. To gotowy przykład odejścia od obiektów w stronę danych.

### 3.2. Deklaracje a repozytorium

| Tekst lub notatki mówią | Repozytorium pokazuje | Wniosek |
|---|---|---|
| Stan jako dane ciągłe w pamięci | `BattleGrid` to tablica `shared_ptr<Tile>`. `Battle` trzyma `shared_ptr<BattleField>`, dwa `shared_ptr<Character>` i kolejkę `reference_wrapper<UnitStack>`. W nagłówkach `main`: 88 × `shared_ptr`, 47 × `virtual`, 0 × `alignas`, 0 × `memcpy` | Stan to graf obiektów. Nie da się go skopiować jednym blokiem |
| Tanie kopiowanie | Są metody `copy()`: `Battle`, `Character`, `Army`, `Tile` (12 wystąpień w nagłówkach) | Kopiowanie istnieje, ale obiektowe. To gotowy wariant bazowy do porównania |
| API Forward Model | `IState` ma `legalCommands()`, `isLegalCommand()`, `applyCommand()`. `Game` ma odpowiedniki | Bardzo dobry zalążek. Brakuje kopii na poziomie interfejsu, oceny stanu końcowego i kontroli losowości |
| Command Pattern | `UserCommand` to `std::variant`, a `BattleCommand` to zwykłe dane | To lepsza, „danowa” wersja wzorca. Warto ją opisać |
| Agenci AI | `MinimaxAI` na `main`. `IAlgorithm` i `MCTSAlgorithm` tylko na `feature/AIInterface`, gdzie wciąż używają `Move`, a `main` przeszedł na `BattleCommand`. Commit mówi `Missing rollout policy` | Bez scalenia tych gałęzi nie będzie czego mierzyć |
| Deterministyczna funkcja przejścia | `UnitData` ma `min_damage_` i `max_damage_`, `UnitStack` ma `morale_` i `luck_`, a w `Battle.cc` są 2 trafienia wzorca losowości | Reguły są losowe, więc tekst przeczy projektowi |
| Niezależność silnika od algorytmu | `Battle` ma pole `is_minimax_` i konstruktor z tą flagą | Logika gry wie o algorytmie. Do usunięcia przed opisem architektury |
| Pomiary (VTune, `perf` w notatkach) | Brak benchmarków. `chrono` pojawia się tylko przy zakomentowanych opóźnieniach w `Main2.cc` | Dziś nie ma części badawczej |

**Dobra wiadomość:** struktura `engine/model`, `engine/platform`, `core`, `aux`, `app` to praktyczna realizacja granicy z rozdziału 2. Rozmiar `main` to około 9,2 tys. wierszy kodu i 1,3 tys. wierszy testów. Dla porównania oryginalna gra miała około 250 tys. wierszy C++ ✅, a VCMI ponad 300 tys. ✅. Te liczby dobrze uzasadniają uproszczenie zakresu.

### 3.3. Decyzja, którą trzeba podjąć

**Wariant A (zalecany): płaski stan bitwy i pomiar względem wersji obiektowej.**
Tylko bitwa przechodzi na ciągły stan. Obecny `Battle` zostaje jako wariant bazowy, więc grupa kontrolna już istnieje. Praca dostaje prawdziwe wyniki i spójny łańcuch od historii do pomiarów.

Punkty projektowe:

- dane statyczne zostają w tablicy `UnitData`, która już działa jak wzorzec *flyweight*;
- zamiast `reference_wrapper` i wskaźników trzeba użyć indeksów. Tak samo Tribes trzyma na planszy identyfikatory ✅;
- stan generatora liczb losowych musi być częścią stanu;
- `static_assert(std::is_trivially_copyable_v<...>)` da twardy dowód, że stan można kopiować blokiem.

Skala jest niewielka. 22 oddziały (po 7 + 4 machiny na stronę), 165 pól, kolejka i stan RNG to **szacunkowo** 0,5–1 KB, czyli kilkanaście linii pamięci podręcznej. Liczbę trzeba potwierdzić przy implementacji.

**Wariant B: uczciwy opis obecnej architektury.**
Praca dotyczy separacji silnika, poleceń i maszyny stanów, a o DOD mówi tylko jako o kierunku rozwoju. Mniej pracy, ale trzeba przepisać rozdział 4, usunąć tezę o koszcie kopiowania jako motywację i liczyć się z pytaniem „gdzie jest wynik?”.

**Czego nie robić:** zostawić tekstu o DOD bez implementacji i pomiarów. Recenzent zajrzy do repozytorium.

**Warianty dodatkowe, jeśli starczy czasu:** Pomysł 1 z notatek (kopiowanie przy zapisie) oraz cofanie ruchów. W szachach wybór między kopiowaniem a cofaniem zależy od stosunku rozmiaru stanu do liczby zmienionych bitów 🟡, a VCMI w `HypotheticBattle` stosuje nakładkę podobną do kopiowania przy zapisie ✅. Trzy porównane strategie to znacznie mocniejszy rozdział niż jedna.

### 3.4. Niewykorzystany argument z notatek

Notatka „minimax pracuje 1000 godzin, 8 TB RAM-u dla głębokości 6 (tak naprawdę 3)” to potencjalnie najmocniejsza motywacja w całej pracy. Jest to wynik z własnego projektu, ale w tej formie nie nadaje się do użycia. Trzeba go odtworzyć, opisać warunki pomiaru i wyjaśnić, co znaczy „głębokość”. README wersji demo mówi o stałej głębokości 5, więc liczby trzeba uzgodnić.

---

## 4. Autorstwo i samodzielność

To sprawa ważniejsza niż styl, bo praca ma zawierać oświadczenie, że „została napisana przeze mnie samodzielnie”.

**Fakty z repozytoriów:**

- Projekt powstał jako zespołowe zadanie z ZPR. Plik `documentation/Naszkowski_Nejman.pdf` podaje zespół: Tomasz Nejman i Tomasz Naszkowski.
- W starym repozytorium jest 128 commitów Tomasza Nejmana i 119 Tomasza Naszkowskiego (`Tomasz Naszkowski`, `tomek`, `tnaszkow`). Bitwa, minimax i grafika mają mocny udział drugiego autora.
- W obecnym repozytorium 8 commitów z październka 2026 podpisano `--global <tomek.naszkowski05@gmail.com>`. Dotyczą `IAlgorithm`, MCTS, Minimaxa, pętli AI w `Main.cc` i `Player`.
- 15 nagłówków na `main` ma znacznik `Autor: Tomasz Naszkowski`, w tym `Battle.h`, `BattleField.h`, `Tile.h`, `UnitStack.h`, `Game.h`, `Player.h`.
- Od lipca 2026 wszystkie 8 commitów w module bitwy na gałęzi `main` należy do Tomasza Nejmana, więc moduł mógł się mocno zmienić. Na gałęzi AI `Battle.cc` zmieniał też Tomasz Naszkowski.

Liczba zmienionych wierszy mocno przemawia na korzyść Tomasza Nejmana (około 46 tys. wobec 7 tys. w starym repozytorium), ale obejmuje też przenoszenie i formatowanie, więc nie jest dobrą miarą autorstwa.

**Co zrobić:**

1. Zapytać promotora, czy kod z zaliczonego projektu ZPR może być podstawą pracy i jak to opisać.
2. Ustalić, czy Tomasz Naszkowski pisze własną pracę na tej bazie. Jeśli tak, granica musi być jasna: na przykład silnik i stan po jednej stronie, algorytmy po drugiej. Uwaga: jeśli MCTS pozostaje czyjąś częścią, to P3 trzeba oprzeć na uzgodnionym użyciu tego kodu.
3. Dodać do wstępu krótką sekcję o zakresie prac własnych z tabelą moduł → autor.
4. Zastąpić „mój silnik” sformułowaniem zgodnym z tym podziałem.
5. Zaktualizować nieaktualne znaczniki `Autor:`, jeśli pliki zostały napisane od nowa.

**Porządek w repozytorium przed oddaniem:** autor `--global` to efekt błędnego `git config`. Warto też ujednolicić adres: 28 commitów ma służbowy e-mail `trumpf.com`, co warto sprawdzić pod kątem zasad pracodawcy, jeśli praca powstawała na firmowym sprzęcie. W gałęzi `main` nie ma pliku `LICENSE`. Sprite'ów z oryginalnej gry nie należy dołączać do pracy bez sprawdzenia praw, bo README wersji demo pomija je właśnie jako własność Ubisoftu.

---

## 5. Proponowany pełny spis treści

Oznaczenia: `[jest]` tekst istnieje, `[popraw]` trzeba przebudować, `[nowe]` do napisania, `[kod]` wymaga pracy w repozytorium. Przybliżona objętość to 60–80 stron. Przy wariancie B rozdziały 6–8 będą krótsze.

```text
Streszczenie / Abstract
Oświadczenia
Spis treści

1. Wstęp (4 s.) [nowe]
   1.1. Kontekst i motywacja
   1.2. Cel, pytania badawcze i zakres pracy
   1.3. Zakres prac własnych
   1.4. Układ pracy

2. Architektura silników gier (12 s.)
   2.1. Silnik a gra: definicje i granica [popraw]
   2.2. Od układów cyfrowych do silników wielokrotnego użytku [popraw, skrócić]
   2.3. Warstwy silnika i pętla gry [nowe]
   2.4. Zarządzanie stanem: maszyny stanów, polecenia, ECS [nowe]
   2.5. Hierarchia pamięci i Data-Oriented Design [nowe]
   2.6. Specyfika silników gier strategicznych [popraw]

3. Planowanie w grach turowych (9 s.)
   3.1. Złożoność drzewa gry [jest, przenieść z rozdz. „Istniejące rozwiązania”]
   3.2. Minimax i przycinanie alfa-beta [nowe]
   3.3. Monte Carlo Tree Search i Statistical Forward Planning [nowe]
   3.4. Losowość i niepełna informacja [nowe]
   3.5. Forward Model: definicja i wymagania [popraw]
   3.6. Strategie obsługi stanu: kopiowanie, cofanie, współdzielenie [nowe]

4. Przegląd istniejących rozwiązań (12 s.)
   4.1. Zakres i kryteria porównania [popraw]
   4.2. Środowiska badawcze: Tribes, microRTS, Stratega [popraw, Stratega nowe]
   4.3. Wysokowydajne symulatory wsadowe: Pgx, Madrona [nowe]
   4.4. Rozwiązania dla HoMM III: VCMI, vcmi-gym, MMAI [popraw]
   4.5. Silniki komercyjne [nowe, krótko]
   4.6. Porównanie i wnioski: luka, którą wypełnia praca [nowe]

5. Wymagania (4 s.) [nowe]
   5.1. Odwzorowane i pominięte mechaniki HoMM III
   5.2. Wymagania funkcjonalne
   5.3. Wymagania niefunkcjonalne (mierzalne)
   5.4. Wybór technologii: C++26, CMake, SFML, GoogleTest

6. Projekt silnika (12 s.) [nowe]
   6.1. Architektura modułów: app, core, engine, aux
   6.2. Oddzielenie logiki od renderowania i wejścia
   6.3. Maszyna stanów gry: StateStack, IState
   6.4. Polecenia jako dane: UserCommand, SystemCommand
   6.5. Interfejs Forward Model
   6.6. Projekt stanu bitwy: dane statyczne, dane zmienne, losowość
   6.7. Interfejs agentów

7. Implementacja (10 s.) [nowe, kod]
   7.1. Środowisko i proces budowania, CI
   7.2. Stan bitwy i jego kopiowanie
   7.3. Generowanie legalnych poleceń i przejście
   7.4. Integracja agentów (z zaznaczeniem autorstwa)
   7.5. Warstwa prezentacji
   7.6. Testy

8. Badania (10 s.) [nowe, kod]
   8.1. Metodyka: sprzęt, warianty, scenariusze, statystyka
   8.2. Koszt operacji Forward Modelu
   8.3. Pamięć i pamięć podręczna
   8.4. Przepustowość przeszukiwania
   8.5. Siła gry przy stałym budżecie czasu
   8.6. Dyskusja i ograniczenia wyników

9. Podsumowanie (3 s.) [nowe]
   9.1. Wnioski i odpowiedzi na pytania badawcze
   9.2. Kierunki rozwoju

Bibliografia
Wykaz symboli i skrótów
Spis rysunków
Spis tabel
Spis załączników
Załącznik 1. Instrukcja uruchomienia
Załącznik 2. Scenariusze badawcze
Załącznik 3. Pełne wyniki pomiarów
```

**Dlaczego ta kolejność.** Rozdziały 2 i 3 dostarczają pojęć. Rozdział 4 pokazuje lukę. Rozdziały 5–7 ją wypełniają. Rozdział 8 sprawdza, czy się udało. Cel trafia do wstępu, a nie między dwa rozdziały przeglądu, jak w spisie z pliku PDF.

**Metryki do rozdziału 8:**

- czas `copy`, `applyCommand` i `legalCommands` w nanosekundach;
- liczba stanów i iteracji MCTS na sekundę;
- bajty na węzeł i liczba alokacji;
- braki trafień w L1 i LLC z `perf stat`;
- procent wygranych przy równym budżecie czasu.

Warto użyć Google Benchmark, opisać sprzęt i flagi kompilatora, powtarzać pomiary i podawać przedziały ufności. Scenariusze powinny obejmować małą, średnią i pełną bitwę.

**Gdzie trafia obecny tekst:**

| Obecna sekcja w `main.tex` | Słowa | Nowe miejsce |
|---|---:|---|
| Granica między silnikiem a grą | 64 | 2.1 |
| Od układów cyfrowych do monolitycznego kodu | 216 | 2.2 |
| Wzrost mocy obliczeniowej i separacja | 233 | 2.2 i 2.1 (Bishop) |
| Silniki gier strategicznych | 203 | 2.6, a Gregory do 2.1 |
| Złożoność gier turowych | 144 | 3.1 |
| Paradygmat Forward Model | 136 | 3.5 i 3.6 |
| Zakres przeglądu | 99 | 1.2 i 4.1 |
| Tribes | 195 | 4.2 |
| MicroRTS | 171 | 4.2, a $M^N$ także do 3.1 |
| Ograniczenia środowisk opartych na Javie | 69 | 4.2 lub 4.6 |
| VCMI | 154 | 4.4 |

---

## 6. Kształt i logika obecnego tekstu

### 6.1. Rozdział o architekturze nie opisuje architektury

Tytuł zapowiada architekturę, a treść to głównie historia: 449 z 716 słów, czyli 63%. Nie ma warstw silnika, pętli gry, zarządzania zasobami, maszyn stanów ani ECS. Ten rozdział powinien kończyć się tak, by czytelnik rozumiał, czym różni się stan obiektowy od stanu ciągłego.

### 6.2. W historii brakuje ogniwa, które uzasadnia całą pracę

Tekst przechodzi od prawa Moore'a do większej mocy, a potem do separacji silnika. Pomija to, co jest najważniejsze dla tematu: **procesory przyspieszały znacznie szybciej niż pamięć**. Wulf i McKee opisali to w 1995 roku jako „memory wall” 🟡. To dlatego układ danych zaczął decydować o wydajności i dlatego powstało Data-Oriented Design.

Bez tego ogniwa późniejsze wzmianki o `cache miss` i blokach pamięci biorą się znikąd. Pełny łańcuch powinien wyglądać tak:

> sprzęt ograniczony → monolit → więcej mocy i złożoności → silnik i abstrakcje obiektowe → ściana pamięci → koszt rozproszonych danych → DOD i ECS → symulacja na potrzeby AI, gdzie ten koszt mnoży się przez miliony wywołań.

To daje historii puentę i naturalnie prowadzi do tematu pracy. Wtedy można ją spokojnie skrócić o połowę. Warto też zacytować Bishopa: wczesne gry pisano w asemblerze „with little or no intervening system software” ✅. To lepsze uzasadnienie monolitu niż niesprawdzalny PAC-MAN. Bishop opisuje też silnik obiektowy (NetImmerse), więc pasuje jako punkt „złotej ery OOP”, po którym przychodzi krytyka.

### 6.3. Definicja silnika nie pracuje na rzecz projektu

Trzy podejścia (Bishop, Gregory, kryterium danych) warto zebrać w jednym miejscu i odnieść do własnego kodu. Gregory, o ile dobrze pamiętam, wskazuje architekturę sterowaną danymi jako cechę odróżniającą silnik od gry ❓. To niewygodne pytanie do Twojego projektu: `UnitsLib.h` i `SpellLib.h` to tabele w kodzie, a nie dane z pliku. Recenzent może zapytać, co tu jest silnikiem. Lepiej odpowiedzieć na to samemu, wskazując `engine/` i uzasadniając tabele `constexpr` wydajnością.

### 6.4. Rozdział „Istniejące rozwiązania” miesza trzy rzeczy

Zaczyna się od złożoności szachów, przechodzi do Forward Modelu, potem do zakresu pracy, a dopiero po 379 słowach do istniejących rozwiązań. Pierwsze trzy sekcje są dobre, ale należą gdzie indziej (patrz tabela w rozdziale 5).

Przegląd powinien zaczynać się od **kryteriów**, potem opisywać każde rozwiązanie według nich, a kończyć **tabelą i wnioskiem**. Teraz urywa się na niedokończonym zdaniu o VCMI. Plik PDF zapowiada w tytule „rozwiązania komercyjne”, a ich w tekście nie ma, jest tylko lista na stronie 6.

### 6.5. Własny przypadek jest nieobecny, a to zmienia wnioski

Tekst cytuje $M^N$ z pracy Ontañóna, ale nie sprawdza, czy dotyczy to HoMM III. **W bitwie akcję wykonuje naraz jeden oddział**, więc rozgałęzienie jest raczej sumą dostępnych pól, celów, czekania, obrony i czarów niż iloczynem akcji wszystkich jednostek. Eksplozja $M^N$ dotyczy gier, w których w jednej decyzji dowodzi się wieloma jednostkami, jak RTS.

Trudność bitwy leży gdzie indziej:

- jedna runda to wiele kolejnych decyzji, co tłumaczy notatkę „głębokość 6 (tak naprawdę 3)”;
- kolejność stron nie musi być naprzemienna, bo wyznacza ją kolejka oddziałów;
- obrażenia, szczęście i morale są losowe, więc klasyczny minimax nie wystarcza i potrzebne są węzły losowe, próbkowanie albo wartości oczekiwane.

Warto zmierzyć średnią liczbę legalnych poleceń na własnych scenariuszach przez `legalCommands().size()`. To będzie mocniejsze niż liczba tur z Reddita.

### 6.6. Spójność wewnętrzna

- Gregory i rozmyta granica pojawiają się dwa razy, na początku i końcu rozdziału.
- Tekst mówi, że każda gra była „własnym, unikalnym silnikiem”, a chwilę później definiuje silnik jako coś wielokrotnego użytku. To sprzeczność. Monolit to gra **bez** silnika.
- W sekcji o Javie przeciwstawiasz Tribes i microRTS rozwiązaniu w C++, ale własne repozytorium ma dziś te same problemy (rozproszone obiekty i kopie po kolei). Bez wariantu A ten akapit działa przeciwko pracy.

### 6.7. Co dopisać w ramach obecnych sekcji

| Sekcja | Czego brakuje |
|---|---|
| Forward Model | Definicji formalnej, źródła pojęcia, losowości, wymagań mierzalnych |
| Tribes | Wniosku „co z tego dla nas”, nie tylko opisu |
| microRTS | Wyniku pracy Ontañóna, czyli `NaïveMCTS` |
| Java | Pomiaru lub cytowanego źródła, rozróżnienia kosztu alokacji i kosztu pamięci podręcznej |
| VCMI | Zakończenia `HypotheticBattle`, vcmi-gym, MMAI |
| Całość | Podsumowań na końcu rozdziałów i tabeli porównawczej |

---

## 7. Weryfikacja faktów

### 7.1. Historia i definicje

| # | Twierdzenie w tekście | Ocena | Ustalenia | Co zrobić |
|---|---|---|---|---|
| 1 | Atari 600XL: 64 KB | ❌ 🟡 | Model 600XL miał 16 KB, rozszerzalne do 64 KB. 64 KB miał 800XL | Zmienić na 16 KB albo użyć 800XL |
| 2 | Automat PAC-MAN: 2 KB RAM | ⚠️ 🟡 | Źródła podają 3 KB: 1 KB robocze, 1 KB wideo, 1 KB koloru. Spotyka się też 4 KB (przestrzeń adresowa) | Podać 3 KB z rozbiciem i źródłem, a na koniec sprawdzić w sterowniku MAME |
| 3 | Commodore 64: 64 KB, 1982 | ✅ ❓ | Zgodne z powszechną wiedzą | Zacytować podręcznik C64 z listy źródeł |
| 4 | Analiza patentu dowodzi, że PONG nie miał procesora | ⚠️ ✅ | Patent US 3 793 483: zgłoszony 24.11.1972, udzielony 19.02.1974. W odczytanej części (około 60%) nie pada nazwa PONG ani stwierdzenie o braku procesora. Opisuje liczniki, bramki i sumator. Sam fakt jest prawdziwy 🟡 | Zmienić źródło na historię gry lub schemat. W bibliografii rok 1974, nie 1972 |
| 5 | PAC-MAN: logika, AI i joystick wymieszane | ⚠️ 🟡 | Odnośnik prowadzi prawdopodobnie do portu na Atari 8-bit (Roklan), a nie automatu z tabeli. Bibliografia podaje rok 1986, a wydanie portu to 1982. Strona zwróciła błąd 502. Forum wspomina o uszkodzeniach tego kodu | Sprawdzić samodzielnie albo oprzeć monolit na Bishopie |
| 6 | Każda gra była „swoim własnym silnikiem” i nie dało się jej użyć ponownie | ❌ logika | Sprzeczne z własną definicją. Ponowne użycie istniało wcześniej: maszyna Z Infocomu, SCUMM ❓ | Pisać o grach bez wydzielonego silnika, unikać „nie dało się” |
| 7 | Moore, 1965 | ✅ ❓ | Teza znana. Przewidywanie dotyczyło pierwotnie około dekady | Podać pełny opis artykułu |
| 8 | Daty i rodzaje gier 3D | ✅ ❓ | Zgodne | Dodać źródła zamiast `REF P3D` i `REF T3D` albo uprościć zdanie |
| 9 | Kryzys zarządzania stanem wymusił „ostateczną separację” logiki | ❌ niepoparte | Brak źródła, a chronologia nie pasuje. Rozdział 1 Gregory'ego w streszczeniu wykładu z UMD wskazuje na **Dooma** (1993), czyli pseudo-3D, i na rozdzielenie rdzenia, zasobów i reguł, które ułatwiało modyfikacje 🟡 | Przepisać przyczynę: ponowne użycie, koszty, moddowanie. Zob. też Bishop ✅ |
| 10 | Bishop: definicja i cztery składniki | ⚠️ ✅ | Dobre streszczenie, ale brakuje pętli zdarzeń: „all elements in Figure 1 that have no effect on actual content, that is, everything indicated by dashed lines plus an event loop”. „Neither economical nor necessary” dotyczy budowy **komponentów** dla każdego tytułu, nie całych gier | Doprecyzować |
| 11 | Cytat Gregory'ego | ✅ ❓ | Zgodny z pamięcią recenzenta, nieodczytany | Podać stronę, wydanie i „tłum. własne” |
| 12 | Silniki klasyczne vs strategiczne | ⚠️ | Rozsądne, bez źródeł. „Predykcja stanów” jest myląca, bo to właśnie Forward Model | Doprecyzować: przewidywanie po stronie klienta w grach sieciowych |

### 7.2. Złożoność i Forward Model

| # | Twierdzenie | Ocena | Ustalenia | Co zrobić |
|---|---|---|---|---|
| 13 | Shannon: $10^{120}$, 40 ruchów | ✅ | Około 30 ruchów w pozycji, około $10^3$ na parę posunięć, 40 ruchów. To warianty (*variations*), a 40 oznacza pary | Dopisać założenia |
| 14 | Mikrosekunda, $10^{90}$ lat | ✅ z poprawką | Dokładnie: „one variation per micro-second would require **over** $10^{90}$ years”. To nie „średni czas”. Z samych danych wychodzi około $3 \cdot 10^{106}$ lat, więc Shannon podaje luźną dolną granicę | Pisać „ponad”, usunąć „średni” |
| 15 | Gry strategiczne przewyższają szachy „praktycznie zawsze” | ❌ niepoparte | Brak źródła i definicji | Pokazać to na konkretnej miarze |
| 16 | Civilization VI: 350 tur, 10–20 decyzji | ⚠️ | Źródłem jest wątek na Reddicie. Inne źródła niskiej jakości mówią o 300–500 turach i limicie 500. Decyzje bez źródła | Usunąć liczby lub znaleźć publikację |
| 17 | AI opiera się na rozgrywaniu tysięcy gier | ⚠️ | Dotyczy SFP i uczenia ze współzawodnictwa, ale nie minimaxa | Zawęzić do SFP |
| 18 | Silnik „musi” dostarczać deterministyczną funkcję | ❌ | Forward Model nie musi być deterministyczny. README Tribes mówi wprost o grze „stochastic” ✅. microRTS ma tryb niedeterministyczny ✅. HoMM III jest losowe | Użyć zapisu z ziarnem albo rozkładem, zob. 11.3 |
| 19 | `(S+1)` | ❌ zapis | Wygląda jak dodawanie | $s_{t+1} = f(s_t, a_t)$ |
| 20 | Algorytmy drzewiaste „wymagają” kopiowania stanu | ⚠️ | Wymagają rozgałęzienia stanu. Alternatywy: cofanie ruchu 🟡, nakładka `HypotheticBattle` ✅, współdzielenie `cloneIssue()` w microRTS ✅ | Opisać strategie, a nie jedną |
| 21 | StarCraft II przez lata nie miał FM przez złożoność | ❌ niepoparte | `REF STARCRAFT` nie ma pozycji. Artykuł Vinyalsa i in. w ogóle nie mówi o Forward Modelu ani o przyczynie jego braku ✅. Nie ma śladu, że został później dodany | Usunąć albo zastąpić przykładem vcmi-gym |
| 22 | Kopiowanie bloków „(cache hit)” | ❌ pojęcie | Trafienie w pamięci podręcznej to nie to samo co kopiowanie bloku. Koszt `memcpy` rośnie też z rozmiarem stanu | Zob. 11.4 |

### 7.3. Tribes i microRTS

| # | Twierdzenie | Ocena | Ustalenia | Co zrobić |
|---|---|---|---|---|
| 23 | Tytuł z „Strategy Games” | ❌ | „Game”, liczba pojedyncza ✅. Strona autorów ma „for AI”, Twoja notatka „for AI Research” | Sprawdzić wersję w materiałach AIIDE |
| 24 | Bazuje na Civilization i Polytopii | ⚠️ ✅ | README: „open source implementation [...] of the game The Battle of Polytopia” | Napisać: implementacja gry *The Battle of Polytopia* |
| 25 | Plansza N × N | ✅ | Dla dwóch graczy 11 × 11 | Podać przykład |
| 26 | Mgła wojny | ✅ z poprawką | Dostępna, ale opcjonalna: `PLAY_WITH_FULL_OBS = true` | Zaznaczyć |
| 27 | Command Pattern | ✅ | `Action` (dane), `ActionCommand.execute`, np. `MoveCommand`. Bez cofania, co też zgadza się z tekstem | Można rozwinąć |
| 28 | Powodem był narzut kopiowania | ❓ | Abstrakt i README mówią o badaniach nad grą, przestrzeni akcji i porównaniu agentów. O kopiowaniu nie ma mowy. Pełnego tekstu nie odczytano | **Sprawdzić w artykule**. Jeśli brak, usunąć |
| 29 | Tribes używa płaskich struktur i prymitywów | ⚠️ ✅ | Plansza to tablice `int[][]` i enumów, ale aktorzy leżą w `HashMap<Integer, Actor>` i są głęboko kopiowani jeden po drugim. Kopia dostaje nowy `Random` | Opisać dokładnie. To argument dla pracy, bo stan nie jest ciągły |
| 30 | Ontañón, tytuł i $M^N$ | ✅ | Strony 58–64. Przykład: 10 jednostek × 5 akcji daje $5^{10}$ ≈ 10 mln. Na mapie `FullGame8x8` maksymalne rozgałęzienie to 623 700 | Dodać liczby |
| 31 | „eksploracja-eksploracja” | ❌ | Ma być eksploracja–eksploatacja | Poprawić |
| 32 | Decyzje „asynchronicznie, w dowolnym momencie” | ❌ | Autor: gracze „can issue actions simultaneously”, akcje są trwałe. Rozkazu nie można przerwać ✅ | „równocześnie” |
| 33 | microRTS implementuje Command Pattern | ❌ niepoparte | Dokumentacja o tym nie mówi. Najbliżej jest `uaa.action.execute(...)` | Usunąć albo sprawdzić w kodzie |
| 34 | microRTS radzi sobie przez abstrakcję akcji i nie planuje co tick | ❌ odwrócone | Abstrakcję stosują **inne** algorytmy i konkretne boty. Praca Ontañóna proponuje NaïveMCTS, który działa na pełnych akcjach gracza ✅. Silnik generuje akcje jednostek ✅ | Przepisać, zob. 11.5 |
| 35 | Kopiowanie „globalnym copy” | ⚠️ ✅ | `GameState.clone()`. Jest też `cloneIssue()`, „quick clone” współdzielący część stanu | Podać nazwy. To ciekawy przykład współdzielenia |
| 36 | Stan projektu microRTS | brak w tekście | Od 11.08.2025 repozytorium jest oznaczone jako *deprecated* ✅ | Warto dodać |
| 37 | Java: obiekty na stercie | ✅ ❓ | Zgodne ze specyfikacją, choć JIT może usuwać alokacje przez analizę ucieczki 🟡 | Dodać zastrzeżenie |
| 38 | „Kaskada braków trafienia”, „wysoce nieoptymalne” | ⚠️ niepoparte | Hipoteza rozsądna, ale nie zmierzona. Alokacje w TLAB są ułożone obok siebie 🟡, więc świeża głęboka kopia może mieć niezłą lokalność. Koszt może leżeć w alokacji, GC i wskaźnikach | Ująć jako hipotezę albo zmierzyć |

### 7.4. HoMM III i VCMI

| # | Twierdzenie | Ocena | Ustalenia | Co zrobić |
|---|---|---|---|---|
| 39 | Oryginał w C++ | ✅ | Ryan Gordon: „approximately 250,000 lines of C++ code”, edytor 75 tys. | Podać tę liczbę. Tekst nie ma daty, około 2000 r. |
| 40 | VCMI „od zera” | ✅ z poprawką | „Built from the ground up”, ale z oryginalnymi zasobami. Do uruchomienia potrzebne są dane gry ✅ | Dopisać |
| 41 | Replikacja „1 do 1” | ⚠️ ✅ | FAQ: jak najwierniej, ale z poprawkami, opcjonalnymi funkcjami i naciskiem na modowanie | Zmienić |
| 42 | Serwer jako osobny proces, narzut sieciowy | ⚠️ nieaktualne | Od 1.5 gra nie uruchamia osobnego `vcmiserver` na komputerach 🟡, a od 1.6.6 gra jednoosobowa nie wymaga połączenia sieciowego ✅. Serwer dedykowany nadal istnieje. Strona `Networking` wciąż pisze o TCP, a `Code_Structure` o komunikacji wewnątrzprocesowej | Przepisać i podać wersję |
| 43 | Brak Forward Modelu | ✅ z uzupełnieniem | Ogólnego FM nie ma, ale `BattleAI` używa `HypotheticBattle` i `BattleExchangeEvaluator` ✅. To właśnie urwana treść z `@TODO` | Dokończyć, zob. 11.6 |
| 44 | Twórcy nie myśleli o agentach AI | ⚠️ | Pierwotnie tak, ale istnieje MMAI w wersji 1.7 i vcmi-gym ✅ | Uzupełnić |

**Dwa cytaty, które warto wykorzystać.** Oba pochodzą z opisu vcmi-gym ✅ i są najlepszym argumentem za Twoim projektem:

- „VCMI battles are not designed to ‘rewind’ cleanly.”
- Wyuczony model świata okazał się „not a viable replacement for the VCMI game engine for the purposes of MCTS, which requires a much more faithful simulator”.

Autor vcmi-gym podaje też punkt odniesienia dla szybkości: 862 akcje na sekundę w samym VCMI na M2 przy mapie 2×2 ✅. Warto go przywołać ostrożnie, bo to inne warunki.

---

## 8. Braki w przeglądzie literatury

### 8.1. Pozycje do dodania

| Pozycja | Dlaczego jest potrzebna | Status |
|---|---|---|
| **Stratega** (Perez-Liebana, Dockhorn, Hurtado Grueso, Jeurissen, 2020) | Najbliższy odpowiednik: gry strategiczne, FM jako główny komponent, stan jako „data container without any logic”. **Recenzent zna to z marszu** | ✅ |
| **vcmi-gym i MMAI** (Manolov) | Uczenie agentów bitew HoMM III na VCMI. Bez tego zdanie, że VCMI nie służy AI, jest podważalne | ✅ |
| Wilisowski, Dreżewski (2015) | Programowanie genetyczne i TD(1) dla bitew VCMI. Praca z polskiego ośrodka | 🟡 |
| Pgx (Koyamada i in., NeurIPS 2023) | Szybkie symulatory gier dla AlphaZero | 🟡 |
| Madrona (Shacklett i in., ACM TOG 2023) | ECS i DOD użyte właśnie do symulacji dla AI. **Najlepszy most między rozdziałami 2 i 4** | 🟡 |
| Wulf, McKee (1995) | Ściana pamięci | 🟡 |
| Browne i in. (2012) | Przegląd MCTS | 🟡 |
| Kocsis, Szepesvári (2006) | UCT | 🟡 |
| Nystrom, rozdział o lokalności danych | Książka jest już na liście, ale ten rozdział nie jest wykorzystany | ❓ |
| Drepper (2007), Acton (CppCon 2014), Fabian (2018) | Pamięć i DOD | ❓ |
| Knuth, Moore (1975) | Alfa-beta | ❓ |
| Yannakakis, Togelius (2018) | Podręcznik AI w grach | ❓ |
| ELF, OpenSpiel, Griddly | Inne platformy badawcze, do krótkiego wyliczenia | ❓ |

### 8.2. Silniki komercyjne

Lista ze strony 6 jest cenna, ale ma być krótka. Wystarczy jeden akapit lub tabela. Firaxis, Clausewitz, Unreal i Unity są zamknięte albo ogólne i nie oferują publicznego, szybkiego Forward Modelu dla gier strategicznych. To zdanie wymaga źródeł. Materiały GDC o Civilization V z Twojej listy warto wykorzystać, jeśli mówią o stanie gry. Silnik Gamebryo (Civilization IV) to prawdopodobnie następca NetImmerse opisanego przez Bishopa ❓, co ładnie wiązałoby rozdziały.

### 8.3. Szkielet tabeli porównawczej

| Kryterium | Tribes | microRTS | Stratega | VCMI | Ten projekt |
|---|---|---|---|---|---|
| Język | Java ✅ | Java ✅ | C++ 🟡 | C++20 ✅ | C++26 |
| Stan | tablice + `HashMap` aktorów | obiekty, `clone()` | kontener danych bez logiki | sprawdzić | do uzupełnienia |
| FM | `advance`, `copy` | `issue`, `cycle`, `clone` | główny komponent | brak ogólnego, `HypotheticBattle` | `applyCommand` i kopia |
| Czas | turowa | rzeczywisty | głównie turowa | turowa | turowa |
| Losowość | tak | domyślnie nie | sprawdzić | tak | tak |
| Obserwacja | pełna lub częściowa | pełna lub częściowa | sprawdzić | częściowa na mapie | pełna w bitwie |
| Rozgrywka bez GUI | sprawdzić | sprawdzić | tak 🟡 | ograniczona | **wymaganie do spełnienia** |
| Mechanika HoMM | nie | nie | nie | tak | tak, uproszczona |
| Pomiary FM | sprawdzić w artykule | sprawdzić | brak w artykule ✅ | częściowe | rozdział 8 |

### 8.4. Formułowanie luki

Trzeba uważać. Stratega ma FM i C++, a VCMI ma HoMM III. Teza „nikt tego nie zrobił” nie obroni się. Bezpieczniej:

> Spośród przeanalizowanych rozwiązań środowiska badawcze zapewniają Forward Model, ale nie obejmują mechaniki walki HoMM III, a VCMI odwzorowuje tę mechanikę, lecz nie udostępnia ogólnego modelu przejść. W pracy zbadano więc, jak zaprojektować stan uproszczonej bitwy tego typu, aby był tani w symulacji, i ile daje to w praktyce.

To skromniejsze, ale obroni się na obronie.

---

## 9. Źródła i bibliografia

### 9.1. Mapa znaczników `REF`

| Znacznik | Proponowana pozycja | Uwagi |
|---|---|---|
| `REF GEA` | Gregory J., *Game Engine Architecture*, wyd. 3, CRC Press 2018 | Istnieje też wyd. 4. Podać używane wydanie i strony |
| `REF NEEDED` (Unreal) | Do znalezienia | Wywiad, post-mortem lub książka Gregory'ego |
| `REF PONG` | Zmienić źródło | Patent zostawić tylko jako uzupełnienie, rok 1974 |
| `REF PACMAN` | Do weryfikacji | Port, nie automat |
| `REF MOORE` | Moore G. E., *Cramming more components onto integrated circuits*, Electronics, 1965 | Link jest na Twojej liście źródeł |
| `REF P3D`, `REF T3D` | Brak | Gregory lub usunąć |
| `REF BISHOP` | Bishop L., Eberly D., Whitted T., Finch M., Shantz M., *Designing a PC Game Engine*, IEEE CG&A, I/II 1998 | Sprawdzić wolumin i strony (❓ 18(1), 46–53) |
| `REF SHANNON` | Shannon C. E., *Programming a Computer for Playing Chess*, Philosophical Magazine, ser. 7, vol. 41, no. 314, marzec 1950, s. 256–275 | ✅ strony 🟡 |
| `REF CIV` | Usunąć Reddita | Lub zastąpić publikacją |
| `REF STARCRAFT` | Brak | Usunąć twierdzenie |
| `REF TRIBES` | Perez-Liebana D., Hsu Y.-J., Emmanouilidis S., Khaleque B., Gaina R., AIIDE 2020 | Sprawdzić tytuł i strony |
| `REF MULTI ARMED BANDIT` | Ontañón S., AIIDE 2013, s. 58–64 | ✅ |
| `REF NEEDED` (Java) | Własny pomiar lub Drepper | Materiały Intela dotyczą ogółu, nie tych projektów |
| `REF ICCULUS` | Gordon R. C., *From MFC to GTK+* lub właściwy tytuł | Bez daty. Podać dostęp |
| `REF VCMI` | Repozytorium i dokumentacja | Podać wersję |

### 9.2. Uwagi ogólne

- `wut-thesis.cls` używa stylu `ieee`, więc zapis „Perez-Liebana et al. (2020)” nie pasuje. W tekście: „Perez-Liebana i in. [12]”.
- `main.tex` nie ma `\addbibresource` ani `\printbibliography`. Potrzebny jest plik `bibliografia.bib`.
- Reddit, YouTube i blogi to słabe źródła. Dopuszczalne są tylko przy dokumentacji projektów i danych niedostępnych gdzie indziej, z datą dostępu.
- Link Google Books na stronie 7 ma w adresie zapytanie `enacting platforms feminist technoscience`. Nie wiadomo, jakiej książki dotyczy. Trzeba to wyjaśnić albo usunąć.
- Prace dyplomowe z Theseus i Aalto ze strony 6 są dobre do szukania pomysłów, ale nie powinny być głównym oparciem.
- Na stronach 6–8 są prywatne dopiski, na przykład `kod pacmana ziuuu`. W `main.tex` ich nie ma i tak ma zostać.

---

## 10. Język i styl

### 10.1. Problemy systemowe

| Problem | Przykład | Zamiana |
|---|---|---|
| Pierwsza osoba i czas przyszły | „będę skupiał się”, „mojego silnika” | „w przeglądzie skupiono się”, „projektowanego silnika” |
| Hiperbole | „monumentalne”, „absolutnie”, „gigantyczną”, „drastycznie”, „wysoce” | Liczba albo brak przymiotnika |
| Wypełniacze | „warto wspomnieć”, „przede wszystkim”, „tak naprawdę”, „de facto”, „co więcej” | Usunąć |
| Mowa potoczna | „per rozgrywka”, „rozstrzał”, „w sensownym czasie”, „do tego” | „na partię”, „zróżnicowanie”, „w akceptowalnym czasie” |
| Anglicyzmy | „dedykowanych”, „ewaluując”, „etc.”, „wzorzec Command Pattern” | „specjalizowanych”, „oceniając”, „itd.”, „wzorzec Polecenie (ang. *Command*)” |
| Niespójne nazwy | FM, Forward Model, FM API, API typu FM. HoMM III, Heroes III. MicroRTS, microRTS | Jedna nazwa, skrót objaśniony przy pierwszym użyciu |
| Nieobjaśnione skróty | FPS, TPS, MMO, MOBA, RTS, TBS, 4X, SFP, FM, API, OOP | Wykaz skrótów i rozwinięcie |
| Odwołania | „poniższa tabela” | „tabela 2.1” przez `\ref` |
| Cudzysłowy | “gra” | „gra” |
| Bardzo długie akapity | Najdłuższy ma 246 słów | Jeden akapit, jedna myśl |
| Sierotki | jednoliterowe „w”, „i”, „a”, „z” na końcu wiersza | `~`, bo ochrona w klasie działa tylko w LuaLaTeX, a projekt buduje się pdfLaTeX-em |

**Powtórzenia:** „oprócz tego” ×4, „przykładowo” ×3, „warto” ×3, „w konsekwencji” ×3, „w przypadku” ×3, „jak sama nazwa wskazuje” ×2.

### 10.2. Poprawki szczegółowe

| Oryginał | Propozycja | Powód |
|---|---|---|
| rozróżnienie, gdzie kończy się silnik, a zaczyna gra bywa trudne | ...a zaczyna gra, bywa trudne | Przecinek zamykający wtrącenie |
| Do tego silniki gier rzadko powstają w próżni. | Silniki rzadko powstają też w oderwaniu od konkretnego tytułu. | Styl |
| nie rozróżniali pojęć “gra” i “silnik” w ogóle, gdyż nie było możliwości myślenia na takim poziomie abstrakcji | Pierwsze gry nie miały wydzielonego silnika, ponieważ ich działanie wynikało wprost z układu elektronicznego. | Domysł o sposobie myślenia zastąpiony faktem |
| z wykorzystywaniem języka programowania | w języku programowania | Zwięzłość |
| wzrósł ponad okablowanie do kodu maszynowego | Logika gry przeniosła się z połączeń elektrycznych do kodu maszynowego. | Niezręczna przenośnia |
| Ilość pamięci RAM | Rozmiar pamięci RAM | „Ilość” dla niepoliczalnych, a tu i tak chodzi o rozmiar |
| swoim własnym, unikalnym silnikiem | gra nie miała wydzielonego silnika | Pleonazm i sprzeczność |
| Ów wzrost spowodował monumentalne i stabilne zwiększanie się mocy | Przełożyło się to na szybki wzrost mocy obliczeniowej | Styl |
| wpierw | najpierw | Archaizm |
| nie mających wpływu | niemających wpływu | Pisownia „nie” z imiesłowem |
| tj: | to znaczy: albo sam dwukropek | Interpunkcja |
| et al.REF BISHOP | Bishop i in. [5] | Brak spacji, styl |
| Ten gatunkowy rozstrzał | To zróżnicowanie gatunków | Potoczne |
| Problem ten definiuje Jason Gregory | Gregory ujmuje ten problem, określając silnik jako... | Definiowane jest pojęcie silnika, nie problem |
| z jednej strony elastyczność z drugiej optymalizacja | z jednej strony elastyczność, z drugiej optymalizacja | Przecinek |
| nie sposób jest zbadać | nie sposób zbadać | Zbędne „jest” |
| z założeniem średnio 40 ruchów per rozgrywka | przy założeniu, że partia trwa około 40 ruchów | Styl i ścisłość |
| Jest to liczba absolutnie przekraczająca możliwości | Wyklucza to pełne przeszukanie także na współczesnym sprzęcie. | Liczba niczego nie przekracza |
| ok 350 | ok. 350 | Kropka |
| każda z nich mająca wiele różnych wyników | a każda z nich ma wiele możliwych wyników | Składnia |
| musi spełniać paradygmat Forward Model | musi udostępniać interfejs Forward Model | To nie paradygmat |
| Mając dany Stan (S) i Akcję (A) funkcja oblicza | Dla stanu $s_t$ i akcji $a_t$ funkcja wyznacza | Imiesłów z innym podmiotem, zbędne wielkie litery |
| wiele algorytmów drzewiastych [...] wymagają | wiele algorytmów [...] wymaga | Liczebnik i czasownik |
| Mając na celu dostarczenie szybkie i wygodne FM API | Aby dostarczyć szybki i wygodny Forward Model | Błąd przypadka |
| algorytmach zbierania danych od użytkownika | obsłudze wejścia | Ścisłość |
| na cele treningu | do trenowania agentów | Styl |
| Perez-Liebana et al. (2020) skupia się | Perez-Liebana i in. skupiają się | Liczba mnoga |
| Real-Time Strategy GamesREF | Real-Time Strategy Games [14] | Spacja |
| (tick , a gracze | (ticku), a gracze | Niedomknięty nawias, przecinek |
| Środowiska badawcze [...] z powodzeniem realizują [...], ich implementacja opiera się | Choć środowiska badawcze [...] realizują [...], ich implementacja opiera się | Zdania bez spójnika |
| zarządzanie pamięci operate jest o garbage collector | zarządzaniem pamięcią zajmuje się mechanizm odśmiecania | Błąd i termin |
| dziel się na | dzieli się na | Literówka |
| oznaczałby | oznaczałaby | Podmiotem jest „separacja” |
| wymagają częstego duplikowania obecnego stanu gry na poddrzewa | wymaga częstego rozgałęziania stanu gry | Ścisłość |

### 10.3. Terminy do ustalenia

| Pojęcie | Propozycja |
|---|---|
| *Forward Model* | „Forward Model (FM)” z objaśnieniem jako „model przejść” |
| *cache miss / hit* | „brak trafienia / trafienie w pamięci podręcznej” |
| *garbage collector* | „mechanizm odśmiecania pamięci” |
| *tick* | „cykl gry” |
| *branching factor* | „współczynnik rozgałęzienia” |
| *rollout / playout* | „symulacja rozgrywki” |
| *Data-Oriented Design* | „projektowanie zorientowane na dane” |
| *copy-on-write* | „kopiowanie przy zapisie” |

---

## 11. Przykładowe przeredagowania

To wzorce, nie gotowy tekst do wklejenia. Znacznik `[ref]` oznacza miejsce cytowania. Wszystkie fakty pochodzą z rozdziału 7.

### 11.1. Otwarcie

> Zrozumienie współczesnej architektury silników gier wymaga w pierwszej kolejności określenia granicy między silnikiem a logiką gry. Gregory zauważa, że w praktyce trudno wskazać, gdzie kończy się silnik, a zaczyna gra [ref]. Silnik rzadko powstaje też w oderwaniu od konkretnego tytułu: *Unreal Engine 1* (1998) tworzono z myślą o grze *Unreal* studia Epic Games [ref].

### 11.2. Złożoność

> Pełne przeszukanie drzewa gry jest możliwe tylko w najprostszych grach. Shannon oszacował, że w typowej pozycji szachowej dostępnych jest około 30 legalnych ruchów, co daje około $10^3$ wariantów na parę posunięć. Przy partii trwającej 40 takich par otrzymujemy około $10^{120}$ wariantów [ref]. Maszyna analizująca jeden wariant na mikrosekundę potrzebowałaby na wybór pierwszego ruchu ponad $10^{90}$ lat [ref]. Praktyczne algorytmy nie przeglądają więc całego drzewa: ograniczają głębokość przeszukiwania i korzystają z funkcji oceny albo próbkują przestrzeń wariantów.

### 11.3. Forward Model

> Algorytmy planowania statystycznego (ang. *Statistical Forward Planning*, SFP), takie jak MCTS, oceniają ruchy, symulując ich skutki. Potrzebują do tego modelu przejść (ang. *Forward Model*, FM), czyli funkcji, która dla stanu $s_t$ i akcji $a_t$ wyznacza stan następny: $s_{t+1} = f(s_t, a_t)$. Jeśli reguły gry zawierają element losowy, wynik zależy także od ziarna generatora liczb pseudolosowych $\xi$, czyli $s_{t+1} = f(s_t, a_t, \xi)$. Przy ustalonym ziarnie przejście pozostaje powtarzalne. Autorzy frameworka Stratega ujmują to obrazowo: stany są węzłami drzewa gry, a model przejść jego krawędziami [ref]. Liczba symulacji możliwych w ramach budżetu czasu zależy bezpośrednio od kosztu tych operacji, dlatego koszt ten należy traktować jako jeden z głównych parametrów silnika.

### 11.4. Dane ciągłe

> Stan zapisany w jednym, ciągłym bloku pamięci, bez wskaźników, można skopiować pojedynczą operacją kopiowania bloku. Taka kopia nie wymaga przydziału pamięci dla poszczególnych obiektów ani odtwarzania powiązań między nimi, a dane położone obok siebie lepiej wykorzystują pamięć podręczną procesora [ref]. Koszt kopii nadal rośnie jednak wraz z rozmiarem stanu. Gdy stan jest duży, a ruch zmienia niewielką jego część, korzystniejsze może być współdzielenie niezmienionych danych albo cofanie ruchów [ref]. Tego wyboru nie da się rozstrzygnąć bez pomiarów, które przedstawiono w rozdziale 8.

### 11.5. microRTS

> microRTS to prosta gra czasu rzeczywistego napisana w Javie na potrzeby badań nad sztuczną inteligencją [ref]. W konfiguracji domyślnej jest deterministyczna i w pełni obserwowalna. Gracze wydają rozkazy równocześnie, a rozpoczętej akcji jednostki nie można przerwać [ref]. Ontañón wykorzystał to środowisko do zbadania problemu kombinatorycznej przestrzeni akcji: już 10 jednostek z 5 akcjami każda daje współczynnik rozgałęzienia $5^{10}$, czyli około 10 milionów, a na mapie *FullGame8x8* sięgał on 623 700 [ref]. Wcześniejsze metody przeszukiwania upraszczały ten problem przez abstrakcję. Ontañón zaproponował natomiast próbkowanie *Naïve Sampling* i oparty na nim algorytm NaïveMCTS, który przy dużym rozgałęzieniu osiąga znacząco lepsze wyniki niż pozostałe badane metody [ref]. Stan jest kopiowany metodą `GameState.clone()`, a pomocnicza metoda `cloneIssue()` współdzieli część danych między kopiami [ref].

### 11.6. VCMI z dokończeniem urwanego wątku

> VCMI to otwartoźródłowy silnik napisany od podstaw w C++, który odtwarza mechanikę *Heroes of Might and Magic III*, korzystając z oryginalnych zasobów gry [ref]. Twórcy dążą do możliwie wiernego odwzorowania reguł, ale chcą też usunąć ograniczenia oryginału i ułatwić modyfikacje [ref]. Silnik dzieli się na serwer, który nadzoruje stan gry, oraz klientów. W grze jednoosobowej oba elementy działają obecnie w jednym procesie, bez warstwy sieciowej [ref].
>
> VCMI nie udostępnia ogólnego modelu przejść. Najbliższym odpowiednikiem jest klasa `HypotheticBattle`, której używa `BattleAI`: nakładka na stan bitwy pozwala ocenić skutek akcji bez zmiany stanu właściwego [ref]. Rozwiązanie to służy jednak wewnętrznym potrzebom tego agenta. Trudność tę potwierdza projekt vcmi-gym, służący do trenowania agentów bitew na VCMI: jego integracja wymagała znacznej ingerencji w silnik. Jak zauważa autor projektu, bitew VCMI nie zaprojektowano z myślą o cofaniu, a wyuczony model świata okazał się niewystarczający dla MCTS, który wymaga znacznie wierniejszego symulatora [ref].

### 11.7. Zakres przeglądu

> Ponieważ celem pracy jest szybki i wygodny dla agentów AI Forward Model dla gry typu TBS (ang. *Turn-Based Strategy*), w przeglądzie pominięto potoki renderowania, synchronizację sieciową i obsługę wejścia. Skupiono się na tym, jak analizowane silniki reprezentują i kopiują stan gry oraz jak łatwo można wykorzystać je do trenowania i uruchamiania agentów.

---

## 12. Uwagi do kopii w `main.tex`

Treść sprawdzono automatycznym porównaniem z plikiem PDF. **Jedyne różnice to zmiany zamierzone:** usunięty spis treści z PDF, przeniesiona tabela, potęgi zapisane w trybie matematycznym, pominięty znacznik `@TODO`. Literówek nie poprawiano celowo.

**Rzeczy do zmiany, wynikające z tej kopii:**

- Podtytuły są moją propozycją, a nie częścią Twojego tekstu. Rozdział 5 proponuje lepszy podział.
- Podpis tabeli „Ilość pamięci RAM wybranych urządzeń” to moje sformułowanie i powtarza błąd „ilość”. Lepiej: „Rozmiar pamięci RAM wybranych urządzeń”.
- W tabeli nadal są błędne dane z PDF (pozycje 1 i 2 w rozdziale 7).
- Kursywa dla nazw gier jest ujednolicona bardziej niż w PDF. W pracy naukowej to dobry wybór, ale nazwy programów, jak VCMI, można zostawić prostym krojem. To kwestia do ustalenia raz.
- `\section{Cel pracy}` stoi między rozdziałami, jak w spisie z PDF. Zgodnie z rozdziałem 5 powinien trafić do wstępu.
- Tytuł, instytut, kierunek, numer albumu, promotor, streszczenia i słowa kluczowe to nadal zaślepki.

---

## 13. Plan działania

**Etap 1: decyzje**

- [ ] Rozmowa z promotorem: temat, autorstwo, wykorzystanie projektu ZPR, oczekiwana objętość.
- [ ] Uzgodnienie granicy pracy z Tomaszem Naszkowskim.
- [ ] Wybór wariantu A lub B.
- [ ] Zapisanie celu, pytań i zakresu.

**Etap 2: projekt**

- [ ] Scalenie gałęzi AI z `main` i dostosowanie do `BattleCommand`.
- [ ] Usunięcie `is_minimax_`.
- [ ] Dodanie benchmarków i **pomiar stanu obecnego przed przebudową**.
- [ ] Dokończenie symulacji w MCTS.
- [ ] Implementacja nowego stanu i porównanie.

**Etap 3: tekst**

- [ ] Poprawienie błędów z rozdziału 7.
- [ ] Założenie `bibliografia.bib` i usunięcie znaczników `REF`.
- [ ] Sprawdzenie w bibliotece: Gregory, pełny tekst Tribes, Wolf, Kent.
- [ ] Przebudowa układu według rozdziału 5.
- [ ] Dopisanie DOD, ściany pamięci, algorytmów i Stratega.
- [ ] Napisanie rozdziałów 5–8 równolegle z kodem.
- [ ] Na końcu: wstęp, podsumowanie, streszczenia.

**Etap 4: oddanie**

- [ ] Czyszczenie repozytorium, licencja, usunięcie zasobów chronionych z załączników.
- [ ] Kontrola stylu z rozdziału 10.
- [ ] Sprawdzenie odpowiedzi na pytania z punktu 2.4.

---

## 14. Źródła użyte w recenzji

**Odczytane:**

- [Shannon, *Programming a Computer for Playing Chess* (transkrypcja)](https://www.pi.infn.it/~carosi/chess/shannon.txt)
- [Bishop i in., *Designing a PC Game Engine*](https://www.cs.princeton.edu/courses/archive/spring2001/cs598b/papers/bishop98.pdf)
- [Ontañón, *The Combinatorial Multi-Armed Bandit Problem...*](https://cdn.aaai.org/ojs/12681/12681-52-16198-1-2-20201228.pdf)
- [Perez-Liebana i in., *The Design Of "Stratega"*](https://arxiv.org/abs/2009.05643)
- [Vinyals i in., *StarCraft II: A New Challenge for Reinforcement Learning*](https://arxiv.org/abs/1708.04782)
- [Patent US 3 793 483](https://patents.google.com/patent/US3793483A/en) (około 60% tekstu)
- [Gordon, tekst o portowaniu z MFC do GTK+](https://www.icculus.org/~icculus/writing/linux.com/mfc_to_gtk.txt)
- [Repozytorium Tribes](https://github.com/GAIGResearch/Tribes) i [strona artykułu](https://gaigresearch.github.io/2020/06/15/perez2020tribes/) (tylko abstrakt)
- [Repozytorium microRTS](https://github.com/Farama-Foundation/MicroRTS) i [definicja gry](https://github.com/Farama-Foundation/MicroRTS/wiki/Game-Definition)
- VCMI: [repozytorium](https://github.com/vcmi/vcmi), [FAQ](https://vcmi.eu/faq/), [AI](https://vcmi.eu/developers/AI/), [struktura kodu](https://vcmi.eu/developers/Code_Structure/), [sieć](https://vcmi.eu/developers/Networking/), [lista zmian](https://vcmi.eu/ChangeLog)
- [Opis projektu vcmi-gym](https://smanolloff.github.io/projects/vcmi-gym)

**Tylko pośrednio (wyszukiwarka, abstrakty, opisy):**

- [Atari 800XL/600XL](https://en.wikipedia.org/wiki/Atari_800XL), [Computing History: Atari 600XL](https://www.computinghistory.org.uk/det/64076/Atari-600XL/)
- [Electronic Design o sprzęcie PAC-MAN-a](https://www.electronicdesign.com/technologies/embedded/article/55292209/electronic-design-pac-man-the-revolutionary-video-game-turns-45), [opis płyty Namco Pac-Man](https://obscure.nesdev.org/wiki/Namco_Pac-Man)
- [Atarimania: PAC-MAN na Atari 8-bit](https://www.atarimania.com/games/atari-400-800-xl-xe-games-pac-man-3857), [AtariAge o kodzie źródłowym](https://forums.atariage.com/topic/349572-roklans-16k-pac-man-released-in-bin-and-car-form)
- [Tom's Hardware o PONG-u](https://www.tomshardware.com/video-games/retro-gaming/pong-debuted-on-this-day-in-1972-ataris-pioneering-cpu-less-video-arcade-games-creation-was-the-result-of-an-engineer-training-exercise), [The story of PONG](https://pong-story.com/atpong1.htm)
- [Wykład UMD streszczający rozdział 1 Gregory'ego](https://www.cs.umd.edu/class/spring2018/cmsc425/Lects/lect02-architecture.pdf)
- [Slajdy o Stratega](https://adockhorn.github.io/files/slides/AIIDE_SGW_2020.pdf)
- [Wilisowski, Dreżewski (2015)](https://rd.springer.com/chapter/10.1007/978-3-319-19728-9_7)
- [Pgx](https://arxiv.org/abs/2303.17503v3), [Madrona](https://madrona-engine.github.io/shacklett_siggraph23.pdf)
- [Wulf, McKee (1995)](https://libraopen.lib.virginia.edu/public_view/k35694375), [Browne i in. (2012)](https://ieeexplore.ieee.org/document/6145622/)
- Chessprogramming: [Copy-Make](https://chessprogramming.org/Copy-Make), [Unmake Move](https://chessprogramming.org/Unmake_Move)
- [Wykład o alokacji w JVM](https://esw.pages.fel.cvut.cz/lectures/ESW11_2026.pdf)
