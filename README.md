# DMM English Teachers Analysis
This is my 6th project of Udacity Data Scientist Nanodegree.  In this project, I analyzed the data of over 6k teachers of [DMM Eikaiwa](https://eikaiwa.dmm.com). I also wrote a blog page based on the findings which is in BlogPostPage.pdf.

## Repo structure  
├── README.md (this file)  
├── 190111_reviews_scraping.ipynb (ipynb to scrape anonymous 300k reviews)  
├── 190117_teachers_scraping.ipynb (ipynb to scrape 6k teachers profiles)  
├── 190306_wrangling_for_writing_blog.ipynb (ipynb to wrangle datasets)  
├── BlogPostPage.pdf (a main deliverable of this repo)    
(** Files below are not pushed to remote repo **)  
├── teacher_urls.txt (txt files which stores urls of all teachers)  
├── raw_teachers_data2.db (database file of uncleaned teachers data)  
├── teachers_data.db (database file of cleaned teachers data)  
├── reviews.db (database file of anonymous reviews which includes duplicates)  
└── reviews2.db (database file of anonymous reviews whose duplicates removed)  

## Repo Detail
I recommend you don't run 2 files in this repo, which are "190111_reviews_scraping.ipynb" and "190117_teachers_scraping.ipynb". This is because it takes many many hours to scrape data from the website. Also, datasets gained from them are sensitive. To avoid problems, I want you to just read these ipynb files and also enjoy reading the blog post ("BlogPostPage.pdf") which is a main deliverable of this analysis.
