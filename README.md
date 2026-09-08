# Character Choice's Effect on Winrates Across Ranks
This project aims to interpret data from the multiplayer game "League of Legends" to analyze how character choice's effect on winrates differs across each of its different ranks.

## Overview
League of Legends (LoL) is a massively popular multi-player online game with ~117-135 million monthly players. Each player chooses a unique character in what's known as the draft stage. It is at this stage that characters are banned and then chosen in an alternating pattern. LoL has 172 characters to chose from, each with their own strengths and weaknesses. As with any competitive game, a metagame has developed with certain characters being stronger on average compared to other characters. Because of this, these characters naturally become more contested during the drafting phase. It is common for players to believe that by picking these stronger characters, they may be able to expedite the long process of climbing ranks in LoL. Using cross-sectional data collected by webscraping the website [LoLalytics](https://lolalytics.com/lol/tierlist/), I compare how winrates change between each of the 10 League of Legend's Ranks. This resulted in the discovery that winrates are most strongly affected in middle-to-high ranks compared to higher and lower ranks. Suggesting that players who do put in the time to climb high in ranks, but are not skilled enough to reach the highest ranks are the ones who benefit the most from selecting dominant strategies.

*This project primarily served as a practical application of webscraping through Python rather than a pure econometric excercise.*

## Data
### Source: 
[LoLalytics](https://lolalytics.com/lol/tierlist/); collects live match data from League of Legend's Server and compiles it into it into an interactable database. 

### Sample Size:
1,702 observations across 10 ranks + 1 aggregated rank.

### Key Variables:
#### Pick Ban Influence Index:
**PBI.Index** - A measure of character competitiveness using a ratio of pick rate to ban rate multiplied by the difference in winrate in a rank and the average winrate across ranks. 
**PBI.no** - The PBI index measurement with winrate removed from the calculation. 

#### Winrate:
**wr** - How often a character choice results in a victory when picked. 

## Webscraping with Selenium 
The project file "Parts 1-3" includes more detail for how each part of the data was gathered.
```
# Required Packages
import os 
import pandas as pd
import numpy as np
import time
from time import sleep
import selenium 
from selenium import webdriver

from selenium.webdriver.common.by import By #Allows for selenium to click things 
from selenium.webdriver.chrome.service import Service #https://stackoverflow.com/questions/64717302/deprecationwarning-executable-path-has-been-deprecated-selenium-python
from selenium.webdriver.support import expected_conditions as EC #Allows for more complex code 
from selenium.webdriver.chrome.options import Options #Allows you to change aspects of the browser

# Establish options we can change
chrome_options = Options() 
chrome_options.add_argument("--window-size=1900,1000")
```

LoLalytics has multiple pages of data I need. Each of the pages all follow the same format, which makes using a For loop simple for data collection. This For loop is nested within another For loop which cycles through each of the different ranks. It does this by physically clicking on the page by using the .click() function. Each page takes time to load so I include code to scroll down the page to allow every element to fully load in. If this step was not included, then the code would be unable to search for later elements at the bottom of the page. I use explicit wait commands to allow each element to load in. 

```
driver = webdriver.Chrome(options = chrome_options) # establish driver

url = 'https://lolalytics.com/lol/tierlist/' # dataset for each champions
driver.get(url) # Get the url

ranknum=[3, 4, 6, 8, 11, 13, 15, 17, 18, 19] # the element for each rank selection

import time            # importing time package
start=time.time()      # start time

from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
wait = WebDriverWait(driver, 20) # explicit wait object, 20s timeout

out2 = [] # empty list to convert into dataframe later

for r in ranknum:
    out = [] # empty list for data
    count = 3 # starts at 3 to skip the first 2 which are irrelevant elements
    wait.until(EC.element_to_be_clickable((By.XPATH,'/html/body/main/div[1]/div/div/div[3]/div/div/div[1]/img'))).click() # clicks the rank sort
    wait.until(EC.element_to_be_clickable((By.XPATH,f'/html/body/main/div[1]/div/div/div[3]/div[2]/a[{r}]/div'))).click() # clicks the rank
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);") # scrolls to the very bottom of the page to load everything (takes ~20 seconds)
    wait.until(lambda d: len(d.find_elements(By.XPATH, '/html/body/main/div[6]/div')) > 2) # want to wait until things load
    driver.execute_script("window.scrollBy(0, -6000);") # scrolls back up to load everything
    for i in range(0,18): # scrolls downs for the next 18 seconds to make sure everything loads
        driver.execute_script("window.scrollBy(0, 320);")
        wait.until(EC.presence_of_element_located((By.XPATH, f'/html/body/main/div[6]/div[{3+i}]')))
    driver.execute_script("window.scrollTo(0,0);") # scrolls back to the top of the page to also open up the rank selection menu for once the loop resets
    
    buckets = driver.find_elements(By.XPATH, '/html/body/main/div[6]/div') # creating a bucket of each champion
    del buckets[:2] # removes the first 2 from the list to skip the first 2 which are just the title rows
    
    rank = driver.find_elements(By.XPATH, '/html/body/main/div[1]/div/div/div[3]/div/div/div[2]')[0].text # grabs the rank

    temp1=driver.find_elements(By.XPATH, f'/html/body/main/div[1]/div/div/div[10]/div')[0].text # collects the average win rate for that rank
    rankwr=float(temp1.split('%')[0].split("Rate: ")[1]) # collects the average win rate for that rank

    for bucket in buckets: # runs the below code for every champion in the list
        champion = driver.find_elements(By.XPATH, f'/html/body/main/div[6]/div[{count}]/div[3]/a')[0].text # grabs champion name
        wr = driver.find_elements(By.XPATH, f'/html/body/main/div[6]/div[{count}]/div[6]/div/span')[0].text # grabs win rate
    
        temp2= driver.find_elements(By.XPATH, f'/html/body/main/div[6]/div[{count}]/div[6]/div/span')
        wrdelta= temp2[1].text if len(temp2) > 1 else 0 # checks to see if there is an entry for wrdelta (some are missing on the website)

        pick = driver.find_elements(By.XPATH, f'/html/body/main/div[6]/div[{count}]/div[7]')[0].text # grabs pick rate
        ban = driver.find_elements(By.XPATH, f'/html/body/main/div[6]/div[{count}]/div[8]')[0].text # grabs ban rate
        PBI = driver.find_elements(By.XPATH, f'/html/body/main/div[6]/div[{count}]/div[9]')[0].text # grabs PBI
        lane = driver.find_elements(By.XPATH, f'/html/body/main/div[6]/div[{count}]/div[5]/div/img')[0].get_attribute('alt') # grabs lane
        games = driver.find_elements(By.XPATH, f'/html/body/main/div[6]/div[{count}]/div[10]')[0].text # grabs the number of games

        data = {
            'Champion':champion,
            'wr':wr,
            'wrdelta':wrdelta,
            'rankwr':rankwr,
            'Pick Rate':pick,
            'Ban Rate':ban,
            'PBI Index':PBI,
            'lane':lane,
            'games':games,
            'rank':rank
        }
        out.append(data) # adds champion data to a list that will be added at the end of this for loop
        count+=1
    out2.append(out) # EACH OF THE RANKS'S DATA IS ADDED TO A FINAL LIST THAT CAN BE SLICED TO GRAB EACH SET
    ## EDIT: STILL KEEPING THIS FUNCTION, BUT PUTTING EVERYTHING INTO ONE DATASET INSTEAD OF TEN. STILL KEEPING JUST IN CASE !!!

end=time.time()        # end time

total_time=end-start   # measures total time by subtracting start by end
print(f' The code takes {round(total_time,5)} seconds to run.')
```
