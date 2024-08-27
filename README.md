Here's a README file with step-by-step instructions on how you scraped the DEWR website:

---

# DEWR Evaluations Scraper

This project uses a Scrapy spider to scrape DOCX and PDF files from the DEWR Employment Services Evaluations website.

## Step-by-Step Guide

### 1. **Set Up Your Environment**

   - Install Python 3.11 or later.
   - Install Scrapy by running:
     ```bash
     pip install scrapy
     ```

### 2. **Create the Scrapy Project**

   - Create a new Scrapy project:
     ```bash
     scrapy startproject dewr_scraper
     ```
   - Navigate to the `dewr_scraper/spiders` directory and create a new spider file `dewr_evaluations_spider.py`.

### 3. **Write the Spider**

   - In the `dewr_evaluations_spider.py` file, define the spider class. Set the `start_urls` to the DEWR evaluations page.
   - Implement the `parse` method to extract DOCX and PDF links using CSS selectors.
   - Use the `save_file` method to download and save the files. Ensure filenames are unique by appending a counter or timestamp.
   - Categorize the files by extension and save them in designated folders.

   Create a new spider file named `dewr_evaluations_spider.py` and paste the following code:

   ```python
import scrapy
import os
from urllib.parse import urljoin

class DewrEvaluationsSpider(scrapy.Spider):
    name = 'dewr_evaluations'  # Name of the spider
    allowed_domains = ['dewr.gov.au']  # Allowed domain to prevent the spider from going off-site
    start_urls = ['https://www.dewr.gov.au/employment-services-evaluations']  # Starting URL for the spider
    
    # Custom settings for the spider
    custom_settings = {
        'DOWNLOAD_DELAY': 2,  # Delay of 2 seconds between requests to avoid overloading the server
        'USER_AGENT': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36'
    }
    
    file_counter = 1  # Counter to ensure unique filenames for downloaded files

    def parse(self, response):
        """
        The main parsing function. This handles the response from the initial request 
        and follows links to DOCX and PDF files, as well as other internal pages.
        """
        # Check if the response content is an HTML page
        if "text/html" in response.headers.get("Content-Type", b"").decode("utf-8"):
            # Extract links to DOCX files
            for link in response.css('a.file::attr(href)').getall():
                if link.endswith('.docx'):
                    yield response.follow(link, callback=self.save_file)

            # Follow other internal links to continue crawling the site
            for link in response.css('a::attr(href)').getall():
                if link.startswith('/'):
                    yield response.follow(link, callback=self.parse)
        else:
            # If the response is a direct file (like PDF or DOCX), save it
            self.save_file(response)
            
    def save_file(self, response):
        """
        Saves the downloaded file with a unique filename in an appropriate folder 
        based on the file extension.
        """
        # Extract the original filename from the URL
        file_name = response.url.split('/')[-1]
        file_extension = file_name.split('.')[-1]
        
        # Create a unique filename using the counter
        unique_filename = f"{self.file_counter}_{file_name}"
        self.file_counter += 1  # Increment the counter for the next file

        # Map file extensions to corresponding folders
        folder_map = {
            'txt': 'downloaded_files/txt_files',
            'html': 'downloaded_files/html_files',
            'epub': 'downloaded_files/epub_files',
            'mobi': 'downloaded_files/mobi_files',
            'pdf': 'downloaded_files/pdf_files',
            'doc': 'downloaded_files/doc_files'
        }

        # Determine the appropriate folder, defaulting to 'others' if the extension is unrecognized
        folder = folder_map.get(file_extension, 'downloaded_files/others')
        save_path = os.path.join(folder, unique_filename)
        
        # Create the directory if it doesn't exist
        os.makedirs(os.path.dirname(save_path), exist_ok=True)

        # Save the file content to the specified path
        with open(save_path, 'wb') as f:
            f.write(response.body)
        
        # Log the file saving action
        self.logger.info(f'Saved file {unique_filename}')

   ```

This code defines the spider that will crawl the DEWR website, download DOCX and PDF files, and organize them into categorized folders.

### 4. **Run the Spider**

   - Navigate to the project directory:
     ```bash
     cd dewr_scraper
     ```
   - Run the spider:
     ```bash
     scrapy crawl dewr_evaluations
     ```

   - The files will be downloaded and saved in categorized folders within the `downloaded_files/` directory.

### 5. **File Organization**

   - Files are saved in the following directories based on their extension:
     - `txt_files/`
     - `html_files/`
     - `epub_files/`
     - `mobi_files/`
     - `pdf_files/`
     - `doc_files/`
     - `others/`

### 6. **Customization**

   - Adjust the `folder_map` dictionary in the `save_file` method to customize file categorization.

### 7. **Further Processing**

   - Once downloaded, the files can be processed or cleaned as needed.
