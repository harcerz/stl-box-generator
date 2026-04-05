# Drawer Planner - Design Spec

## Context

Rozszerzenie STL Box Generator o planer szuflad. Użytkownik rysuje kształt szuflady (wielokąt - nie zawsze prostokąt), rozmieszcza pudełka wewnątrz, konfiguruje każde osobno, eksportuje jako osobne pliki STL. Konfiguracja szuflady zapisywana/odczytywana z pliku JSON.

## Architektura

Dwa widoki w jednym pliku `index.html`:
- **Widok Planner (2D)** - rysowanie kształtu szuflady, rozmieszczanie pudełek, eksport
- **Widok Box Editor (3D)** - istniejący edytor pudełka, otwarty z kontekstem wybranego pudełka

Przełączanie widoków przez przyciski w nagłówku.

## Widok Planner (2D)

### Canvas 2D

Rysowany na elemencie `<canvas>` z ręcznym renderingiem (nie Three.js).

**Nawigacja:**
- Scroll = zoom (ze skalowaniem siatki)
- Środkowy przycisk myszy / Shift+drag = przesuwanie (pan)
- Siatka milimetrowa, gęstość zależna od zoomu (1mm/5mm/10mm)

**Tryb rysowania szuflady:**
1. Użytkownik klika punkty na canvas, tworząc wierzchołki wielokąta
2. Snap to grid (konfigurowalne: 1mm / 5mm / 10mm)
3. Kliknięcie blisko pierwszego punktu zamyka wielokąt
4. Po zamknięciu: kształt szuflady jest zdefiniowany
5. Edycja: przeciąganie wierzchołków, dodawanie/usuwanie punktów (prawy klik)

**Wyświetlanie:**
- Kształt szuflady: szary obrys, jasne wypełnienie
- Pudełka: kolorowe prostokąty z numerami/etykietami
- Zaznaczone pudełko: pogrubiony obrys, podświetlenie
- Wymiary: linie wymiarowe przy zaznaczonym pudełku
- Luzy: widoczne jako cienkie przerwy między pudełkami

### Panel lewy - Kontrolki

**Sekcja "Szuflada":**
- Przycisk "Rysuj kształt" (aktywuje tryb rysowania wielokąta)
- Przycisk "Edytuj kształt" (tryb edycji wierzchołków)
- Przycisk "Wyczyść" (reset kształtu)
- Wyświetlanie wymiarów bounding box szuflady

**Sekcja "Pudełka":**
- Przycisk "Auto-podział" → pola: kolumny x wiersze → generuje siatkę pudełek wypełniającą bounding box szuflady (obcinając te, które wychodzą poza kształt)
- Przycisk "Dodaj pudełko" (ręczne rysowanie prostokąta wewnątrz szuflady)
- Lista pudełek z miniaturami

**Sekcja "Globalne parametry":**
- Wysokość pudełek (domyślna dla nowych): 50mm
- Grubość ścianek: 2mm
- Grubość podstawy: 2mm
- Luz między pudełkami: 0.3mm (0 = wyłączony)

**Sekcja "Wybrane pudełko" (widoczna gdy pudełko zaznaczone):**
- Pozycja X, Y (edytowalna)
- Wymiary: szerokość, głębokość (edytowalne)
- Wysokość (nadpisuje globalną)
- Promień zaokrąglenia
- Otwory odpływowe: checkbox + średnica + odstęp
- Przycisk "Pobierz STL tego pudełka"
- Przycisk "Usuń pudełko"

**Sekcja "Eksport/Import":**
- Przycisk "Zapisz konfigurację" → pobiera plik JSON
- Przycisk "Wczytaj konfigurację" → input file, ładuje JSON
- Przycisk "Pobierz wszystkie STL (ZIP)" → generuje ZIP ze wszystkimi pudełkami

### Auto-podział

Algorytm:
1. Oblicz bounding box kształtu szuflady
2. Podziel bounding box na siatkę (kolumny x wiersze)
3. Uwzględnij luzy między pudełkami
4. Dla każdej komórki: sprawdź czy cała komórka mieści się w wielokącie szuflady
5. Jeśli nie mieści się: pomiń (nie twórz pudełka)
6. Wygeneruj pudełka z domyślnymi parametrami

Po auto-podziale użytkownik może:
- Przesuwać ścianki siatki (zmiana proporcji kolumn/wierszy)
- Łączyć sąsiednie komórki (merge) w jedno większe pudełko
- Usuwać pojedyncze pudełka
- Dodawać nowe pudełka ręcznie

## Widok Box Editor (3D)

Istniejący edytor pudełka z drobnymi zmianami:
- Przycisk "Wróć do plannera" w nagłówku
- Parametry wypełnione z danych wybranego pudełka
- Zmiany parametrów zapisują się z powrotem do modelu pudełka w plannerze
- Podgląd 3D jak dotychczas

## Format JSON konfiguracji

```json
{
  "version": 1,
  "drawer": {
    "points": [[0, 0], [400, 0], [400, 500], [0, 500]],
    "snapGrid": 5
  },
  "globalParams": {
    "boxHeight": 50,
    "wallThickness": 2,
    "baseThickness": 2,
    "gap": 0.3
  },
  "boxes": [
    {
      "id": "box-1",
      "x": 0,
      "y": 0,
      "width": 195,
      "depth": 245,
      "boxHeight": null,
      "cornerRadius": 2,
      "drainEnabled": false,
      "holeDiameter": 3,
      "holeSpacing": 14
    }
  ]
}
```

- `boxHeight: null` oznacza "użyj globalnej wartości"
- `version` do przyszłej kompatybilności
- Współrzędne w mm, punkt (0,0) = lewy dolny róg szuflady

## Eksport STL

### Pojedyncze pudełko
- Kliknięcie "Pobierz STL" przy wybranym pudełku
- Generowane przez istniejący `generateBoxGeometry()` z parametrami pudełka
- Wymiary pudełka = wymiary komórki minus luz (gap/2 z każdej strony)
- Plik: `box-{id}_{width}x{depth}x{height}.stl`

### Wszystkie pudełka (ZIP)
- Biblioteka JSZip z CDN
- Dla każdego pudełka: generuj STL, dodaj do ZIP
- Plik: `drawer_layout.zip`

## Zależności CDN (nowe)

- **JSZip** - generowanie archiwum ZIP w przeglądarce

## Walidacja

- Pudełka nie mogą wychodzić poza kształt szuflady
- Pudełka nie mogą na siebie nachodzić
- Wymiary pudełka muszą być > 2 * grubość ścianki
- Wizualne ostrzeżenia: pudełko poza szufladą = czerwony obrys, kolizja = pomarańczowy obrys

## Weryfikacja

1. Otworzyć `index.html` w przeglądarce → widok Planner
2. Narysować kształt szuflady (prostokąt 400x500mm)
3. Kliknąć "Auto-podział" 3x2 → 6 pudełek
4. Przesunąć ściankę siatki → zmiana proporcji
5. Kliknąć pudełko → panel z parametrami, zmienić zaokrąglenie
6. Przejść do Box Editor → podgląd 3D wybranego pudełka
7. Wrócić do plannera → zmiany zachowane
8. "Zapisz konfigurację" → pobrać JSON
9. Wyczyścić, "Wczytaj konfigurację" → wszystko odtworzone
10. "Pobierz wszystkie STL" → ZIP z osobnymi plikami
11. Otworzyć STL w slicerze → wymiary z uwzględnieniem luzów
