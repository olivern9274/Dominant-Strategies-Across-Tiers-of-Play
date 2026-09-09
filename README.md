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

LoLalytics has multiple pages of data I need. Each of the pages all follow the same format, which makes using a For loop simple for data collection. This For loop is nested within another For loop which cycles through each of the different ranks. It does this by physically clicking on the page by using the .click() function. Each page takes time to load so I include code to scroll down the page to allow every element to fully load in. If this step was not included, then the code would be unable to search for later elements at the bottom of the page. I use explicit wait commands to allow each element to load in. Then using for loops, I am able to scrape the data I need from each row of data. 

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

## Methodology
### Step 1: Dataset Construction
Using pandas, I created a dataframe for each rank's data on each character. These dataframe were added to a list which was initially a part of an effort to create 10 different datasets, but has since been made redundant by the next step. Using concat functions, I merged each of the dataframes within the list into one large dataframe.
```
league=pd.DataFrame() # making an empty dataframe
c=0 # counter
for rank in out2:
    temp = pd.DataFrame(out2[c]) # slicing to turn each entry into a new dataframe to concat later
    league = pd.concat([league, temp], ignore_index=True)
    c+=1
```

### Step 2: Dataset Clean-up
Selenium's webscraping collects data as strings. Given the fact that I was working with numerical values, I needed to convert each value into a float so it would be usable in regression analysis. Additionally, it was at this step that I created my variable of interest PBI.no to remove bias introduced by the winrate differential being included in the initial PBI calculation. Finally, I performed a series of sanity checks to ensure that my dataset was properly imported. The dataset passed the sanity checks, so I was able to convert it into a CSV and save it.

### Step 3: Data Visualization
In this step, I used Seaborn's graph functionality to create visualizations for my data. These graphs both serve to verify the validity of the research while also confirming/affirming qualities of the game. 

<img src="https://github.com/olivern9274/Dominant-Strategies-Across-Tiers-of-Play/blob/main/Graphs/Games%20Across%20Ranks.png" width="800">
The vast majority of games are played in the lower ranks. Master to Challenger are considered the "apex" ranks and are the most difficult to achieve due to the exclusivity. Diamond is considered the "gate" before you can enter the apex ranks and this is reflected in the graph as the number of games played takes a steep decline.

<img src="https://github.com/olivern9274/Dominant-Strategies-Across-Tiers-of-Play/blob/main/Graphs/Lane%20Win%20Rates.png" width="800">
The median win rate for every lane hovers around 50% which tracks given how League does not want to have a higher win rate for one particular role over the other. However bottom lane has the highest range of win rates across every rank. Overall community sentiment over the bottom lane is that it has the highest volatility out of the five roles, so this graph tracks with that concensus.

<img src="https://github.com/olivern9274/Dominant-Strategies-Across-Tiers-of-Play/blob/main/Graphs/PBIvsWR.png" wifth="800">
There is a clear positive trend between PBI and win rate which makes sense. A character is more favored in the meta when they achieve higher win rates, so naturally they should be highly correlated. The important finding is in how this changes across ranks. Visually, it is difficult to see and make judgements on. However once we run our regression analysis, this relationship will become more clear. 

The bowtie shape of the graph was strange, but it did not interfere with the obtained results. This shape is still present in the PBI calculation with win rate included, so it is not a result of any data transformations. Rather, it is a quirk of the data where there is a high clustering of characters around 0 PBI as a result of of many "neutral characters" that have a winrate close to the average winrate of that rank.

### Step 4: Regression Analysis (in R)
*Refer to project file "Part 4"*
After importing my dataset into R, I used three different OLS regressions to analyze and interpret the data. My first regression was a simple regression where I simply using PBIno across every rank as my independent variable and wr as my dependent. The second is my preferred regression that included an interaction term for ranking so I can build comparisons between each rank. The third included an additional filter only comparing characters who are primarily in the "top lane". This was mostly for personal curiosity as it is my primary role whenever I play the game. For each of the three OLS models, I calculated heteroskedasticity-consistent (HC1) standard errors by taking the square root of the diagonal elements of the robust variance-covariance matrix. This corrects the standard errors for potential heteroskedasticity in the residuals, without altering the coefficient estimates themselves. The results were all saved to the regression table below.


