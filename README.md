DEWR Evaluations Scraper
This project uses a Scrapy spider to scrape DOCX and PDF files from the DEWR Employment Services Evaluations website.

Step-by-Step Guide
1. Set Up Your Environment
Install Python 3.11 or later.
Install Scrapy by running:
bash
Copy code
pip install scrapy
2. Create the Scrapy Project
Create a new Scrapy project:
bash
Copy code
scrapy startproject dewr_scraper
Navigate to the dewr_scraper/spiders directory and create a new spider file dewr_evaluations_spider.py.
3. Write the Spider
In the dewr_evaluations_spider.py file, define the spider class. Set the start_urls to the DEWR evaluations page.
Implement the parse method to extract DOCX and PDF links using CSS selectors.
Use the save_file method to download and save the files. Ensure filenames are unique by appending a counter or timestamp.
Categorize the files by extension and save them in designated folders.
4. Run the Spider
Navigate to the project directory:

bash
Copy code
cd dewr_scraper
Run the spider:

bash
Copy code
scrapy crawl dewr_evaluations
The files will be downloaded and saved in categorized folders within the downloaded_files/ directory.

5. File Organization
Files are saved in the following directories based on their extension:
txt_files/
html_files/
epub_files/
mobi_files/
pdf_files/
doc_files/
others/
6. Customization
Adjust the folder_map dictionary in the save_file method to customize file categorization.
7. Further Processing
Once downloaded, the files can be processed or cleaned as needed.
License
This project is licensed under the MIT License.
