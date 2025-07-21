# Medallion Architecture DAB Testing with Flight Radar Data

This project demonstrates **best practices for DAB (Databricks Asset Bundle) testing** using the **Medallion Architecture** with  flight radar fixture data.

## Architecture Overview

### Medallion Data Layers

```
BRONZE LAYER (Raw Data)
├── Raw flight radar data ingestion
├── Basic validation flags
└── Ingestion metadata

SILVER LAYER (Cleaned Data)  
├── Data cleaning & standardization
├── Business rule validation
├── Derived fields & quality scores
└── Ready for analysis

GOLD LAYER (Business Ready)
├── Flight summary analytics
├── Airport traffic metrics
├── Aircraft performance KPIs
└── Dashboard-ready aggregates
```

## Fixture Data

**File:** `fixtures/2025-07-20_17-50-14_hkg_flights.json`
- **5 flight records** from Hong Kong area
- **Real-time flight tracking data** with comprehensive attributes
- **Perfect for testing** data transformations and business logic

### Sample Data Structure
```json
{
  "data": [
    {
      "fr24_id": "3b567852",
      "flight": "HUA103", 
      "lat": 22.05259, "lon": 113.79069,
      "alt": 5000, "gspeed": 304,
      "dest_iata": "HKG", "orig_iata": "BUD",
      "timestamp": "2025-07-20T17:50:03Z"
    }
  ]
}
```

## Quick Start

### 1. Run the Complete Example
```bash
python examples/medallion_dab_testing_guide.py
```

### 2. Execute Tests
```bash
# Run all medallion architecture tests
pytest tests/main_test.py -v

# Run integration tests
pytest tests/test_dab_integration.py -v

# Run specific layer tests
pytest tests/main_test.py::TestBronzeLayer -v
pytest tests/main_test.py::TestSilverLayer -v  
pytest tests/main_test.py::TestGoldLayer -v
```

### 3. Deploy with DAB
```bash
# Deploy to development environment
databricks bundle deploy --target dev

# Run the DLT pipeline
databricks workspace run-job --job-id <job_id>
```

## DAB Testing Patterns

### Pattern 1: Layer-by-Layer Testing
```python
def test_medallion_pipeline_layers():
    pipeline_results = process_medallion_pipeline(spark)
    
    # Bronze Layer Testing
    assert pipeline_results["bronze"].count() == 5
    assert "is_valid_record" in pipeline_results["bronze"].columns
    
    # Silver Layer Testing  
    assert "data_quality_score" in pipeline_results["silver"].columns
    assert pipeline_results["silver"].filter("data_quality_score >= 0.8").count() > 0
    
    # Gold Layer Testing
    assert pipeline_results["gold_flight_summary"].count() > 0
```

### Pattern 2: Data Lineage Testing
```python
def test_data_lineage():
    # Track specific flight through pipeline
    test_flight_id = "3b567852"  # From fixture
    
    bronze_count = bronze_df.filter(col("fr24_id") == test_flight_id).count()
    silver_count = silver_df.filter(col("fr24_id") == test_flight_id).count()
    
    assert bronze_count == 1
    assert silver_count == 1  # Should flow through pipeline
```

### Pattern 3: Business Rule Validation
```python
def test_business_rules():
    silver_df = get_silver_flights()
    
    # Aviation business rules
    valid_coords = silver_df.filter(
        col("latitude").between(10.0, 35.0) &
        col("longitude").between(100.0, 140.0)
    ).count()
    
    assert valid_coords == silver_df.count()
```

### Pattern 4: Data Quality Monitoring
```python
def test_data_quality_metrics():
    silver_df = get_silver_flights()
    
    avg_quality = silver_df.agg({"data_quality_score": "avg"}).collect()[0]
    assert avg_quality["avg(data_quality_score)"] >= 0.8
```

## Project Structure

```
flightradar_databricks/
├── fixtures/
│   └── 2025-07-20_17-50-14_hkg_flights.json    # Test data
├── src/
│   ├── dlt_pipeline.ipynb                   # Medallion DLT pipeline  
│   └── flightradar_databricks/
│       └── main.py                             # Medallion functions
├── tests/
│   ├── main_test.py                            # Layer-by-layer tests
│   ├── test_dab_integration.py                 # End-to-end tests
│   └── test_utils.py                          # Testing utilities
├── examples/
│   └── medallion_dab_testing_guide.py         # Complete example
└── resources/
    ├── flightradar_databricks.pipeline.yml    # DLT config
    └── flightradar_databricks.job.yml         # Job config
```

## Key Functions

### Bronze Layer
```python
# Raw data ingestion with metadata
bronze_df = create_bronze_flights(spark, json_path)
validated_df = validate_bronze_data(bronze_df)
```

