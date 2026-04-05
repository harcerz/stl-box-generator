# STL Box Generator - Design Spec

## Context

Generator pudełek/wypełnień do szuflad z eksportem do STL (do druku 3D). Aplikacja webowa w pojedynczym pliku HTML z interaktywnym podglądem 3D. Pudełka bez górnej pokrywy, z konfigurowalnymi wymiarami, zaokrągleniami narożników i opcjonalnymi otworami odpływowymi w podstawie.

## Technologia

- **Single-file HTML app** w `box-generator/index.html`
- **Three.js** (CDN) - renderer 3D + OrbitControls
- **three-bvh-csg** (CDN) - operacje boolowskie CSG (subtract)
- **Binary STL export** - ręczna serializacja geometrii Three.js do formatu STL

## Parametry użytkownika

| Parametr | Typ | Domyślnie | Opis |
|---|---|---|---|
| Szerokość (X) | number, mm | 100 | Wymiar zewnętrzny X |
| Głębokość (Y) | number, mm | 100 | Wymiar zewnętrzny Y |
| Wysokość (Z) | number, mm | 50 | Wymiar zewnętrzny Z |
| Grubość ścianek | number, mm | 2 | Grubość bocznych ścianek |
| Grubość podstawy | number, mm | 2 | Grubość dna pudełka |
| Promień zaokrąglenia | number, mm | 0 | Fillet narożników (0 = ostre) |
| Otwory odpływowe | boolean | false | Włącz/wyłącz siatkę otworów |
| Średnica otworu | number, mm | 5 | Średnica pojedynczego otworu |
| Odstęp otworów | number, mm | 15 | Odległość między centrami otworów w siatce |

## Architektura UI

```
+------------------+----------------------------------+
| Parametry        |                                  |
|                  |                                  |
| [Wymiary]        |       Podgląd 3D (Three.js)      |
| [Ścianki]        |       OrbitControls              |
| [Zaokrąglenia]   |       Grid helper                |
| [Otwory]         |                                  |
|                  |                                  |
| [Pobierz STL]    |                                  |
+------------------+----------------------------------+
```

- Panel lewy: formularz z parametrami, scrollowalny
- Panel prawy: canvas Three.js, responsywny
- Podgląd aktualizuje się na żywo przy zmianie parametrów (z debounce ~200ms)

## Generowanie geometrii (CSG)

### Krok 1: Zewnętrzny kształt
- Jeśli promień zaokrąglenia = 0: `BoxGeometry(width, height, depth)`
- Jeśli promień > 0: ExtrudeGeometry z zaokrąglonym prostokątem (RoundedRectShape) wyciągnięty na wysokość

### Krok 2: Wewnętrzna wnęka (do odęcia)
- Box pomniejszony o 2x grubość ścianek (X, Y)
- Wysokość = height - grubość_podstawy + 1mm (nadmiar na górze, żeby wyciąć otwartą górę)
- Pozycja Y przesunięta w górę o grubość_podstawy / 2
- Jeśli zaokrąglenie > 0: zaokrąglenie wewnętrzne = max(0, promień - grubość_ścianki)

### Krok 3: CSG subtract
- Wynik = zewnętrzny - wewnętrzny = pudełko ze ściankami i dnem

### Krok 4: Otwory odpływowe (opcjonalne)
- Generowanie siatki cylindrów (CylinderGeometry) w obszarze podstawy
- Siatka wycentrowana, z marginesem od ścianek (>= grubość ścianki + promień otworu)
- Każdy cylinder: średnica z parametru, wysokość = grubość_podstawy + 2mm (nadmiar)
- CSG subtract: wynik z kroku 3 - wszystkie cylindry

## Eksport STL (Binary)

Format binary STL:
1. Header: 80 bajtów (zerowane lub z opisem)
2. Liczba trójkątów: uint32
3. Dla każdego trójkąta: normal (3x float32) + 3 wierzchołki (3x float32 każdy) + attribute byte count (uint16)

Generowany z `BufferGeometry` Three.js - odczyt position buffer, obliczenie normali.
Pobieranie przez dynamiczny `<a>` z `Blob` URL.

## Walidacja parametrów

- Grubość ścianek < szerokość/2 i < głębokość/2
- Grubość podstawy < wysokość
- Promień zaokrąglenia <= min(szerokość, głębokość) / 2
- Średnica otworu < min(szerokość, głębokość) - 2 * grubość ścianki
- Wizualne podświetlenie nieprawidłowych wartości (czerwona ramka)

## Weryfikacja

1. Otworzyć `index.html` w przeglądarce
2. Sprawdzić podgląd 3D - obrót, zoom, domyślne pudełko
3. Zmienić wymiary - podgląd aktualizuje się na żywo
4. Ustawić zaokrąglenie > 0 - narożniki się zaokrąglają
5. Włączyć otwory - siatka otworów widoczna w podstawie
6. Pobrać STL - otworzyć w slicer (PrusaSlicer/Cura) i zweryfikować wymiary
