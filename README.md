# STL Box Generator

Browser-based generator for 3D-printable drawer organizer boxes. Set your parameters, preview in 3D, download STL.

**[Open the generator](https://harcerz.github.io/stl-box-generator/)** (requires GitHub Pages enabled)

Or just open `index.html` locally in any modern browser - no installation, no build step, no dependencies to install.

## Features

- **Parametric dimensions** - external width, depth, height in mm
- **Wall & base thickness** - independent control for sides and bottom
- **Rounded corners** - configurable fillet radius (0 = sharp edges)
- **Drainage holes** - optional grid of circular holes in the base (configurable diameter and spacing)
- **Live 3D preview** - interactive orbit controls (rotate, zoom, pan)
- **Binary STL export** - download ready for PrusaSlicer, Cura, or any slicer

## Usage

1. Open `index.html` in a browser
2. Adjust parameters in the left panel - preview updates in real time
3. Click **Pobierz STL** to download
4. Import the `.stl` file into your slicer and print

## Parameters

| Parameter | Default | Description |
|---|---|---|
| Szerokosc (X) | 100 mm | External width |
| Glebokosc (Y) | 100 mm | External depth |
| Wysokosc (Z) | 50 mm | External height |
| Grubosc scianek | 2 mm | Wall thickness |
| Grubosc podstawy | 2 mm | Base thickness |
| Promien zaokraglenia | 0 mm | Corner fillet radius |
| Otwory odplywowe | off | Enable drainage hole grid |
| Srednica otworu | 3 mm | Hole diameter |
| Odstep otworow | 14 mm | Center-to-center hole spacing |

## Tech

Single HTML file, zero build tooling. Libraries loaded from CDN:

- [Three.js](https://threejs.org/) r160 - 3D rendering, OrbitControls, STL export
- Geometry built with `Shape` + `ExtrudeGeometry` (no CSG dependency)

## License

MIT
