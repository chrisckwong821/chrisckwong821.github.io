---
layout: post
header-img: "img/remote.jpg"
title: Web Scraping II - On a Remote Ubuntu Machine
categories: [Programming]
tags: [Python, Web Scraping, Ubuntu]
fullview: true
comments: true
---

In the last article, I have managed to collected and saved the betting odds on the Hong Kong Jockery Club, using **Splinter**, a Python light-weight scrapper leveraging on **Selenium**.

As scraping on local machine is versatile, it is suggested to be done on a remote machine. In this article I would like to implement the crawler over a remote Ubuntu machine through **DigitalOcean.** 

For your reference, I am using a Mac so most of my command would be most suited for Mac-user.


First, for the purpose of creating a basic remote virtual machine, I picked DigitalOcean(DO) because it is the most easiest one to start with. For the most basic virtual machine, with 20GB, 512MB Memory of Ubuntu 16.04 **$USD 5 per month**. The price is comparable to Amazon Web Service(AWS) T2 Nano, but with no up-front cost or annual commitment. 

Moreover, DO is super easy to use and dont have to do any configuration or download an SDK. This brings a huge convenience. I have tried Google Cloud(GC) and Microsoft Azure(MSA), creating a virtual machine on DigitalOcean is by far the most effortless one from my experience.


**Procedures with code descriptions:**

    1. Initializes a machine(droplet) with python 3.5 pre-installed 
    using the one-click app in DigitalOcean

    2. Connect to the machine through ssh built-in in Mac, type 
    $ ssh root@theipofvm 
    Then type the password that would be sent to your email from DigitalOcean.

    3. Install dependencies including 
    - Splinter (also Selenium implcitly)
    - Chromedriver for automated browsing
    - Chrome as a browser. 
    
    For Splinter:
    $ pip3 install Splinter 

    For chromedriver:

    $ CHROME_DRIVER_VERSION=`curl -sS chromedriver.storage.googleapis.com/LATEST_RELEASE`
    $ wget -N http://chromedriver.storage.googleapis.com/$CHROME_DRIVER_VERSION/chromedriver_linux64.zip -P ~/
    $ unzip ~/chromedriver_linux64.zip -d ~/
    $ sudo mv -f ~/chromedriver /usr/local/bin/chromedriver
    $ sudo chown root:root /usr/local/bin/chromedriver
    $ sudo chmod 0755 /usr/local/bin/chromedriver
       
    For Chrome:
    $ wget -N https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb -P ~/
    $ sudo dpkg -i --force-depends ~/google-chrome-stable_current_amd64.deb
    $ sudo apt-get -f install -y
    $ sudo dpkg -i --force-depends ~/google-chrome-stable_current_amd64.deb

Then once all dependencies are settled:

    4. Copy the below code into a .py file, eg: jockery.py, it is basically the exact code over the last article, scraping betting odds from Jockery Club and save it to a csv:



```python
import re
import csv
import time
from splinter import Browser

pattern = '(rmid11[0-9]{4})'
matchtime, home, away = [], [] , []
odd_home, odd_draw, odd_away = [], [], []

browser = Browser('chrome',headless=True)
browser.visit('http://bet.hkjc.com/football/index.aspx?lang=en')
ids = re.findall(pattern, browser.html)
for match in ids:
    entry = browser.find_by_id(match).text
    regex = re.search(r'[\d]\s(.+)\svs\s(\D+)\s(.+)\n(\S+)\s(\S+)\s(\S+)', entry)
    home.append(regex.group(1))
    away.append(regex.group(2))
    matchtime.append(regex.group(3))
    odd_home.append(regex.group(4))
    odd_draw.append(regex.group(5))
    odd_away.append(regex.group(6))

date = [time.strftime("%Y/%m/%d")] * len(ids)
refresh = [browser.find_by_id('sRefreshTime').value] * len(ids)

rows = zip(matchtime,date,refresh,home,away,odd_home,odd_draw,odd_away)
with open('football.odds.csv','a') as f:
    writer = csv.writer(f)
    for row in rows:
        writer.writerow(row)
    f.close()

browser.quit()

```


    5. Check the path to python by typing 
    $ which python3
    Copy the output into crontab, then write the automation:
    * * * * * /pathtoyourpython /root/jockery.py into the editor.

    6. If the above does not work, check the path:
    $ $PATH
    copy and paste the output on the top of crontab.
    PATH=theoutputofyourpath 


For improvement, instead of saving the data locally in the virtual machine, the data can be pushed immediately to some cloud storage. I would add an edit to it later.

Out of curiosity, I have browsed some web-based crawlers. Surprisingly, there are a bunch of them which are easy to use and have some threshold for free development.
So I guess this example is more for fun or proof of concept than what would be done in practice.

