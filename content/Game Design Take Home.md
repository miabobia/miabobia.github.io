
# Research Methodologies

To research ethnic and cultural data in Alberta I would utilize my talents with web-scraping and automation. First I would use Statistics Canada API endpoints to read census data. The Albertan government also has the Alberta Open Data Portal (open.alberta.ca) which breakdown demographics by health regions.

This approach sounds complex, but it boils down to very simple scripts. For example:
```python
import requests import pandas as pd # Statistics Canada API for demographic
data statscan_url = "https://www150.statcan.gc.ca/t1/wds/rest/getFullTableDownloadCSV/en/98-10-0283-01" # Extract: Age 60+, ethnicity, geographic distribution, income levels
``` 