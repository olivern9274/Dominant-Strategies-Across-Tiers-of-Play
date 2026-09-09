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
After importing my dataset into R, I used three different OLS regressions to analyze and interpret the data. My first regression was a simple regression where I simply using PBIno across every rank as my independent variable and wr as my dependent. The second is my preferred regression that included an interaction term for ranking so I can build comparisons between each rank. The third included an additional filter only comparing characters who are primarily in the "top lane". This was mostly for personal curiosity as it is my primary role whenever I play the game. For each of the three OLS models, I calculated heteroskedasticity-consistent (HC1) standard errors by taking the square root of the diagonal elements of the robust variance-covariance matrix. This corrects the standard errors for potential heteroskedasticity in the residuals, without altering the coefficient estimates themselves. Using Stargazer, I compiled the results of the regressions with my newly calculated roboust standard errors below into the table below.

<img width="421" height="869" alt="image" src="https://github.com/user-attachments/assets/c86df0a0-309c-4ebb-93dc-d80962e7555c" />

### Findings
These regressions show the effect of meta-preferred characters on win rate. My base model (1) simply sees the effect of PBI on win rate without considering rank and controlling for the number of games. These results are only statistically significant at the 10% level. However my most interesting results were from the other two models where I added interaction terms with rank and PBI.

What my second model shows is that the strongest effect of PBI on winrate shows itself in the Diamond rank with a single PBI increase results in a 0.610 percentage point (pp) increase in winrate compared to Bronze rank on average. Master (the next highest rank) on the other hand, only sees a 0.344 pp increase on average, showing a tapering off of the effect of PBI on winrate. This effect does pick up in Challenger rank with a 0.460 pp increase on average, but it does tell a story about the ranked system. The jump between Diamond and the "apex ranks" is considered one of the biggest in the community. Overall skill levels increase significantly as the highest ranks are more exclusive than any other. Generally speaking, Diamond players often struggle with more fundamental aspects of the game while having solid mechanical skill. To make up the difference, Diamond players rely more on stronger character picks to climb. The highest ranking players generally have higher mechanical skill as well as stronger fundamentals, so they may not need to rely on these strong picks. Challenger having a slight pick up in PBI's effect also makes sense when you consider how much more competitive it is. Within the exclusive apex ranks, Challenger is the one with the hardest cutoff. Only allowing 200-300 players at a given time. With the increased competitiveness, any slight advantage gained in character choice makes all the difference. 

Ranks below Diamond have progressively weaker effects of PBI, until Bronze and Iron where it actually begins to have a negative effect. With Bronze having an increase in PBI ressult in a 0.273 pp decrease in winrate on average. So picking “strong” picks can actually be detrimental in the lowest ranks. Low ranking players often have many fundamental issues that can't be remedied by a strong character choice. If a player is choosing a character simply because it is strong without understanding what makes it a good choice, it can easily result in negative outcomes as they fail to meaningfully contribute to their team.

As we look at the filtered regression that only looks at the top lane, these dynamics change. Rather than the effect tapering off as ranks increase, we see the strongest effect at Challenger. It has the highest increase in winrate with a 0.689 pp increase. Top lane is unique from the other lanes where it is oftentimes seperated from what the rest of the team is doing. Therefore, individual performance matters more significantly than other lanes. This exacerbates the issue of increased competitiveness that comes with higher ranks. So picking stronger characters matters even more when compared to all other lanes.  

### Future Works
In the future, I would like to revisit this project when I become more proficient in Python. This project served as my introduction to Python and webscraping so I am entirely sure that there are many ways to optimize my code to avoid redundancies and to more efficiently collect data. Additionally, I believe that there are additional variables or controls I would be able to implement into future analysis that LoLalytics did not have. While PBI is a solid approximation of character strength, the strange shape of the data can potentially be of concern. Other datasets or websites could potentially hold more prudent data that can address bias concerns. If I were able to pull match data directly from League of Legend's servers, I can circumvent the issue of lack data altogether. 

As this project was primarily an application of Python rather than a formal implementation of econometric techniques, I would also like to implement more advanced models rather than simple OLS. As of now, the current model only measures the correlation between the variables rather than establishing a causal relationship. In the future, I believe a Logistic regression model may be more appropriate for approximating this relationship between winrate and character competitiveness. 

*This project was completed by me with assistance from Ahmad Alexander for ECO590, Data Analytics (R and Python).*
