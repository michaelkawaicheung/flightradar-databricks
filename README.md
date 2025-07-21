# FlightRadar24 Data Analytics with Databricks

A comprehensive data engineering project that collects and analyses flight data from the [FlightRadar24 API](https://fr24api.flightradar24.com/docs/getting-started) using Databricks for processing and analytics.

## Personal Motivation

As someone passionate about aviation - having explored A320 simulator introductory training, flown the C172, and always loving to watch planes - this project combines my fascination with flight with data engineering skills. There's something magical about understanding the patterns behind all those aircraft we see in the sky!

## Project Overview

This project focuses on understanding air traffic patterns, particularly around Hong Kong International Airport (VHHH), with potential expansion to analyse the world's busiest airports. The pipeline extracts real-time flight data, processes it through Databricks, and generates insights about:

- **Aircraft Types**: Analysis of the main types of planes operating in the region
- **Airlines**: Understanding which airlines dominate the routes
- **Passenger Flow**: Estimating passenger volume in and out of Hong Kong
- **Traffic Patterns**: Peak times, seasonal trends, and route analysis

## Target Airports

### Hong Kong International Airport (HKIA)
- **ICAO Code**: VHHH (4-letter code used by air traffic control and aviation professionals)
- **IATA Code**: HKG (3-letter code used by airlines and passengers)
- **Analysis Approach**: Using bounding box coordinates to define the geographical area around HKIA for flight tracking

### Airport and Airline Codes Explained
- **ICAO (International Civil Aviation Organisation)**: 4-letter codes used globally for aviation operations
- **IATA (International Air Transport Association)**: 3-letter codes used by the airline industry

**Key Airlines at HKIA:**
- Cathay Pacific: CX (IATA) / CPA (ICAO)
- Hong Kong Airlines: HX (IATA) / CRK (ICAO)
- And many international carriers

### Future Expansion
- Analysis of the world's busiest airports by passenger volume
- Comparative studies across different regions and time zones

## API Endpoint - Starting Point

### Live Flight Positions Full
We'll begin with the **Live Flight Positions Full** endpoint from FlightRadar24 API, which provides comprehensive real-time flight information.

**Endpoint:** `GET /api/live/flight-positions/full`

**What it returns:**
- Real-time aircraft flight movements
- Flight details (origin, destination)  
- Aircraft information (type, registration)
- Flight status and position data

**Hong Kong Bounding Box:**
For HKIA (VHHH/HKG) analysis, we'll use these coordinates to capture flights in Hong Kong airspace:
- **North:** 22.6° (covering approach/departure routes)
- **South:** 22.0° (capturing southern air traffic)
- **West:** 113.4° (western approach corridors)
- **East:** 114.4° (eastern flight paths)

**Example API Call:**
```bash
curl --location 'https://fr24api.flightradar24.com/api/live/flight-positions/full?bounds=22.6,22.0,113.4,114.4' \
--header 'Accept: application/json' \
--header 'Accept-Version: v1' \
--header 'Authorization: Bearer <your-token>'
```

**Bounding Box Format:** `bounds=north,south,west,east`

This captures all aircraft operating in and around Hong Kong International Airport's airspace, including arrivals, departures, and overflights.

## Data Pipeline Architecture

```
FlightRadar24 API → JSON Data → CSV/JSON Storage → Databricks Processing → Analytics & Insights
```

1. **Data Collection**: Periodic API calls to FlightRadar24 to collect flight data
2. **Data Storage**: Raw JSON data converted and stored as CSV/JSON files
3. **Data Processing**: Databricks pipelines process and transform the data
4. **Analytics**: Generate insights and visualizations from processed data

## Data Analysis Goals

### Phase 1: Hong Kong Airport Analysis
- Track flights in and out of VHHH
- Identify peak traffic hours and seasonal patterns
- Analyse aircraft types and airline distribution
- Estimate passenger flow based on aircraft capacity

### Phase 2: Global Airport Comparison
- Extend analysis to world's busiest airports
- Compare traffic patterns across different regions
- Identify global aviation trends

### Phase 3: Advanced Analytics
- Predictive modelling for flight delays
- Route optimisation insights
- Environmental impact analysis

## Getting Started

### Prerequisites

0. Install UV: https://docs.astral.sh/uv/getting-started/installation/

1. Install the Databricks CLI from https://docs.databricks.com/dev-tools/cli/databricks-cli.html

2. Get access to FlightRadar24 API: https://fr24api.flightradar24.com/docs/getting-started

### Setup

3. Set your email as an environment variable (to avoid hardcoding it in the repo):
    
    **macOS/Linux (bash/zsh):**
    ```bash
    export BUNDLE_VAR_user_email="your-email@example.com"
    ```

    **Windows PowerShell:**
    ```powershell
    $env:BUNDLE_VAR_user_email = "your-email@example.com"
    ```
    
    **Windows Command Prompt:**
    ```cmd
    set BUNDLE_VAR_user_email=your-email@example.com
    ```
    
    **To make it persistent, add to your shell profile:**
    - Windows: Add to PowerShell profile or set as system environment variable
    - macOS/Linux: Add the export command to `~/.bashrc`, `~/.zshrc`, or `~/.profile`

4. Authenticate to your Databricks workspace:
    ```
    $ databricks configure
    ```

5. Deploy a development environment:
    ```
    $ databricks bundle deploy --target dev
    ```
    
    This deploys the complete data pipeline including:
    - Data ingestion jobs for FlightRadar24 API
    - Processing workflows for flight data analysis
    - Analytics notebooks for insights generation

6. For production deployment:
   ```
   $ databricks bundle deploy --target prod
   ```

7. Run the data pipeline:
   ```
   $ databricks bundle run
   ```

### Development Environment

8. Optionally, install the Databricks extension for Visual Studio Code: 
   https://docs.databricks.com/dev-tools/vscode-ext.html
   
   This enables local development with Databricks Connect for testing and debugging.

## Project Structure

- `src/`: Main application code and Databricks notebooks
- `resources/`: Databricks job and pipeline configurations
- `tests/`: Unit tests for data processing logic
- `fixtures/`: Sample data for testing

## Documentation

For more information on Databricks asset bundles and CI/CD configuration:
https://docs.databricks.com/dev-tools/bundles/index.html

## API Reference

FlightRadar24 API Documentation: https://fr24api.flightradar24.com/docs/getting-started