<table style="text-align:center"><caption><strong>The Effect of PBI on Win Rate Across Ranks</strong></caption>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="3">Win Rate</td></tr>
<tr><td style="text-align:left"></td><td>OLS</td><td>Across Every Lane</td><td>Top Lane Only</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td><td>(3)</td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Challenger WR</td><td></td><td>8.995<sup>***</sup></td><td>9.006<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.295)</td><td>(0.311)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Diamond WR</td><td></td><td>5.795<sup>***</sup></td><td>5.744<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.155)</td><td>(0.140)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Emerald WR</td><td></td><td>4.321<sup>***</sup></td><td>4.276<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.155)</td><td>(0.139)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Grandmaster WR</td><td></td><td>5.992<sup>***</sup></td><td>5.961<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.229)</td><td>(0.216)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Gold WR</td><td></td><td>2.596<sup>***</sup></td><td>2.556<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.220)</td><td>(0.195)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Iron WR</td><td></td><td>-5.142<sup>***</sup></td><td>-5.213<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.233)</td><td>(0.210)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Master WR</td><td></td><td>4.751<sup>***</sup></td><td>4.676<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.170)</td><td>(0.159)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Platinum WR</td><td></td><td>3.365<sup>***</sup></td><td>3.290<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.189)</td><td>(0.168)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Silver WR</td><td></td><td>1.503<sup>***</sup></td><td>1.443<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.221)</td><td>(0.199)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">PBI</td><td>0.002<sup>*</sup></td><td>-0.273<sup>***</sup></td><td>-0.252<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.001)</td><td>(0.055)</td><td>(0.050)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left"># of Games</td><td>-0.00000</td><td>0.00000<sup>***</sup></td><td>0.00000<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.00000)</td><td>(0.00000)</td><td>(0.00000)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">PBI * Challenger</td><td></td><td>0.460<sup>***</sup></td><td>0.689<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.081)</td><td>(0.105)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">PBI * Diamond</td><td></td><td>0.610<sup>***</sup></td><td>0.566<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.075)</td><td>(0.072)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">PBI * Emerald</td><td></td><td>0.426<sup>***</sup></td><td>0.397<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.058)</td><td>(0.053)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">PBI * Grandmaster</td><td></td><td>0.290<sup>***</sup></td><td>0.373<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.068)</td><td>(0.058)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">PBI * Gold</td><td></td><td>0.273<sup>***</sup></td><td>0.251<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.055)</td><td>(0.050)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">PBI * Iron</td><td></td><td>-0.131</td><td>-0.119</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.115)</td><td>(0.133)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">PBI * Master</td><td></td><td>0.344<sup>***</sup></td><td>0.359<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.059)</td><td>(0.054)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">PBI * Platinum</td><td></td><td>0.291<sup>***</sup></td><td>0.268<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.055)</td><td>(0.050)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">PBI * Silver</td><td></td><td>0.268<sup>***</sup></td><td>0.247<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.055)</td><td>(0.050)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>50.257<sup>***</sup></td><td>46.814<sup>***</sup></td><td>46.886<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.150)</td><td>(0.142)</td><td>(0.132)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Rank Interaction</td><td>No</td><td>Yes</td><td>Yes</td></tr>
<tr><td style="text-align:left">Observations</td><td>1,702</td><td>1,702</td><td>1,309</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.001</td><td>0.796</td><td>0.829</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>-0.001</td><td>0.794</td><td>0.827</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>4.245 (df = 1699)</td><td>1.927 (df = 1681)</td><td>1.777 (df = 1288)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>0.509 (df = 2; 1699)</td><td>328.481<sup>***</sup> (df = 20; 1681)</td><td>312.885<sup>***</sup> (df = 20; 1288)</td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="3" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
<tr><td style="text-align:left"></td><td colspan="3" style="text-align:right">*Reference Group is Bronze Rank*</td></tr>
</table>
