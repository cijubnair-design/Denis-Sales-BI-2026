# Data Sources

Source data files are not included in this repository as they contain proprietary business data belonging to Denis Company.

## Data Structure

### Denis_G.csv
Main sales transactions file containing:
- Transaction ID
- Date
- Product
- Category / SubCategory
- City / Region
- Revenue
- Cost

### Access_Table.csv
RLS access mapping file containing:
- Username (email)
- Location_Access (city/region or "All")

## Data Model

The report uses a **star schema** design:
- **Fact table:** Sales transactions (Denis_G.csv)
- **Dimension tables:** Date table (DAX-generated), Products, Regions
- **Security table:** Access_Table (for dynamic RLS)

All relationships are single-direction (one-to-many) from dimension to fact, following Power BI best practices for filter propagation.
