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

### Key Variables
#### Pick Ban Influence Index:
**PBI.Index** - A measure of character competitiveness using a ratio of pick rate to ban rate multiplied by the difference in winrate in a rank and the average winrate across ranks. 
**PBI.no** - The PBI index measurement with winrate removed from the calculation. 

#### Winrate:
**wr** - How often a character choice results in a victory when picked. 
