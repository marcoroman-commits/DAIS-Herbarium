# Herbarium

Herbarium specimens mapped by collection site · DAIS, Ca’ Foscari University of Venice

Herbarium is a lightweight web application for exploring the herbarium collection of the Department of Environmental Sciences, Informatics and Statistics (DAIS). Every specimen recorded in the collection spreadsheet is placed on an interactive map, and can be searched, filtered and inspected together with its label data and photographs.

The application is part of the tools developed by DESC – Data-centric Environmental Studies Centre.

## Architecture

The application has no server or database of its own: a Google Sheet holds the data, a Google Apps Script exposes it as a JSON API, and a single static HTML page published on GitHub Pages provides the interface.

```
Google Sheet (specimen records, "Luoghi" sheet for place corrections)
        │  read
        ▼
Google Apps Script  ──  Code.gs  ──  JSON API (public, read-only actions)
        │  fetch GET
        ▼
GitHub Pages  ──  index.html  ──  map interface (HTML, CSS, JavaScript, Leaflet)
```

Curators keep working in the spreadsheet as usual; the public page always shows the current state of the sheet when it is loaded or refreshed.

## Features

**Map**

- Each collection site is drawn as a circle whose area is proportional to the number of specimens collected there.
- Colours indicate how precisely each site is located:
  - GPS position recorded with the specimen;
  - locality recognised from the label;
  - centre of the province;
  - centre of the country.
- Three base maps: streets (OpenStreetMap), relief with contour lines (Esri, contours from zoom 11), and satellite imagery (Esri).
- Clicking the map shows the estimated terrain elevation at that point (Copernicus GLO-90 DEM, via Open-Meteo). This is an approximate value for the point, not a measurement taken on the specimen.
- Quick views of all points or of North-East Italy, and an area selection tool: drag on the map to restrict the results to a rectangle.

**Search and filters**

- Free-text search across species, genus, family, locality and collector.
- Filters for family, collector (legit), collection year range and location precision.
- Option to show only specimens with photographs.

**Panel**

- Most represented families, clickable to filter.
- List of collection sites with the number of specimens.
- For each site: the original label wording and the list of specimens.
- For each specimen, a label card with:
  - family, site, province or country, date;
  - collector, identifier (determinavit), mapping precision and notes;
  - linked photographs.
- Photo search in external databases (iNaturalist, GBIF, Wikimedia Commons). Images stay on their original sites and are shown with their credits and licences.

**General**

- Italian and English interface.
- Automatic light and dark theme.
- Responsive layout for desktop and mobile.

## Data model

The backend reads the first sheet whose header contains the herbarium columns, recognising them by name:

| Column | Content |
|---|---|
| ID | Numeric record identifier |
| Nr scatola | Storage box (not shown in the public version) |
| Famiglia, Genere, Specie, Genere e specie, Autore | Taxonomy |
| Sito di raccolta, Provincia, Stato | Collection locality as written on the label |
| Data | Collection date |
| Legit, Determinavit | Collector and identifier |
| Note | Free notes |
| Latitudine GPS, Longitudine GPS, Precisione GPS (m) | Optional GPS position |
| Foto | Linked photographs, one per line: `image URL \| credit \| source page` |
| Inserito da / il, Modificato da / il | Audit trail |

When a label has no GPS position, the locality text is matched against a built-in gazetteer of places in the Venetian lagoon and North-East Italy. Places that are not recognised fall back to the province or country centre. Curators can add or correct coordinates in the **Luoghi** sheet:

| Column | Content |
|---|---|
| A | Text to look for in the collection site |
| B, C | Latitude and longitude |
| D | Place name to display |

## API

The Apps Script web app answers GET requests:

| Request | Response |
|---|---|
| `?action=getData&lang=it` | Sheet header, rows, row numbers, place corrections and metadata |
| `?action=searchPhotos&scientificName=…` | Candidate images from iNaturalist, GBIF and Wikimedia Commons |

Requests accept `&callback=name` for JSONP.

Editing actions (adding and updating records, uploading and linking photos) exist in the backend but are restricted to the accounts listed in `EDITORS`. They are not available from the public GitHub page, which is read-only.

## Setup

1. **Spreadsheet.** Open the Apps Script project linked to the herbarium spreadsheet and paste `Code.gs`. Set `SPREADSHEET_ID` (when the script is not bound to the sheet), `EDITORS` and, if photos are uploaded, `PHOTO_FOLDER_ID`.
2. **Sheet tools.** Reload the spreadsheet and use the **Erbario** menu to:
   - add the GPS, photo and audit columns;
   - create the **Luoghi** sheet;
   - create the photo folder.
3. **Deploy the API.** Choose Deploy → New deployment → Web app, with *Execute as: Me* and *Who has access: Anyone*. Copy the URL ending in `/exec`.
4. **Configure the page.** In `index.html`, set `const API_URL = '…/exec';`.
5. **Publish.** Upload `index.html` to this repository and enable GitHub Pages: Settings → Pages → Deploy from a branch → `main` / `(root)`.

After any change to `Code.gs`, publish a new version of the existing deployment (Manage deployments → Edit → New version) so the URL stays the same.

## Third-party services

The page loads the following resources at run time; each remains subject to its own terms and licence:

- [Leaflet](https://leafletjs.com) 1.9.4, Esri Leaflet and Esri Leaflet Vector;
- OpenStreetMap tiles;
- Esri World Imagery, World Hillshade and contour vector tiles;
- Mapzen Terrain Tiles on AWS;
- Open-Meteo Elevation API;
- iNaturalist, GBIF and Wikimedia Commons APIs (photo search only);
- Google Fonts: Barlow, Spectral, IBM Plex Mono.

## Privacy

The page sets no cookies and uses no analytics. The browser stores only the chosen interface language (`localStorage`).

## Licence

© 2026 [copyright holder], DAIS – Ca’ Foscari University of Venice

This project is licensed under the [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International licence (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/). See the `LICENSE` file for the full legal text.

The following are not covered by this licence:

- the DESC and Ca’ Foscari logos;
- photographs retrieved from iNaturalist, GBIF and Wikimedia Commons, which remain under the licences stated by their authors;
- third-party libraries and map data, which remain under their own licences.

[![CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
