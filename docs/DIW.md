# Dokumentacja inżynierii wymagań

**System automatycznej anonimizacji materiałów wideo i audio dla ReportersWatch Sp. z o.o.**

| | |
|---|---|
| Przedmiot | Inżynieria Oprogramowania, AGH WIEiT, Teleinformatyka, 2026/27 |
| Klient | ReportersWatch Sp. z o.o., Karol Wiśniewski (Kierownik Działu Produkcji Wideo) |
| Zespół | Joanna Żupnik, Marcel Kuźma, Arkadiusz Baran |
| Prowadzący | Andrzej Matiolański |
| Wersja dokumentu | 0.1 (robocza) |

## Spis treści

1. [Wstęp](#1-wstęp)
2. [Członkowie zespołu](#2-członkowie-zespołu)
3. [Pytania do klienta i uzyskane odpowiedzi](#3-pytania-do-klienta-i-uzyskane-odpowiedzi)
4. Słownik pojęć *(w przygotowaniu)*
5. Wymagania użytkownika *(w przygotowaniu)*
6. Wymagania systemowe *(w przygotowaniu)*
7. Ustalony format danych *(w przygotowaniu)*
8. Model systemu *(w przygotowaniu)*
9. Diagram sekwencji UML *(w przygotowaniu)*
10. Projekt architektury systemu *(w przygotowaniu)*
11. Kryteria akceptacji *(w przygotowaniu)*
12. Ryzyka projektowe *(w przygotowaniu)*
13. Ewolucja systemu *(w przygotowaniu)*
14. Sugerowany język i technologie implementacji *(w przygotowaniu)*

---

## 1. Wstęp

### 1.1. Cel dokumentu

Dokument opisuje wymagania wobec systemu automatycznej anonimizacji nagrań zamówionego przez firmę ReportersWatch Sp. z o.o. Stanowi podstawę do planowania prac zespołu, projektu architektury, implementacji oraz testów akceptacyjnych. Struktura dokumentu odpowiada typowym elementom dokumentacji wymagań opisanym przez Sommerville'a [1] oraz w normie ISO/IEC/IEEE 29148 [2].

### 1.2. Treść zamówienia klienta

Firma ReportersWatch zajmuje się produkcją i dystrybucją materiałów wideo: nagrań reporterskich, szkoleniowych oraz transmisji wydarzeń na żywo. W publikowanych materiałach coraz częściej pojawiają się osoby postronne i dane osobowe, a ręczna anonimizacja stała się zbyt czasochłonna i podatna na błędy.

Klient poszukuje systemu, który automatycznie usunie z nagrań informacje pozwalające zidentyfikować osoby:

- w obrazie: twarze, tablice rejestracyjne i dane widoczne na ekranie,
- w dźwięku: dane osobowe wypowiadane przez rozmówców oraz sam głos.

Dodatkowe oczekiwania klienta:

- anonimizacja ma być skuteczna także wobec prób odtworzenia tożsamości,
- w uzasadnionych przypadkach osoby uprawnione mogą w kontrolowany sposób odzyskać oryginalny materiał,
- system obsługuje również transmisje na żywo,
- ostateczną decyzję o publikacji podejmuje redaktor, dlatego musi on mieć możliwość przejrzenia i poprawienia wyników przed wydaniem materiału.

### 1.3. Zakres systemu

System jest aplikacją webową z interfejsem w języku polskim, działającą na serwerze w infrastrukturze klienta (założenie, rozdział 1.4). Użytkownik loguje się z uwierzytelnianiem dwuskładnikowym (2FA), wgrywa plik (MP4, MOV lub MP3) albo wskazuje transmisję na żywo i wybiera, co ma zostać zanonimizowane: twarze, tekst w kadrze lub głos. Następnie przegląda wynik, w razie potrzeby go poprawia i zatwierdza.

System:

- wykrywa i śledzi twarze, tablice rejestracyjne i tekst w kadrze, a następnie je maskuje,
- rozpoznaje mowę w języku polskim i angielskim, wykrywa wypowiadane dane osobowe i je usuwa,
- modyfikuje głos wszystkich osób poza reporterem,
- rozpoznaje reportera na podstawie zdjęć i nie anonimizuje go,
- przechowuje oryginały zaszyfrowane algorytmem AES-256 i umożliwia ich kontrolowane odtworzenie (pseudonimizacja odwracalna),
- usuwa dane po skonfigurowanym czasie retencji (od 5 s do 7 dni),
- prezentuje statystyki dla pojedynczego pliku oraz globalne.

Poza zakresem projektu są: montaż i edycja materiałów (cięcie, napisy, efekty), publikacja i dystrybucja materiałów, aplikacje mobilne, materiały dłuższe niż 30 minut, języki mowy inne niż polski i angielski oraz interfejs w języku innym niż polski.

### 1.4. Założenia przyjęte przy interpretacji ustaleń

Część ustaleń z klientem była zwięzła. Tam, gdzie wymagała doprecyzowania, zespół przyjął poniższe interpretacje. Pozycje oznaczone jako **do potwierdzenia** należy potwierdzić z klientem na kolejnym spotkaniu.

| Ustalenie z klientem | Przyjęta interpretacja |
|---|---|
| Format aplikacji: dowolny (web, standalone) | Aplikacja webowa uruchamiana w przeglądarce, a przetwarzanie i przechowywanie danych odbywa się na serwerze w infrastrukturze klienta (on-premise). Uzasadnienie: z systemu korzysta kilku redaktorów pracujących na wspólnych materiałach, wykrywanie twarzy i rozpoznawanie mowy wymagają wydajnej karty graficznej tylko na serwerze, a nie na każdym komputerze, a standard WCAG 2.1 dotyczy wprost treści webowych. Decyzja zespołu. |
| Wynik: plik(i), RTSP | Dla materiału z pliku wynikiem jest zanonimizowany plik w formacie wejściowym. System przyjmuje jeden lub wiele plików naraz (przetwarzanie wsadowe). Dla transmisji na żywo wynikiem jest zanonimizowany strumień RTSP. |
| Transmisja: opóźnienie do 15 s | Czas od odebrania fragmentu obrazu i dźwięku ze źródła do wysłania jego zanonimizowanej wersji w strumieniu wyjściowym nie przekracza 15 s. W trybie na żywo redaktor nie zatwierdza materiału przed emisją; może obserwować podgląd i przerwać transmisję. **Do potwierdzenia.** |
| Skuteczność: twarze 97%, tablice rejestracyjne 95%, tekst w kadrze 95%, WER 90% | Odsetek obiektów danego typu wykrytych i zamaskowanych na zestawie testowym. Dla mowy chodzi o skuteczność rozpoznawania co najmniej 90%, czyli wskaźnik WER (odsetek błędnie rozpoznanych słów) nie większy niż 10%. |
| Tożsamości wszystkich osób poza reporterem | Anonimizowane są wszystkie wykryte osoby z wyjątkiem reportera. |
| Reporter: identyfikacja za pomocą zdjęć | Przed przetwarzaniem użytkownik wgrywa zdjęcia referencyjne reportera (co najmniej 3). Twarz pasująca do zdjęć nie jest maskowana. |
| Modyfikacja głosu (poza reporterem) | Głos pozostałych mówców jest zmieniany tak, aby nie dało się go rozpoznać. W nagraniach bez obrazu (MP3) system dzieli nagranie na mówców, a reportera wskazuje redaktor. **Do potwierdzenia.** |
| Pseudonimizacja odwracalna | Materiał wynikowy jest zamaskowany nieodwracalnie: z opublikowanego pliku nie da się odtworzyć tożsamości. Odwracalność zapewnia przechowywany osobno, zaszyfrowany oryginał, do którego dostęp ma wyłącznie osoba uprawniona. Spełnia to definicję pseudonimizacji z RODO (art. 4 pkt 5) [3] i jednocześnie wymaganie klienta o odporności na próby odtworzenia tożsamości. |
| Szyfrowanie danych lokalnie (AES-256) | Wszystkie przechowywane dane (oryginały, wyniki, informacje o wykryciach, zdjęcia reportera) są zaszyfrowane algorytmem AES-256 na serwerze klienta. Dane nie są wysyłane do usług zewnętrznych, więc modele rozpoznawania obrazu i mowy działają lokalnie. **Do potwierdzenia.** |
| Retencja danych: konfiguracja od 5 s do 7 dni | Czas przechowywania ustawia administrator i dotyczy on wszystkich danych w systemie. Po jego upływie system trwale usuwa oryginały, wyniki, informacje o wykryciach i klucze szyfrujące. Konsekwencja: po upływie retencji odtworzenie oryginału nie jest możliwe. |
| Języki: polski i angielski | Języki mowy w nagraniach, w których system rozpoznaje wypowiedzi i wykrywa dane osobowe. |
| Interfejs: w języku polskim | Interfejs jest dostępny wyłącznie po polsku. |
| Konfiguracja: FACE, TXT, VOICE | Przed przetwarzaniem użytkownik zaznacza, co ma być anonimizowane: twarze (FACE), tekst w kadrze, w tym tablice rejestracyjne (TXT), głos i wypowiadane dane osobowe (VOICE). Kategorie można łączyć dowolnie. |
| Śledzenie obiektu do maskowania | Maska podąża za obiektem między klatkami. Obiekt pozostaje zamaskowany także w klatkach, w których detektor chwilowo go nie wykrył. |
| Statystyki: dla pliku i globalne | Statystyki pliku: liczba usuniętych słów (wypowiadanych danych osobowych), wydajność przetwarzania i ilość przetworzonych danych. Statystyki globalne sumują te wartości dla wszystkich plików. Wydajność to stosunek czasu trwania materiału do czasu przetwarzania (np. 2,0× oznacza minutę materiału przetworzoną w 30 s). |
| Logowanie: 2FA | Logowanie hasłem i jednorazowym kodem z aplikacji uwierzytelniającej (TOTP). |
| Interfejs GUI: WCAG 2.1 (AA) | Interfejs spełnia kryteria sukcesu poziomów A i AA wytycznych WCAG 2.1 [4]. |
| Formaty, długość i rozdzielczość | Wideo MP4 lub MOV z kodekiem H.264 lub H.265, audio MP3. Maksymalna długość materiału 30 min, rozdzielczość od 720p (1280×720) do 4K (3840×2160). Plik spoza tego zakresu jest odrzucany z czytelnym komunikatem. |
| Przegląd wyników przez redaktora (z zamówienia, nie omawiano na spotkaniu) | Przed eksportem redaktor ogląda wynik w podglądzie i może dodać, usunąć lub przesunąć maskę oraz zmienić decyzję o wyciszeniu fragmentu dźwięku. Eksport następuje dopiero po zatwierdzeniu. **Do potwierdzenia.** |

---

## 2. Członkowie zespołu

Zespół: Joanna Żupnik, Marcel Kuźma, Arkadiusz Baran.

### 2.1. Macierz kompetencji zespołu

Macierz obejmuje kompetencje kluczowe dla tego projektu. Skala: Posiada, Posiada (podstawy), Nie posiada. *Do uzupełnienia przez członków zespołu.*

| Kompetencje | Joanna Żupnik | Marcel Kuźma | Arkadiusz Baran |
|---|---|---|---|
| Programowanie w Pythonie | | | |
| Przetwarzanie obrazu i wideo (OpenCV, ffmpeg) | | | |
| Modele AI do wykrywania obiektów (twarze, tablice, tekst) | | | |
| Rozpoznawanie mowy i przetwarzanie dźwięku | | | |
| Transmisja strumieniowa (RTSP) | | | |
| Aplikacje webowe (frontend, backend) | | | |
| Bazy danych (SQL) | | | |
| Bezpieczeństwo i kryptografia (AES, 2FA) | | | |
| Dostępność interfejsu (WCAG) | | | |
| Znajomość UML | | | |
| Testowanie oprogramowania | | | |

### 2.2. Proponowany podział ról

| Rola | Zakres odpowiedzialności | Osoba |
|---|---|---|
| Obraz | Wykrywanie i śledzenie twarzy, tablic rejestracyjnych i tekstu, maskowanie, rozpoznawanie reportera | *do ustalenia* |
| Dźwięk | Rozpoznawanie mowy, wykrywanie wypowiadanych danych osobowych, modyfikacja głosu | *do ustalenia* |
| Aplikacja i bezpieczeństwo | Interfejs (WCAG), logowanie 2FA, szyfrowanie, retencja, transmisja na żywo, statystyki | *do ustalenia* |
| Testy i dokumentacja | Zestawy testowe, testy akceptacyjne, diagramy UML | *wspólnie* |

---

## 3. Pytania do klienta i uzyskane odpowiedzi

Zamówienie klienta nie wystarczało do rozpoczęcia prac. Nie określało wymaganej skuteczności anonimizacji, formatów i parametrów materiałów, platformy, sposobu logowania i przechowywania danych ani tego, czyja tożsamość ma zostać zachowana. Poniższe pytania sformułowano w celu uszczegółowienia wymagań funkcjonalnych i niefunkcjonalnych. Odpowiedzi zanotowano w trakcie dyskusji na zajęciach; ich interpretację opisano w rozdziale 1.4.

| Pytanie | Odpowiedź | Uwagi |
|---|---|---|
| Jakie informacje identyfikujące mają być usuwane? | Twarze, tekst w kadrze (w tym tablice rejestracyjne) i głos, czyli dane osobowe w rozumieniu RODO. | Kategorie: FACE, TXT, VOICE. |
| Czy użytkownik wybiera, co ma być anonimizowane? | Tak, konfiguracja FACE, TXT, VOICE. | Wybór przed przetwarzaniem. |
| Czyje tożsamości mają być ukryte? | Wszystkich osób poza reporterem. | |
| Jak system rozpoznaje reportera? | Na podstawie zdjęć. | Liczba zdjęć: założenie (1.4). |
| Czy głos ma być modyfikowany? | Tak, poza głosem reportera. | Nagrania audio: do potwierdzenia (1.4). |
| Jakiej skuteczności wymagacie? | Twarze 97%, tablice rejestracyjne 95%, tekst w kadrze 95%, rozpoznawanie mowy 90% (WER). | WER nie większy niż 10%. |
| Czy maska ma podążać za obiektem? | Tak, śledzenie obiektu do maskowania. | |
| Czy anonimizacja ma być odwracalna? | Tak, pseudonimizacja odwracalna. | Odwracalność przez zaszyfrowany oryginał (1.4). |
| Jak mają być przechowywane dane? | Lokalnie, szyfrowane AES-256. | |
| Jak długo przechowujemy dane? | Konfigurowalnie, od 5 s do 7 dni. | Dotyczy wszystkich danych. |
| Jakie formaty materiałów obsługujemy? | MP4, MOV (kodeki H.264, H.265), MP3. | |
| Jaka jest maksymalna długość materiału? | 30 min. | |
| Jaka rozdzielczość? | Od 720p do 4K. | |
| W jakich językach są wypowiedzi w nagraniach? | Polski i angielski. | |
| W jakim języku jest interfejs? | Polski. | |
| Jaka ma być postać wyniku? | Plik(i), a dla transmisji na żywo strumień RTSP. | |
| Czy system obsługuje transmisje na żywo i z jakim opóźnieniem? | Tak, opóźnienie do 15 s. | |
| Jaką formę ma mieć aplikacja? | Dowolną (web lub standalone). | Wybór zespołu: aplikacja webowa (1.4). |
| Jakie wymagania dotyczą interfejsu? | GUI zgodne z WCAG 2.1 na poziomie AA. | |
| Jak ma wyglądać logowanie? | Z uwierzytelnianiem dwuskładnikowym (2FA). | |
| Jakie statystyki ma pokazywać system? | Dla pliku i globalne: liczba usuniętych słów, wydajność, ilość przetworzonych danych. | Definicje w 1.4. |
| Czy wszystkie wymagania klienta są możliwe do spełnienia? | Tak, z zastrzeżeniami. | Patrz akapit poniżej. |

**Czy wszystkie wymagania klienta są możliwe do spełnienia?** TAK, z zastrzeżeniami, które należy przekazać klientowi:

1. Modele wykrywania obrazu i mowy nie działają bezbłędnie. Przy skuteczności 97% część twarzy nie zostanie wykryta automatycznie, dlatego w trybie plikowym kluczowy jest przegląd wyników przez redaktora przed publikacją.
2. W trybie na żywo nie ma przeglądu przed emisją, więc obiekty niewykryte przez system mogą trafić do widzów. Ryzyko należy zaakceptować albo ograniczyć tryb na żywo do wybranych zastosowań.
3. Przetwarzanie materiału 4K na żywo z opóźnieniem do 15 s zależy od sprzętu serwera (karta graficzna). Wymagania sprzętowe trzeba ustalić z klientem.
4. Pseudonimizacja odwracalna działa tylko w czasie retencji. Po jej upływie oryginał jest usuwany i nie da się go odtworzyć.

---

## Literatura

1. Sommerville I., *Software Engineering*, 10th edition, Pearson, 2016.
2. ISO/IEC/IEEE 29148:2018, *Systems and software engineering. Life cycle processes. Requirements engineering*.
3. Rozporządzenie Parlamentu Europejskiego i Rady (UE) 2016/679 z dnia 27 kwietnia 2016 r. (RODO), Dz.U. UE L 119 z 4.05.2016.
4. W3C, *Web Content Accessibility Guidelines (WCAG) 2.1*, W3C Recommendation, 2018.
