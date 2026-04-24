# 🐟 Fishy - Dark Vessel Monitor

FISHY is an interactive dark-vessel monitoring workspace built on top of Global Fishing Watch data. It helps analysts inspect fishing activity around EEZ boundaries, spot AIS transmission gaps, compare AIS tracks against SAR detections, and estimate where a vessel may have traveled while it was dark.

The app opens centered on the Galapagos-focused workflow and is designed for fast operational scanning rather than static reporting.

## What The Project Does

- Monitors fishing vessels operating inside or near selected EEZ regions
- Surfaces AIS gap events for individual vessels using Global Fishing Watch events data
- Overlays SAR detections to highlight contacts with and without AIS correlation
- Predicts likely vessel positions during dark periods with a Python LSTM service
- Falls back to a JavaScript dead-reckoning model if the ML service is unavailable
- Lets users narrow the picture by date window, EEZ, flag-state exclusions, and vessel selection

## Product Workflow

1. Select an EEZ to investigate.
2. Review the ranked vessel list near the selected boundary.
3. Open a vessel panel and inspect its AIS gap history.
4. Run a path prediction for a specific gap.
5. Compare the predicted path and uncertainty cloud against SAR detections on the map.

## Architecture

### Frontend

- Next.js 16 App Router
- React 19 + TypeScript
- Tailwind CSS 4
- Mapbox GL JS for the geospatial interface
- Three.js / `react-globe.gl` for the loading and globe visuals

### Data + APIs

- Global Fishing Watch 4Wings reports for vessel presence, fishing effort, and regional summaries
- Global Fishing Watch events API for AIS gap events
- Marine Regions geometry for EEZ boundary display
- SAR tile styling and detections proxied through the app

### Prediction Service

- FastAPI service in [`ml/`](C:\Users\OMAR\Desktop\MyProjects\fishy\fishy\ml\README.md)
- LSTM velocity model for post-gap position estimation
- Probability cloud generation for uncertainty visualization
- JavaScript fallback inside [`app/api/predict-path/route.ts`](C:\Users\OMAR\Desktop\MyProjects\fishy\fishy\app\api\predict-path\route.ts) when the ML server is offline

## Project Structure

```text
app/
  api/
    eez-boundary/              EEZ geometry and boundary helpers
    eez-regions/               searchable EEZ catalog
    eez-report/                regional reports from Global Fishing Watch
    predict-path/              ML-backed and fallback vessel path prediction
    sar-style/                 SAR layer style resolution
    tiles-sar/                 SAR tile proxy
    vessel-details/            vessel metadata lookup
    vessel-gaps/               AIS gap event lookup
    vessels-near-eez/          vessel discovery around selected EEZs
  components/
    FishingMap.tsx             main geospatial workspace
    VesselList.tsx             ranked vessel sidebar
    VesselPanel.tsx            vessel detail + gap inspection
    PredictionOverlay.tsx      predicted path and uncertainty cloud
    EEZSelector.tsx            region switching
    CountryFilter.tsx          flag-state exclusion filters
    TimelineSlider.tsx         date range controls
ml/
  prediction_server.py         FastAPI prediction service
  train_v3.py                  LSTM training entry point
  data/                        track fetch, preprocessing, gap detection
  features/                    feature engineering + normalization
  models/                      trained artifacts and baseline logic
```

## Local Development

### Prerequisites

- Node.js 20+
- npm
- Python 3.10+ for the ML server
- A Global Fishing Watch API token
- A Mapbox access token

### Environment Variables

Create a local `.env` file in the repo root:

```env
FISH_API=your_global_fishing_watch_token
NEXT_PUBLIC_MAPBOX_TOKEN=your_mapbox_token
ML_SERVER_URL=http://localhost:8000
```

### Run The Frontend

```bash
npm install
npm run dev
```

The app runs at [http://localhost:3000](http://localhost:3000).

### Run The ML Service

```bash
cd ml
pip install -r requirements.txt
python prediction_server.py
```

The prediction API runs at `http://localhost:8000`.

## Key Endpoints

- `POST /api/vessels-near-eez` discovers vessels operating in or near an EEZ
- `GET /api/vessel-gaps` returns AIS gap events for a vessel
- `POST /api/predict-path` generates a predicted path and uncertainty cloud
- `POST /api/eez-report` proxies regional report requests to Global Fishing Watch

## Deployment

Deployment notes for the split frontend + ML setup live in [`DEPLOYMENT.md`](C:\Users\OMAR\Desktop\MyProjects\fishy\fishy\DEPLOYMENT.md).

## Notes

- The default viewport and date window are currently tuned to the Galapagos monitoring scenario.
- Prediction quality depends on the availability of track context and the ML service.
- If the ML server is down, FISHY still works with a simpler baseline prediction model.
