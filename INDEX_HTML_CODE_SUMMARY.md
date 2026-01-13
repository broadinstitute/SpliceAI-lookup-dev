# SpliceAI Lookup - index.html Code Summary

## Overview

`index.html` is a single-page web application for looking up and visualizing splice prediction scores (SpliceAI, Pangolin) and other variant effect predictors (AlphaMissense, CADD, PrimateAI-3D, PromoterAI, etc.) for genomic variants. The app is built with jQuery, Semantic UI, and IGV.js.

---

## Architecture

### External Dependencies
- **jQuery 3.5.1** - DOM manipulation
- **Semantic UI 2.4.2** - UI components (forms, buttons, tables, modals, popups)
- **Underscore.js 1.12.0** - Utility functions (sorting)
- **@gmod/tabix 1.6.1** - Tabix file parsing for lookup tables
- **IGV.js** (custom build) - Genomic visualization
- **Google Analytics** - Usage tracking

### API Endpoints
```javascript
baseApiUrl = {
    "normalize": "https://liftover-xwkwwwxdwq-uc.a.run.app",
    "pangolin-37": "https://pangolin-37-xwkwwwxdwq-uc.a.run.app",
    "pangolin-38": "https://pangolin-38-xwkwwwxdwq-uc.a.run.app",
    "spliceai-37": "https://spliceai-37-xwkwwwxdwq-uc.a.run.app",
    "spliceai-38": "https://spliceai-38-xwkwwwxdwq-uc.a.run.app",
}
```

---

## Key Constants

| Constant | Purpose |
|----------|---------|
| `GENCODE_VERSION` | Current Gencode version ("v49") |
| `VARIANT_RE` | Regex for parsing variant strings |
| `TRANSCRIPT_PRIORITY` | Priority ordering: MS > MP > C > N |
| `nameMapForPredictorScores` | Display names for predictor scores |
| `predictorPointsToColor` | Color coding for pathogenicity points |
| `predictorPointsToLabel` | Labels for pathogenicity categories |
| `predictorScoreToPoints` | Score-to-points conversion functions |
| `colorMapForPredictorScores` | Background colors based on scores |
| `helpTextForPredictorScores` | Tooltip/help text generators |

---

## Key Functions

### Variant Normalization

#### `normalizeVariant(variant, genomeVersion)` (lines 897-1023)
Converts user input to standardized `{chrom}-{pos}-{ref}-{alt}` format.

- Uses regex (`VARIANT_RE`) for standard formats
- Falls back to Ensembl HGVS API for HGVS notation
- Returns object with `variant` and `consequence` fields

### API Communication

#### `makeRequest(url)` (lines 1025-1054)
Promise-based XMLHttpRequest wrapper for GET requests.

- Returns parsed JSON with added `ok` boolean
- Handles network errors

### Results Table Generation

#### `generateSplicingResultsTable(normalizedVariant, variantConsequence, tool, variant, genomeVersion, basicOrComprehensive, maxDistance, mask, showRefAltScoreColumns)` (lines 1149-1337)
Generates SpliceAI or Pangolin results tables.

- Calls appropriate API endpoint
- Sorts transcripts by priority (MANE Select > MANE Plus > Canonical)
- Creates rows with delta scores, positions, REF/ALT scores
- Handles insertion variants with detailed score tables
- Initializes modal dialogs for inserted base scores

#### `generateOtherPredictorsTable(normalizedVariant, variantConsequence, variant, genomeVersion)` (lines 1340-1604)
Generates table for additional predictors.

- Queries tabix-indexed lookup tables (PrimateAI-3D, PromoterAI, AlphaMissense)
- Queries gnomAD GraphQL API
- Queries myvariant.info API
- Displays scores with color coding and pathogenicity points

### Score Formatting

#### `formatScore(score)` (lines 836-842)
Formats numeric scores to 2 decimal places.

#### `getScoreStyle(score)` (lines 844-863)
Returns inline style string for score cells based on thresholds:
- >= 0.8: light red background (`#fccfb8`)
- >= 0.5: light yellow background (`#fff19d`)
- >= 0.2: light green background (`#cdffd7`)
- < 0.01: gray text

### Insertion Variant Handling

#### `considerInsertedBases(score, position, ref, alt, scoresForInsertedBases)` (lines 1056-1059)
Determines if insertion variant needs special handling.

#### `generateTableOfScoresForInsertedBases(modalId, score, position, ref, alt, scoresForInsertedBases)` (lines 1061-1117)
Creates detailed score table modal for inserted bases.

#### `updatePositionAccountingForInsertedBases(scoreKey, score, position, ref, alt, scoresForInsertedBases)` (lines 1119-1143)
Updates position display for insertion variants to show position within inserted sequence.

### IGV.js Visualization

#### `generateIgvConfig(spliceaiResponseJson, pangolinResponseJson, genomeVersion)` (lines 1692-1992)
Generates IGV.js configuration object with tracks:
- Refseq genes
- Variant position
- SpliceAI REF/ALT scores
- SpliceAI delta scores
- Pangolin delta scores
- Gencode genes
- MANE genes
- Precomputed SpliceAI scores (gain/loss at 0.5/0.2 thresholds)
- GTEx RNA-seq data (multiple tissues)
- Mappability and segmental duplications

