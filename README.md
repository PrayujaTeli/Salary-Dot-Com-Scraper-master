
# Salary.com Data Scraper

This Python project extracts salary information for specific job titles across the 317 largest U.S. cities using web scraping techniques.

## Requirements

- Python 3.x
- BeautifulSoup4
- Requests

You can install the required libraries by running the following command:

```bash
pip install beautifulsoup4 requests
```

## How It Works

The script extracts salary data from the **Salary.com** website using the following methodology:

1. Navigate to [Salary.com](https://www.salary.com/).
2. Enter a job position in the search box and choose the most appropriate match from the result descriptions.
3. Once the salary distribution curve is displayed, click on the "View as Table" link to access the salary data in table format.
4. Examine the URL structure and use it as a template for building the request URL.
5. Modify the URL for different cities to gather the relevant data.

## URL Pattern

The URL pattern to scrape salary information looks like:

```
https://www.salary.com/research/salary/alternate/{}-salary/{}
```

Where `{}` is replaced with the job position and city.

## Data Extraction

The data is embedded in a `script` tag on the webpage in JSON format. The scraper extracts this JSON data and parses it to obtain the following information:

- Job Title
- Location
- Job Description
- Salary Percentiles (10th, 25th, 50th, 75th, 90th)

## Extracting Salary Information

The following function is used to extract salary data:

```python
def extract_salary_info(job_title, job_city):
    """Extract and return salary information"""
    template = 'https://www.salary.com/research/salary/alternate/{}-salary/{}'
    url = template.format(job_title, job_city)
    
    # Request the raw HTML
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    # Extract JSON data embedded in the script tag
    pattern = re.compile(r'Occupation')
    script = soup.find('script', {'type': 'application/ld+json'}, text=pattern)
    json_raw = script.contents[0]
    json_data = json.loads(json_raw)
    
    # Extract salary data
    job_title = json_data['name']
    location = json_data['occupationLocation'][0]['name']
    description = json_data['description']
    ntile_10 = json_data['estimatedSalary'][0]['percentile10']
    ntile_25 = json_data['estimatedSalary'][0]['percentile25']
    ntile_50 = json_data['estimatedSalary'][0]['median']
    ntile_75 = json_data['estimatedSalary'][0]['percentile75']
    ntile_90 = json_data['estimatedSalary'][0]['percentile90']

    return (job_title, location, description, ntile_10, ntile_25, ntile_50, ntile_75, ntile_90)
```

## Iterating Over Multiple Cities

The script extracts salary data for multiple cities from a list of the largest U.S. cities. The data is stored in a CSV file:

```python
with open('salary-results.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerow(['Title','Location', 'Description', 'nTile10', 'nTile25', 'nTile50', 'nTile75', 'nTile90'])
    writer.writerows(salary_data)
```

## Main Function

The `main()` function allows for scraping salary data for a given job title across the top U.S. cities and saves the results to a CSV file:

```python
def main(job_title):
    """Extract salary data from top US cities"""
    
    with open('largest_cities.csv', newline='') as f:
        reader = csv.reader(f)
        cities = [city for row in reader for city in row]
        
    salary_data = []
    for city in cities:
        result = extract_salary_info(job_title, city)
        if result:
            salary_data.append(result)
            sleep(0.5)
            
    with open('salary-results.csv', 'w', newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        writer.writerow(['Title','Location', 'Description', 'nTile10', 'nTile25', 'nTile50', 'nTile75', 'nTile90'])
        writer.writerows(salary_data)
        
    return salary_data
```

## Sample Output

The output CSV file (`salary-results.csv`) will contain the following columns:

- `Title`
- `Location`
- `Description`
- `nTile10`
- `nTile25`
- `nTile50`
- `nTile75`
- `nTile90`

## Usage

1. Modify the job title and cities as needed.
2. Run the script in either Colab or Jupyter Notebook cell by cell to extract and save the salary data.

