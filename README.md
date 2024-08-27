# Dewr-data
Scrapper code

import scrapy
import os
from urllib.parse import urljoin
class DewrEvaluationsSpider(scrapy.Spider):
    name = 'dewr_evaluations'
    allowed_domains = ['dewr.gov.au']
    start_urls = ['https://www.dewr.gov.au/employment-services-evaluations']
    
    custom_settings = {
        'DOWNLOAD_DELAY': 2,  # Delay between requests
        'USER_AGENT': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36'
    }
    file_counter = 1

    def parse(self, response):
        # Check if the content is HTML
        if "text/html" in response.headers.get("Content-Type", b"").decode("utf-8"):
            # Extract links to DOCX and PDF files
            for link in response.css('a.file::attr(href)').getall():
                if link.endswith('.docx'):
                    yield response.follow(link, callback=self.save_file)

            # Follow other internal links
            for link in response.css('a::attr(href)').getall():
                if link.startswith('/'):
                    yield response.follow(link, callback=self.parse)
        else:
            # If the content is a PDF or DOCX, save the file
            self.save_file(response)
            
    def save_file(self, response):
        file_name = response.url.split('/')[-1]
        file_extension = file_name.split('.')[-1]
        unique_filename = f"{self.file_counter}_{file_name}"
        self.file_counter += 1
        # Determine the folder based on file extension
        folder_map = {
            'txt': 'downloaded_files/txt_files',
            'html': 'downloaded_files/html_files',
            'epub': 'downloaded_files/epub_files',
            'mobi': 'downloaded_files/mobi_files',
            'pdf': 'downloaded_files/pdf_files',
            'doc': 'downloaded_files/doc_files'
        }

        folder = folder_map.get(file_extension, 'downloaded_files/others')
        save_path = os.path.join(folder, unique_filename)
        os.makedirs(os.path.dirname(save_path), exist_ok=True)

        with open(save_path, 'wb') as f:
            f.write(response.body)
        self.logger.info(f'Saved file {unique_filename}')