#### `updateIgvBrowser(spliceaiResponseJson, pangolinResponseJson, genomeVersion)` (lines 1994-2006)
Creates or updates IGV browser instance.

### UI Management

#### `updateTranscriptButtons(category)` (lines 1606-1616)
Toggles between showing main transcript only vs all transcripts.

#### `updateVisualizationCheckboxes(readFromLocalStorage, genomeVersion)` (lines 1618-1689)
Manages IGV track checkbox state:
- Reads/writes to localStorage
- Disables hg38-only tracks for hg37

### Form Handling

#### `handleSubmit()` (lines 2009-2095)
Main form submission handler:
1. Reads form values
2. Shows loading state
3. Normalizes variant
4. Calls SpliceAI, Pangolin, and other predictor APIs in parallel
5. Updates results tables
6. Updates URL hash for sharing
7. Shows/hides error messages

#### `applyUrlSettingsToFormElements()` (lines 2097-2133)
Parses URL hash parameters and populates form fields.

#### `$.urlParam(name)` (lines 2152-2155)
jQuery extension to parse URL hash parameters.

---

## Data Flow

```
User Input -> handleSubmit() -> normalizeVariant() -> Ensembl API (if needed)
                                                   |
                                                   v
                              generateSplicingResultsTable() x2 (SpliceAI + Pangolin)
                                                   |
                                                   v
                              generateOtherPredictorsTable()
                                                   |
                                                   v
                              Display Results Tables
                                                   |
                                                   v (user clicks "Show")
                              updateIgvBrowser() -> generateIgvConfig()
```

---

## Score Thresholds and Points System

### SpliceAI/Pangolin Delta Score Thresholds
| Threshold | Color | Meaning |
|-----------|-------|---------|
| >= 0.8 | Red | High precision |
| >= 0.5 | Yellow | Recommended |
| >= 0.2 | Green | High recall |

### Pathogenicity Points (for missense variants)
Based on Bergquist et al. 2024 and Pejaver et al. 2022:

| Points | Classification |
|--------|----------------|
| +8 | Very Strong Pathogenic |
| +4 | Strong Pathogenic |
| +3 | Pathogenic |
| +2 | Moderate Pathogenic |
| +1 | Supporting Pathogenic |
| 0 | Indeterminate |
| -1 | Supporting Benign |
| -2 | Moderate Benign |
| -3 | Benign |
| -4 | Strong Benign |
| -8 | Very Strong Benign |

---

## Transcript Priority System

| Code | Priority | Description |
|------|----------|-------------|
| MS | 3 | MANE Select |
| MP | 2 | MANE Plus Clinical |
| C | 1 | Canonical |
| N | 0 | Non-canonical |

---

## CSS Classes

| Class | Purpose |
|-------|---------|
| `.main-transcript` | Highlighted background for main transcript |
| `.coding-transcript` | White background |
| `.non-coding-transcript` | Gray background |
| `.only-large-screen` | Hidden on small screens |
| `.no-print` | Hidden when printing |
| `.ref-score-column`, `.alt-score-column` | REF/ALT score columns (toggleable) |

---

## External Data Sources

### Tabix-indexed Lookup Tables
- PrimateAI-3D + PromoterAI: `gs://spliceai-lookup-reference-data/PrimateAI_and_PromoterAI_scores.hg{19,38}.*.tsv.gz`
- AlphaMissense: `gs://spliceai-lookup-reference-data/AlphaMissense_hg{19,38}.tsv.gz`

### External APIs
- Ensembl VEP API (HGVS conversion)
- gnomAD GraphQL API (in silico predictors)
- myvariant.info API (additional scores)

### IGV.js Reference Data
- Gencode annotations
- MANE transcripts
- GTEx RNA-seq (blood, fibroblasts, muscle, LCLs, brain cortex)
- Precomputed SpliceAI score tracks
- Mappability and segmental duplications

---

## State Management

### Global Variables
- `lastGenomeVersion` - Genome version from last submit
- `lastSpliceaiResponseJson` - SpliceAI API response
- `lastPangolinResponseJson` - Pangolin API response

### Local Storage
- `tracksToShow` - IGV track visibility preferences

### URL Hash Parameters
- `variant` - Input variant
- `hg` - Genome version (37/38)
- `bc` - Gencode set (basic/comprehensive)
- `distance` - Max distance
- `mask` - Masked scores (0/1)
- `ra` - Show REF/ALT columns (0/1)

---

## Bugs Fixed (January 2026)

| Line | Issue | Fix |
|------|-------|-----|
| 1504 | Typo "elegment" | Changed to "element" |
| 1177 | Missing `style=` attribute | Added `style=` to `<div display: inline-block>` |
| 338 | Invalid HTML `</br>` | Changed to `<br/>` |
| 186 | Extra closing `</span>` tag | Removed orphan tag |
| 1417 | Implicit global variable `query` | Added `const` declaration |

---

## Potential Improvements (Not Implemented)

1. **Race condition risk**: Rapid clicking of submit button could cause state inconsistencies with `last*` variables
2. **Error handling**: Tabix query failures in `generateOtherPredictorsTable` are logged but not displayed to users
3. **Async Promise executor**: `makeRequest()` uses `async` in Promise executor (anti-pattern, but works)
4. **Magic numbers**: Score thresholds could be extracted to named constants for maintainability
