# Healthcare DLT Pipeline Project

## Overview

This project demonstrates a comprehensive **Delta Live Tables (DLT)** pipeline for healthcare data processing, implementing the **Bronze-Silver-Gold** medallion architecture. The pipeline processes patient admission data with real-time streaming capabilities, data quality enforcement, and multi-dimensional analytics.

## Architecture

### Medallion Architecture Implementation

```
Raw Data Sources → Bronze Layer → Silver Layer → Gold Layer
     ↓                ↓              ↓            ↓
Reference Data   Raw Ingestion   Data Enrichment  Analytics
Patient Data     + DQ Checks     + Joins         + Aggregations
```

### Data Flow

1. **Bronze Layer**: Raw data ingestion with data quality constraints
2. **Silver Layer**: Enriched data with business logic and joins
3. **Gold Layer**: Aggregated analytics for business intelligence

## Project Structure

```
Healthcare_DLT_Pipeline_Project/
├── notebook/
│   ├── healthcare_dlt_processing.ipynb    # Main DLT pipeline definition
│   └── feeding_raw_tables.ipynb           # Data ingestion utility
├── data/                                  # Sample data files
│   ├── diagnosis_mapping.csv
│   └── patients_daily_file_*.csv
├── imgs/                                  # Pipeline running screenshots
└── README.md                              # This file
```

## Data Sources

### 1. Diagnosis Mapping (Reference Data)
- **File**: `diagnosis_mapping.csv`
- **Purpose**: Maps diagnosis codes to human-readable descriptions
- **Schema**: `diagnosis_code`, `diagnosis_description`

### 2. Daily Patient Data (Streaming Data)
- **Files**: `patients_daily_file_*.csv`
- **Purpose**: Daily patient admission records
- **Schema**: `patient_id`, `name`, `age`, `gender`, `address`, `contact_number`, `admission_date`, `diagnosis_code`

## DLT Pipeline Components

### Bronze Layer Tables

#### 1. `diagnostic_mapping`
- **Type**: Live Table (Batch)
- **Purpose**: Reference data for diagnosis codes
- **Data Quality**:
  - `diag_code_not_null`: Ensures diagnosis codes are not null
  - `diag_desc_not_null`: Ensures diagnosis descriptions are not null
- **Violation Handling**: Drops rows with null values

#### 2. `daily_patients`
- **Type**: Streaming Table
- **Purpose**: Real-time ingestion of patient admission data
- **Data Quality**:
  - `pk_not_null`: Ensures patient_id is present
  - `required_fields`: Ensures all essential patient fields are populated
- **Violation Handling**: Drops rows with missing critical data

### Silver Layer Tables

#### 1. `processed_patient_data`
- **Type**: Streaming Table
- **Purpose**: Enriched patient data with diagnosis descriptions
- **Business Logic**:
  - LEFT JOIN with diagnosis mapping
  - COALESCE for unmapped diagnosis codes
- **Data Quality**:
  - `has_diagnosis`: Ensures diagnosis description is available
- **Violation Handling**: Drops rows without diagnosis descriptions

### Gold Layer Tables

#### 1. `patient_statistics_by_admission_date`
- **Type**: Live Table
- **Purpose**: Daily operational metrics
- **Metrics**: Patient counts, average age by date and diagnosis
- **Use Case**: Hospital capacity planning, trend analysis

#### 2. `patient_statistics_by_diagnosis`
- **Type**: Live Table
- **Purpose**: Medical insights by diagnosis
- **Metrics**: Patient counts, age statistics, gender distribution
- **Use Case**: Medical research, diagnosis prevalence analysis

#### 3. `patient_statistics_by_gender`
- **Type**: Live Table
- **Purpose**: Demographic analysis by gender
- **Metrics**: Patient counts, age statistics, diagnosis diversity
- **Use Case**: Population health studies, demographic insights

## Data Quality Framework

### Constraint Types
- **EXPECT**: Validates data quality rules
- **ON VIOLATION DROP ROW**: Removes invalid records
- **ON VIOLATION SET**: Sets default values for invalid records

