# IFRC PER Data Fetcher

A Node.js application that fetches, processes, and generates JSON data files for the IFRC Preparedness for Effective Response (PER) Dashboard.

## Related Projects

This data fetcher was developed in January 2025 to support the PER Dashboard components in the IFRC GO Platform:

- **GO Web App**: [IFRCGo/go-web-app](https://github.com/IFRCGo/go-web-app)
  - The main IFRC GO Platform web application
  - Contains two complementary PER Dashboard views in `app/src/views/PERDashboard/`:
    1. **Summary Dashboard**: Overview of global PER implementation, showing:
       - Assessment completion rates by region
       - Global statistics and trends
       - Special considerations coverage (Urban, Epidemic, Climate)
       - Geographic distribution of assessments
    2. **Performance Dashboard**: Detailed analysis of assessment results, including:
       - Component-level ratings and trends
       - Area-wise performance breakdown
       - Cycle-to-cycle comparisons
       - Detailed country-level insights
  - Has a storybook implementation in `packages/go-ui-storybook/src/stories/`
  - Uses the processed data from this fetcher for visualizations and analysis

The PER Dashboards provide interactive visualizations and analysis tools for National Societies' preparedness assessments. This data fetcher optimizes and pre-processes the raw API data to enable efficient dashboard performance and reduce client-side computation.

## Overview

The PER Data Fetcher serves as a data pipeline that:
1. Fetches raw assessment data from the IFRC API
2. Processes and transforms the data into optimized formats
3. Generates JSON files used by the PER Dashboard for visualization and analysis

## Data Flow

### 1. Data Collection (`fetchData.js`)
- Fetches data from multiple IFRC API endpoints using pagination:
  - `/api/v2/per-status/`: Assessment status information
  - `/api/v2/countries/`: Country data with geographical information
  - `/api/v2/per-prioritization/`: Component prioritization data
  - `/api/v2/per-assessments/`: Detailed assessment data

### 2. Data Processing (`processData.js`)
The raw data is processed and transformed into several specialized JSON files:

#### Data Files and Sizes

#### Input Files
Raw data fetched from API endpoints:
- `per-assessments.json` (~26MB): Raw assessment data and responses
- `countries.json` (~397KB): Country and geographical information
- `prioritization.json` (~708KB): Component prioritization data
- `per-status.json` (~116KB): Assessment status information

#### Output Files
Processed and optimized data for dashboard consumption:
- `per-dashboard-data.json` (~2.9MB): Main dashboard data
  - Grouped assessment data
  - Pre-calculated ratings
  - Optimized component structure
- `map-data.json` (~1.4MB): Geospatial visualization data
  - Simplified geographical data
  - Pre-processed regional statistics
- `component-descriptions.json` (~16KB): Component metadata
  - Component descriptions
  - Relationship mappings
- `per-assessments-processed.json` (~7.8MB): Processed assessment data
  - Filtered and normalized responses
  - Optimized for quick queries
- `last-update.json` (~46B): Timestamp of last data update

#### Performance Impact
- Total raw data size: ~27.2MB
- Total processed data size: ~12.1MB
- Overall size reduction: ~55%
- Update frequency: Every 30 minutes via GitHub Actions

### 3. Data Processing Features

#### Special Considerations Processing
The data fetcher includes specialized text analysis for PER consideration fields:

1. **Input Fields**:
   - `urban_considerations`: Text field for urban-specific considerations
   - `epi_considerations`: Text field for epidemic-related considerations
   - `climate_environmental_considerations`: Text field for climate/environmental factors

2. **Text Analysis Process**:
   - Supports multilingual input (English and Spanish)
   - Searches for affirmative words: ['yes', 'sí', 'si']
   - Text normalization:
     - Converts to lowercase
     - Removes diacritical marks
     - Standardizes variations

3. **Output Fields**:
   Each text field is processed into a corresponding boolean field:
   - `urban_considerations_simplified`
   - `epi_considerations_simplified`
   - `climate_environmental_considerations_simplified`

This processing enables:
- Efficient filtering and aggregation
- Simplified visualization logic
- Reduced client-side processing
- Consistent data representation

#### Component Structure
Components are organized into 5 main areas:
1. Policy Strategy and Standards
2. Analysis and planning
3. Operational capacity
4. Coordination
5. Operations support

#### Assessment Data
Each assessment contains:
- Metadata (ID, country, date, phase)
- Component ratings and responses
- Special considerations (Urban, Epidemic, Climate)
- Notes and additional information

#### Rating System
Components are evaluated using a numerical rating system:
- Ratings are stored in `rating_value`
- Text descriptions in `rating_title`
- Changes between assessments are tracked
- Zero-rated duplicates are filtered out

### 4. Data Usage in PER Dashboard

The dashboard uses the processed data for various visualizations:

#### Main Features:
1. **Component Performance Analysis**
   - Rating trends over time
   - Area-wise performance breakdown
   - Regional comparisons

2. **Assessment Cycle Tracking**
   - Progress through assessment phases
   - Completion status monitoring
   - Cycle-to-cycle comparisons

3. **Geographical Analysis**
   - Regional distribution of assessments
   - Country-level performance tracking
   - Spatial patterns visualization

4. **Special Considerations**
   - Urban considerations tracking
   - Epidemic preparedness monitoring
   - Climate and environmental aspects

## API Integration

### Public API Endpoints

This tool exclusively uses publicly accessible IFRC GO Platform API endpoints, requiring no authentication:

1. **PER Status**
   - Endpoint: `https://goadmin.ifrc.org/api/v2/per-process-status/`
   - Documentation: [GO Platform API - PER Process Status](https://goadmin.ifrc.org/docs#operation/api_v2_per-process-status_list)
   - Contains: Assessment status, phases, and metadata

2. **Countries**
   - Endpoint: `https://goadmin.ifrc.org/api/v2/country/`
   - Documentation: [GO Platform API - Countries](https://goadmin.ifrc.org/docs#operation/api_v2_country_list)
   - Contains: Country information, regions, and geographical data

3. **PER Prioritization**
   - Endpoint: `https://goadmin.ifrc.org/api/v2/per-prioritization/`
   - Documentation: [GO Platform API - PER Prioritization](https://goadmin.ifrc.org/docs#operation/api_v2_per-prioritization_list)
   - Contains: Component prioritization and relationships

4. **PER Assessments**
   - Endpoint: `https://goadmin.ifrc.org/api/v2/per-assessment/`
   - Documentation: [GO Platform API - PER Assessment](https://goadmin.ifrc.org/docs#operation/api_v2_per-assessment_list)
   - Contains: Detailed assessment responses and ratings

### API Documentation

- Full API documentation: [IFRC GO Platform API Docs](https://goadmin.ifrc.org/docs)
- API versioning: All endpoints use v2 of the API
- Rate limiting: Public endpoints have standard rate limits
- Response format: All endpoints return JSON data with pagination
- Common query parameters:
  - `limit`: Number of records per page
  - `offset`: Pagination offset
  - `ordering`: Field to sort by
  - `format`: Response format (json/api)

### API Response Format

Example of raw API response structure:
```json
{
  "count": 100,
  "next": "https://goadmin.ifrc.org/api/v2/per-assessment/?offset=30",
  "previous": null,
  "results": [
    {
      "id": 123,
      "country_details": {
        "iso3": "USA",
        "name": "United States of America",
        "region": 1
      },
      "assessment_number": 2,
      "date_of_assessment": "2024-01-15",
      // ... additional fields
    }
  ]
}
```

Note: While these endpoints are public, please follow IFRC's [API usage guidelines](https://goadmin.ifrc.org/docs#section/API-Usage-Guidelines) and implement appropriate caching to avoid unnecessary load on their servers.

## Data Schema and Integration

### Output Files and Schema

1. **per-dashboard-data.json**
   ```typescript
   interface DashboardData {
     assessments: {
       component_id: number;      // Unique identifier for the component
       component_num: number;     // Sequential number of the component
       component_name: string;    // Name of the component
       area_id: number;          // Identifier for the area
       area_name: string;        // Name of the area
       assessments: Array<{
         assessment_id: number;          // Unique identifier for the assessment
         assessment_number: number;      // Sequential number for the assessment
         country_id: number;            // Country identifier
         country_name: string;          // Country name
         region_id: number;             // Region identifier
         region_name: string;           // Region name
         date_of_assessment: string;    // ISO date format
         rating_value: number;          // Numerical rating
         rating_title: string;          // Text description of rating
       }>;
     }[];
     countryAssessments: {
       [countryName: string]: Array<{
         assessment_number: number;
         date: string;             // ISO date format
         area_ratings: any[];      // Area-specific ratings
         components: any[];        // Component-specific data
         phase: number;            // Assessment phase number
         phase_display: string;    // Human-readable phase name
       }>;
     };
   }
   ```

2. **last-update.json**
   ```typescript
   interface LastUpdate {
     lastUpdate: string;  // ISO date format of last successful update
   }
   ```

### File Hosting and Access

The processed data files are hosted in this repository and accessed via GitHub's raw content URLs. The dashboard components fetch these files directly from GitHub.

#### Configuration

In the GO Web App, the data URLs are configured in the dashboard components:

```typescript
// PER Performance Dashboard
// app/src/views/PERDashboard/PERPerformanceDashboard/index.tsx
const PER_DASHBOARD_DATA_URL = 'https://api.github.com/repos/[org]/ifrc-per-data-fetcher/contents/data/per-dashboard-data.json';
const LAST_UPDATE_DATA_URL = 'https://api.github.com/repos/[org]/ifrc-per-data-fetcher/contents/data/last-update.json';
```

To modify the data source:
1. Fork the data fetcher repository
2. Update the URLs in the dashboard components to point to your fork
3. Ensure your repository is public or provide appropriate authentication

### Data Handlers

The dashboard components use specialized data handlers to process and transform the data for visualization:

1. **Performance Dashboard Handler** (`PERPerformanceDashboard/dataHandler.ts`)
   - Processes component ratings and trends
   - Calculates assessment cycles
   - Groups data by region
   - Generates summary statistics

2. **Summary Dashboard Handler** (`PERSummaryDashboard/dataHandler.ts`)
   - Processes global statistics
   - Handles geographic data for map visualization
   - Calculates completion rates
   - Processes special considerations

### Data Flow

1. **Data Fetching**
   ```typescript
   // Example from PERPerformanceDashboard
   const fetchData = async () => {
     const response = await fetch(PER_DASHBOARD_DATA_URL);
     const data = await response.json();
     initializeData(data);  // Initialize data handlers
   };
   ```

2. **Data Processing**
   ```typescript
   // Example usage in components
   const componentRatings = getComponentRatings(filters);
   const cycles = getCycles(filters);
   const summaryData = summarizeData(filters);
   ```

3. **Data Updates**
   - Files are automatically updated daily via GitHub Actions
   - Components check `last-update.json` for the latest data timestamp
   - Data is cached in the browser to improve performance

### Error Handling

The data handlers include robust error handling for:
- Missing or malformed data
- Invalid ratings or dates
- Missing relationships between components and areas
- Network failures during data fetching

## Data Processing Rationale

### Current Challenges

The PER Data Fetcher exists to solve several challenges with the raw API data:

1. **Data Size Optimization**
   - Raw API response sizes:
     - Per Assessments: ~26MB
     - Countries: ~397KB
     - Prioritization: ~708KB
     - PER Status: ~116KB
   - Processed output sizes:
     - Dashboard Data: ~2.9MB (89% reduction)
     - Map Data: ~1.4MB
     - Component Descriptions: ~16KB
   - The significant size reduction is achieved through:
     - Removing redundant nested data
     - Optimizing data structures for frontend consumption
     - Filtering out unused fields
     - Compressing repeated information

2. **Data Normalization**
   - **Text Processing**: The API returns free-text fields for considerations (Urban, Epidemic, Climate) that need parsing
     ```javascript
     // Examples of variations in API responses:
     "urban_considerations": "Yes, this is considered"
     "urban_considerations": "si"
     "urban_considerations": "Sí, parcialmente"
     ```
   - The fetcher normalizes these into boolean flags by:
     - Handling multilingual responses (English/Spanish)
     - Accounting for accent variations
     - Standardizing affirmative/negative responses

3. **Relationship Management**
   - Raw API data requires multiple requests to establish relationships between:
     - Components and their areas
     - Countries and their regions
     - Assessments and their cycles
   - The fetcher pre-computes these relationships to avoid:
     - Multiple API calls from the frontend
     - Complex data joining in the browser
     - Redundant data processing

4. **Performance Optimization**
   - Pre-calculated aggregations for:
     - Regional statistics
     - Assessment cycle analysis
     - Component ratings
   - Reduces frontend computation needs
   - Improves dashboard rendering speed

### Future Recommendations

1. **Dedicated Dashboard API Endpoint**
   - Current challenges could be better addressed with a dedicated API endpoint that:
     - Returns pre-processed data in dashboard-ready format
     - Handles data aggregation server-side
     - Provides optimized response structures
     - Manages caching effectively

2. **API Improvements**
   - Standardize consideration responses (boolean instead of text)
   - Include pre-calculated relationships
   - Provide built-in aggregation options
   - Support partial data updates
   - Add response compression

3. **Migration Strategy**
   - Keep this data fetcher as a reference implementation
   - Gradually move processing logic to API
   - Maintain backward compatibility
   - Support transition period

The data fetcher serves as a crucial bridge between the current API structure and dashboard requirements. While it effectively solves immediate challenges, a more sustainable long-term solution would be API-level optimizations.

## Usage

1. Install dependencies:
```bash
npm install
```

2. Run the data fetcher:
```bash
npm start
```

This will:
- Fetch fresh data from the IFRC API
- Process and transform the data
- Generate updated JSON files in the `data/` directory

## Data Update Frequency

The data is automatically updated daily using GitHub Actions. The workflow:
1. Runs at 00:00 UTC every day
2. Fetches fresh data from the IFRC API
3. Processes and transforms the data
4. Commits and pushes any changes to the repository

The `last-update.json` file tracks the timestamp of the most recent successful update. You can also:
- View the update history in the repository's commit log
- Manually trigger an update from the Actions tab
- Check the Actions tab for any update failures

## Error Handling

The application includes robust error handling:
- API request retries and pagination handling
- Data validation and cleaning
- Missing data management
- Duplicate assessment filtering

## Dependencies

- Node.js
- Axios for API requests
- File system operations for JSON storage
- Data processing utilities

## Contributing

When contributing to this repository, please:
1. Ensure data processing maintains backward compatibility
2. Add appropriate error handling
3. Update tests if modifying data structures
4. Document any new data transformations

## License

This project is licensed under the terms specified by IFRC.
