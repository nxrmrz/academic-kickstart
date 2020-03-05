---
title: Web Scraping
summary: Using Selenium and Python's BeautifulSoup to scrape Salary Information from Glassdoor
tags:
- Data Projects
date: "06-03-2020"

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: Web Scraping feels like Hacking!
  focal_point: Smart

links: ""
url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---

### The Idea

An ex-colleague, [Dhilip](https://www.linkedin.com/in/dhilip-subramanian-36021918b/) and I wanted to do a series of end-to-end projects in Data Science and Analytics for fun and to pick up skills along the way that would serve our future careers in this space. When Dhilip approached me for project ideas, I already had one I've thought about for quite some time. 

It was a timely project for us Qrious interns, who were deliberating on our next career move following the summer internship. Towards the end of our internship, a few of us (excluding me but including Dhilip) were graduating and applying for full-time jobs in the data space. Our discussions naturally gravitated to what starting pay we could expect for careers in data science, data engineering, and analytics. I imagined if we had an amount for each job averaged from recent, accurate salary data collected over a large number of companies operating in different industries, we had evidence to make sure future salary negotiations were fair! 

Salary negotiations usually happen *after* an offer is extended. I then wondered if I could collect data to help my job-searching friends *recieve* an offer in the first place. Data such as the top skills needed for each job title would help them know what to upskill on throughout a job search, or what to highlight in their resumes, for example.

### The Execution

Glassdoor is a job reviews site with job benefits and salary information reported anonymously by employees of various companies. Indeed and Seek are job-hunting sites with job descriptions and the occasional salary information as well, reported by companies themselves. Data from all three sites would be perfect for our analysis! Collecting data from the sites would need to be automated, in a process called *scraping*. This is a technique used to extract content from specific HTML tags in a webpage, and it can exploit two technologies to do so: Selenium and BeautifulSoup.

Selenium is a Java-based tool used in website testing to automate specific interactions with webpages (i.e. clicking on links, logging in, navigating the page, etc). Since it automates interacting with the DOM, it's being used in Data Science to scrape data from websites. BeautifulSoup, on the other hand, is a Python library that parses HTML and makes it easy to extract specific elements from it.

### The code

A work in progress (explanations to follow:)

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import pandas as pd
# from selenium.webdriver.common.keys import Keys
from time import sleep
from bs4 import BeautifulSoup, Comment
import time
import os
import csv
import lxml
from itertools import zip_longest
from webdriver_manager.chrome import ChromeDriverManager

#setting up an automated google chrome browser
driver = webdriver.Chrome(ChromeDriverManager().install())

job_title = []
company_name = []
mean_pay = []
pay_range = []

for pageno in range(1,184):

    driver = webdriver.Chrome(ChromeDriverManager().install())
    
    #getting webpage in glassdoor
    if pageno == 1:
        driver.get("https://www.glassdoor.co.nz/Salaries/us-data-engineer-salary-SRCH_IL.0,2_IN1_KO3,16.htm")
    else:
        driver.get(
            "https://www.glassdoor.co.nz/Salaries/us-data-engineer-salary-SRCH_IL.0,2_IN1_KO3,16.htm" + "_IP" + str(pageno) + ".htm"
        )
    time.sleep(1.5)

    #parsing the page through lxml option of beautifulsoup
    html = driver.page_source
    soup = BeautifulSoup(html, 'lxml')

    #getting each salary block
    salaryBlocks = soup.findAll("div", {'class' : 'row align-items-center m-0 salaryRow__SalaryRowStyle__row'})
    #once you've done that for initial page, scroll to next page and do again. save results into one main file

    for block in salaryBlocks:
        entry = []

        jobTitle = block.find("div", {'class' : 'salaryRow__JobInfoStyle__jobTitle strong'}).find("a").text
        job_title.append(jobTitle)

        companyName = block.find("div", {'class' : 'salaryRow__JobInfoStyle__employerName'}).text
        company_name.append(companyName)

        meanPay = block.find("div", {'class' : 'salaryRow__JobInfoStyle__meanBasePay common__formFactorHelpers__showHH'}).find('span').text
        mean_pay.append(meanPay)
        
        try:
            if block.find("div", {'class' : 'col-2 d-none d-md-block px-0 py salaryRow__SalaryRowStyle__amt'}).find("div", {'class' : 'strong'}):
                payRange = block.find("div", {'class' : 'col-2 d-none d-md-block px-0 py salaryRow__SalaryRowStyle__amt'}).find("div", {'class' : 'strong'}).text
                pay_range.append(payRange)
            elif block.find("div", {'class' : 'col-2 d-none d-md-block px-0 py salaryRow__SalaryRowStyle__amt'}).find("span", {'class' : 'strong'}):
                pay_range.append("N/A")
        except:
            pay_range.append("N/A")

        driver.quit()

final = []
for item in zip_longest(job_title, company_name, mean_pay, pay_range):
    final.append(item)

df = pd.DataFrame(
    final, columns=['jobTitle', 'companyName', 'meanPay', 'payRange'])

df.to_csv("Data Engineer Salaries United States.csv")
```