### Quality Metrics Tracked
- Row counts (total, valid, invalid)
- Constraint violation rates
- Data freshness metrics
- Schema evolution tracking

## Setup Instructions

### Prerequisites
- Databricks workspace with DLT access
- Unity Catalog enabled
- Appropriate permissions for table creation

### 1. Data Preparation
```python
# Run feed_raw_tables.ipynb to load sample data
# This creates the source tables:
# - gds_de_bootcamp_new.default.raw_diagnosis_map
# - gds_de_bootcamp_new.default.raw_patients_daily
```

### 2. DLT Pipeline Configuration
```json
{
  "name": "healthcare_dlt_pipeline",
  "notebook_path": "/notebooks/healthcare_dlt_processing",
  "target": "gds_de_bootcamp_new.default",
  "configuration": {
    "quality": "bronze"
  }
}
```

### 3. Pipeline Execution
1. Create DLT pipeline using the configuration above
2. Start the pipeline
3. Monitor execution in DLT UI
4. View data quality metrics and results

## Key Features

### 1. Real-time Streaming
- Uses `STREAM()` function for continuous data processing
- Automatic checkpointing and recovery
- Low-latency data processing

### 2. Data Quality Enforcement
- Comprehensive constraint validation
- Automatic violation handling
- Quality metrics tracking

### 3. Schema Evolution
- Automatic schema inference
- Type casting for consistency
- Backward compatibility

### 4. Multi-dimensional Analytics
- Time-series analysis (by admission date)
- Medical insights (by diagnosis)
- Demographic analysis (by gender)

## Monitoring and Observability

### DLT UI Features
- **Pipeline Status**: Real-time execution monitoring
- **Data Quality**: Constraint violation tracking
- **Lineage**: Data flow visualization
- **Metrics**: Performance and quality metrics

### Key Metrics to Monitor
- Pipeline execution time
- Data quality violation rates
- Row processing counts
- Schema evolution events

## Best Practices Implemented

### 1. Data Quality
- Explicit type casting
- Comprehensive constraint validation
- Graceful error handling

### 2. Performance
- Efficient joins and aggregations
- Streaming for real-time processing
- Optimized table properties

### 3. Maintainability
- Clear naming conventions
- Comprehensive documentation
- Modular pipeline design

### 4. Scalability
- Streaming architecture
- Automatic scaling
- Resource optimization

## Use Cases

### 1. Hospital Operations
- Daily admission tracking
- Capacity planning
- Resource allocation

### 2. Medical Research
- Diagnosis prevalence analysis
- Demographic health studies
- Trend identification

### 3. Quality Assurance
- Data quality monitoring
- Compliance reporting
- Audit trails

## Troubleshooting

### Common Issues

#### 1. Data Quality Violations
- **Symptom**: High violation rates in DLT UI
- **Solution**: Review source data quality, adjust constraints

#### 2. Schema Evolution
- **Symptom**: Pipeline failures due to schema changes
- **Solution**: Use `mergeSchema` option, update type casting

#### 3. Performance Issues
- **Symptom**: Slow pipeline execution
- **Solution**: Optimize joins, review data volume, adjust cluster size

### Debugging Steps
1. Check DLT UI for error messages
2. Review data quality metrics
3. Validate source data format
4. Check Unity Catalog permissions

## Future Enhancements

### 1. Advanced Analytics
- Machine learning integration
- Predictive modeling
- Anomaly detection

### 2. Data Governance
- Column-level security
- Data lineage tracking
- Compliance frameworks

### 3. Performance Optimization
- Z-ordering optimization
- Partitioning strategies
- Caching mechanisms

## Conclusion

This Healthcare DLT Pipeline demonstrates a production-ready implementation of the medallion architecture with comprehensive data quality enforcement, real-time streaming capabilities, and multi-dimensional analytics. The pipeline provides a solid foundation for healthcare data processing with built-in observability and maintainability features.

## Contact

For questions or issues with this pipeline, please refer to the Databricks documentation or contact the data engineering team.