### Silver Layer  
```python
# Clean and standardize data
silver_df = create_silver_flights(spark, bronze_df)
clean_df = apply_business_rules_silver(silver_df)
```

### Gold Layer
```python
# Business analytics
flight_summary = create_gold_flight_summary(spark, silver_df)
airport_metrics = create_gold_airport_metrics(spark, silver_df)
aircraft_performance = create_gold_aircraft_performance(spark, silver_df)
```

### Complete Pipeline
```python
# Execute full medallion pipeline
pipeline_results = process_medallion_pipeline(spark)
```

## Data Quality Features

### Bronze Layer Validation
- Record validity flags (`is_valid_record`)
- Ingestion timestamps
- Source file tracking

### Silver Layer Quality
- Data quality scoring (0.0 - 1.0)
- Coordinate validation
- Aviation-specific business rules
- Derived fields (flight phases, categories)

### Gold Layer Metrics
- Aggregated KPIs
- Airport traffic analysis
- Aircraft performance metrics
- Real-time dashboard data

## DLT Pipeline Features

The `src/dlt_pipeline.ipynb` implements:

### Bronze Tables
- `bronze_flights`: Raw data with validation

### Silver Tables  
- `silver_flights`: Main cleaned dataset
- `silver_hkg_arrivals`: Hong Kong arrivals
- `silver_critical_phases`: Approach/takeoff flights

### Gold Tables
- `gold_flight_summary`: Business analytics
- `gold_airport_metrics`: Traffic metrics
- `gold_aircraft_performance`: Performance KPIs
- `gold_operational_dashboard`: Real-time metrics

### Data Quality Expectations
```python
@dlt.expect_all_or_fail({
    "valid_coordinates": "latitude BETWEEN -90 AND 90 AND longitude BETWEEN -180 AND 180",
    "valid_altitude": "altitude_ft >= 0 AND altitude_ft <= 50000", 
    "quality_threshold": "data_quality_score >= 0.8"
})
```

## Testing Strategy

### Unit Tests
- Individual layer functionality
- Data transformation accuracy
- Schema validation

### Integration Tests  
- End-to-end pipeline flow
- Cross-layer data consistency
- Performance benchmarks

### Data Quality Tests
- Business rule compliance
- Data drift detection
- Completeness validation

### DAB Environment Tests
- Dev/staging/prod consistency
- Deployment validation
- Pipeline orchestration

## Running the Tests

### Development Setup
```bash
# Create virtual environment (as you prefer)
python -m venv venv
source venv/bin/activate  # Windows: .\venv\Scripts\activate

# Install dependencies
pip install -e ".[dev]"
```

### Test Execution
```bash
# Quick test - basic functionality
python src/flightradar_databricks/main.py

# Comprehensive test suite
python examples/medallion_dab_testing_guide.py

# Unit tests
pytest tests/main_test.py -v

# Integration tests
pytest tests/test_dab_integration.py -v

# All tests
pytest tests/ -v
```

## Benefits of This Approach

### **Medallion Architecture Benefits**
- **Data Lineage**: Clear progression from raw to business-ready
- **Quality Control**: Systematic validation at each layer  
- **Flexibility**: Easy to modify business logic without affecting raw data
- **Debugging**: Simple to trace issues back through layers

### **DAB Testing Benefits**
- **Consistency**: Same tests across dev/staging/prod environments
- **Reliability**: Fixture data ensures predictable test results
- **Coverage**: Tests all pipeline stages and data quality rules
- **Automation**: Integrates with CI/CD for automated validation

### **Flight Radar Specific Benefits**  
- **Aviation Domain**: Business rules specific to flight tracking
- **Real Data**: Actual flight radar data structure and patterns
- **Geographic**: Hong Kong area data for location-based testing
- **Temporal**: Time-series data with timestamps and ETAs

## Next Steps

1. **Extend Fixture Data**: Add more flight scenarios (different airports, weather conditions, etc.)

2. **Streaming Integration**: Adapt for real-time flight data streams

3. **Advanced Analytics**: Add ML models for flight delay prediction, route optimization

4. **Data Governance**: Implement data catalog and lineage tracking

5. **Performance Optimization**: Add partitioning and caching strategies

6. **Monitoring**: Set up data quality dashboards and alerting

## Additional Resources

- [Databricks Asset Bundles Documentation](https://docs.databricks.com/dev-tools/bundles/index.html)
- [Delta Live Tables Guide](https://docs.databricks.com/delta-live-tables/index.html)
- [Medallion Architecture Best Practices](https://www.databricks.com/glossary/medallion-architecture)
- [Flight Radar Data Sources](https://flightradar24.com/)

---

**This project demonstrates comprehensive DAB testing using the Medallion Architecture with flight radar data!** 