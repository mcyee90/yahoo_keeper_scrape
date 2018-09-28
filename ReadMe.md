

```python
# Dependencies
import pandas as pd
from splinter import Browser
from bs4 import BeautifulSoup
import requests
import numpy as np
import time
import json
pd.options.display.max_rows = 999
```


```python
# Create Manager Dataframe to link Manager Name and Current Team Name.
# Had to hardcode manager name because league settings on this are private. 
# If order changes, this section will need to be revisited.

managers_url = 'https://football.fantasysports.yahoo.com/f1/528229/teams'
executable_path = {'executable_path': 'chromedriver.exe'}
browser = Browser('chrome', **executable_path, headless = True)
browser.visit(managers_url)
managers_html = browser.html
managers_soup = BeautifulSoup(managers_html, "html.parser")
managers_table = managers_soup.findAll("tbody")[1]
managers = managers_table.findAll("tr")
team_names = []
for m in managers:
    manager = m.findAll('td')
    team_name = manager[0].text
    team_names.append(team_name)
manager_names = ["Matt", "Ron", "Doug", "Chi Shing", "Evan", "Rajiv", 
                 "Jiwei", "Jake", "Sean", "Ryan", "Andrew", "Dai"]
manager_list = []
for x in np.arange(len(team_names)):
    manager_list.append({"team_name": team_names[x], "manager": manager_names[x]})
manager_df = pd.DataFrame(manager_list)
manager_df.head(12)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>manager</th>
      <th>team_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Matt</td>
      <td>2 Gurley's 1 Cup</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Ron</td>
      <td>Bilbro Naggins</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Doug</td>
      <td>Bye Week</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Chi Shing</td>
      <td>Chi ShingT's Team</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Evan</td>
      <td>Cry me a Philip</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Rajiv</td>
      <td>FirstRoundFlops</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Jiwei</td>
      <td>G</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Jake</td>
      <td>Hunter Punter</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Sean</td>
      <td>Jake News</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Ryan</td>
      <td>Nags</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Andrew</td>
      <td>Thx Ron and Le'Veon</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Dai</td>
      <td>Wu wears the pants!</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Import draft results. This will serve as the starting point for each scrape
# Todd Gurley II's and Dalvin Cook's values were swapped at draft time, so their values are hardcoded.

executable_path = {'executable_path': 'chromedriver.exe'}
browser = Browser('chrome', **executable_path, headless = True)
draft_url = "https://football.fantasysports.yahoo.com/f1/528229/draftresults"
browser.visit(draft_url)
draft_html = browser.html
draft_soup = BeautifulSoup(draft_html, "html.parser")
draft_table = draft_soup.findAll("tbody")[1]
draft_picks = draft_table.findAll("tr")
draft_pick_list = []
for picks in draft_picks:
    pick = picks.findAll("td")
    name = pick[1].text.split(" (")[0]
    team = pick[1].text.split(" (")[1].split(" - ")[0]
    pos = pick[1].text.split(" (")[1].split(" - ")[1].split(")")[0]
    draft_pick_dict = {
        "name": name,
        "NFLTeam": team,
        "Pos": pos,
        "cost": int(pick[2].text[1:]),
        "team": pick[3].text
    }
    draft_pick_list.append(draft_pick_dict)
for player in draft_pick_list:
    if player['name'] == "Todd Gurley II":
        player["cost"] = 40
    if player["name"] == "Dalvin Cook":
        player["cost"] = 64
```


```python
# Create the draft dataframe, as well as keeper column and manager column

draft_df = pd.DataFrame(draft_pick_list)
draft_df['keeper'] = ""
draft_df["manager"] = ""
draft_df.head(10)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NFLTeam</th>
      <th>Pos</th>
      <th>cost</th>
      <th>name</th>
      <th>team</th>
      <th>keeper</th>
      <th>manager</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dal</td>
      <td>QB</td>
      <td>6</td>
      <td>Dak Prescott</td>
      <td>Hunter Punter</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>Pit</td>
      <td>WR</td>
      <td>81</td>
      <td>Antonio Brown</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>NE</td>
      <td>TE</td>
      <td>64</td>
      <td>Rob Gronkowski</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Jake News</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ind</td>
      <td>WR</td>
      <td>51</td>
      <td>T.Y. Hilton</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>5</th>
      <td>NYG</td>
      <td>RB</td>
      <td>74</td>
      <td>Saquon Barkley</td>
      <td>Bye Week</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>6</th>
      <td>Buf</td>
      <td>RB</td>
      <td>29</td>
      <td>LeSean McCoy</td>
      <td>Nags</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>8</th>
      <td>Hou</td>
      <td>RB</td>
      <td>28</td>
      <td>Lamar Miller</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>9</th>
      <td>Cin</td>
      <td>WR</td>
      <td>67</td>
      <td>A.J. Green</td>
      <td>Hunter Punter</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
# Import Keeper CSV
# B denotes keeper for just this past year.
# AB denotes keeper for past two years.

keeper_df = pd.read_csv('keeper.csv')
keeper_df.head(15)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>player</th>
      <th>duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dalvin Cook</td>
      <td>B</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Todd Gurley II</td>
      <td>AB</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Adam Thielen</td>
      <td>B</td>
    </tr>
    <tr>
      <th>3</th>
      <td>DeAndre Hopkins</td>
      <td>B</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Melvin Gordon III</td>
      <td>AB</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Davante Adams</td>
      <td>AB</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Kenyan Drake</td>
      <td>B</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Golden Tate</td>
      <td>AB</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Corey Clement</td>
      <td>B</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Devonta Freeman</td>
      <td>AB</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Keelan Cole</td>
      <td>B</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Will Fuller V</td>
      <td>B</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Jimmy Garoppolo</td>
      <td>B</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Peyton Barber</td>
      <td>B</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Chris Thompson</td>
      <td>B</td>
    </tr>
  </tbody>
</table>
</div>




```python
# merge keeper dataframe with draft dataframe to include keepers into draft dataframe

for i, d in draft_df.iterrows():
    for index, keeper in keeper_df.iterrows():
        if d['name'] == keeper['player']:
            draft_df.at[i,'keeper'] = keeper['duration']
            
draft_df.head(10)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NFLTeam</th>
      <th>Pos</th>
      <th>cost</th>
      <th>name</th>
      <th>team</th>
      <th>keeper</th>
      <th>manager</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dal</td>
      <td>QB</td>
      <td>6</td>
      <td>Dak Prescott</td>
      <td>Hunter Punter</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>Pit</td>
      <td>WR</td>
      <td>81</td>
      <td>Antonio Brown</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>NE</td>
      <td>TE</td>
      <td>64</td>
      <td>Rob Gronkowski</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Jake News</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ind</td>
      <td>WR</td>
      <td>51</td>
      <td>T.Y. Hilton</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>5</th>
      <td>NYG</td>
      <td>RB</td>
      <td>74</td>
      <td>Saquon Barkley</td>
      <td>Bye Week</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>6</th>
      <td>Buf</td>
      <td>RB</td>
      <td>29</td>
      <td>LeSean McCoy</td>
      <td>Nags</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>8</th>
      <td>Hou</td>
      <td>RB</td>
      <td>28</td>
      <td>Lamar Miller</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>9</th>
      <td>Cin</td>
      <td>WR</td>
      <td>67</td>
      <td>A.J. Green</td>
      <td>Hunter Punter</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
# merge manager name with draft picks dataframe

for i, d in draft_df.iterrows():
    for index, team in manager_df.iterrows():
        if d["team"] == team["team_name"]:
            draft_df.at[i, "manager"] = team["manager"]
draft_df.head(15)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NFLTeam</th>
      <th>Pos</th>
      <th>cost</th>
      <th>name</th>
      <th>team</th>
      <th>keeper</th>
      <th>manager</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dal</td>
      <td>QB</td>
      <td>6</td>
      <td>Dak Prescott</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Pit</td>
      <td>WR</td>
      <td>81</td>
      <td>Antonio Brown</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NE</td>
      <td>TE</td>
      <td>64</td>
      <td>Rob Gronkowski</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ind</td>
      <td>WR</td>
      <td>51</td>
      <td>T.Y. Hilton</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>5</th>
      <td>NYG</td>
      <td>RB</td>
      <td>74</td>
      <td>Saquon Barkley</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Buf</td>
      <td>RB</td>
      <td>29</td>
      <td>LeSean McCoy</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Hou</td>
      <td>RB</td>
      <td>28</td>
      <td>Lamar Miller</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Cin</td>
      <td>WR</td>
      <td>67</td>
      <td>A.J. Green</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Atl</td>
      <td>WR</td>
      <td>85</td>
      <td>Julio Jones</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Ari</td>
      <td>WR</td>
      <td>41</td>
      <td>Larry Fitzgerald</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Phi</td>
      <td>RB</td>
      <td>34</td>
      <td>Jay Ajayi</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>13</th>
      <td>KC</td>
      <td>WR</td>
      <td>60</td>
      <td>Tyreek Hill</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>14</th>
      <td>LAC</td>
      <td>WR</td>
      <td>71</td>
      <td>Keenan Allen</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Create a list of transactions by scraping yahoo fantasy sheet for all transactions.
# Value returned as multiple lines so had to split out and remove any injury statuses.
# Placed values in dictionary and merged all the dictionaries.
# Entries go from newest to oldest.

transaction_pages = np.arange(5)*25
transaction_list = []
for page in transaction_pages:
    try:
        transaction_url = 'https://football.fantasysports.yahoo.com/f1/528229/transactions?transactionsfilter=all&count=' + str(page)
        executable_path = {'executable_path': 'chromedriver.exe'}
        browser = Browser('chrome', **executable_path, headless = True)
        browser.visit(transaction_url)
        transaction_html = browser.html
        transaction_soup = BeautifulSoup(transaction_html, "html.parser")
        transaction_table = transaction_soup.findAll("tbody")[1]
        transactions = transaction_table.findAll("tr")
        for t in transactions:
            transaction = t.findAll('td')
            transaction_info = transaction[1].text.splitlines()
            team_info = transaction[2].text.splitlines()
            try:
                try:
                    transaction_info.remove("Q")
                except:
                    pass
                try:
                    transaction_info.remove("D")
                except:
                    pass
                try:
                    transaction_info.remove("O")
                except:
                    pass
                try:
                    transaction_info.remove("NA")
                except:
                    pass
                if transaction_info[3].strip()[0] == "$":
                    price = int(transaction_info[3].strip().split("  ")[0][1:])
                else:
                    price = int(0)
                tm = team_info[3].strip()
                for i, t in manager_df.iterrows():
                    if t['team_name'] == tm:
                        transaction_dict = {
                            "add": transaction_info[1].strip(),
                            "nfl_team": transaction_info[2].strip().split(" - ")[0],
                            "position": transaction_info[2].strip().split(" - ")[1],
                            "price":price,
                            "drop":transaction_info[4].strip(),
                            "ffteam": t['manager'],
                            "type": "waiver"
                        }
                        transaction_list.append(transaction_dict)
            except:
                pass
    except:
        pass


```


```python
# manually code in trade. This will be for reference only.
# Index manually coded as well so that it will appear in the correct order if the entire dataframe is viewed.
# Created index  for draft position for traded players.

trade_list = []
trade00a = {"index": 53.3,
           "add": "Marquise Goodwin",
           "nfl_team": "SF",
           "position": "WR",
           "price":5,
           "drop":"Adrian Peterson",
           "ffteam":"Matt",
           "type": "trade"}
trade00b = {"index": 53.6,
           "add": "Adrian Peterson",
           "nfl_team": "WAS",
           "position": "RB",
           "price":12,
           "drop":"Marquise Goodwin",
           "ffteam":"Jake",
           "type": "trade"}
trade00c = {"index": 66.3,
           "add": "Sony Michel",
           "nfl_team": "NE",
           "position": "RB",
           "price":7,
           "drop":"Geronimo Allison",
           "ffteam":"Ron",
           "type": "trade"}
trade00d = {"index": 66.6,
           "add": "Geronimo Allison",
           "nfl_team": "GB",
           "position": "WR",
           "price":0,
           "drop":"Sony Michel",
           "ffteam":"Andrew",
           "type": "trade"}
trade_list.append(trade00a)
trade_list.append(trade00b)
trade_list.append(trade00c)
trade_list.append(trade00d)

```


```python
# flipped data so that oldest appears first and newest appears last for all waiver transactions, not trades.

transaction_df = pd.DataFrame(transaction_list)
transaction_df = transaction_df.reindex(index=transaction_df.index[::-1]).reset_index(drop=True)
transaction_df = transaction_df.drop([48, 49]).reset_index(drop=True)
transaction_df.head(200)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>add</th>
      <th>drop</th>
      <th>ffteam</th>
      <th>nfl_team</th>
      <th>position</th>
      <th>price</th>
      <th>type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>James White</td>
      <td>Pittsburgh</td>
      <td>Jake</td>
      <td>NE</td>
      <td>RB</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mason Crosby</td>
      <td>Baker Mayfield</td>
      <td>Matt</td>
      <td>GB</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>2</th>
      <td>John Ross</td>
      <td>Dez Bryant</td>
      <td>Dai</td>
      <td>Cin</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Austin Hooper</td>
      <td>Charles Clay</td>
      <td>Dai</td>
      <td>Atl</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Graham Gano</td>
      <td>Donte Moncrief</td>
      <td>Sean</td>
      <td>Car</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Allen Hurns</td>
      <td>LeGarrette Blount</td>
      <td>Ron</td>
      <td>Dal</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Quincy Enunwa</td>
      <td>Denver</td>
      <td>Ron</td>
      <td>NYJ</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Jaylen Samuels</td>
      <td>Austin Hooper</td>
      <td>Dai</td>
      <td>Pit</td>
      <td>RB,TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Geronimo Allison</td>
      <td>Dede Westbrook</td>
      <td>Ron</td>
      <td>GB</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Dede Westbrook</td>
      <td>Matt Prater</td>
      <td>Jiwei</td>
      <td>Jax</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Brandon McManus</td>
      <td>Jaylen Samuels</td>
      <td>Dai</td>
      <td>Den</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Phillip Lindsay</td>
      <td>Matt Breida</td>
      <td>Matt</td>
      <td>Den</td>
      <td>RB</td>
      <td>15</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>12</th>
      <td>T.J. Yeldon</td>
      <td>Delanie Walker</td>
      <td>Sean</td>
      <td>Jax</td>
      <td>RB</td>
      <td>11</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Austin Ekeler</td>
      <td>Duke Johnson Jr.</td>
      <td>Sean</td>
      <td>LAC</td>
      <td>RB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Ted Ginn Jr.</td>
      <td>Jameis Winston</td>
      <td>Sean</td>
      <td>NO</td>
      <td>WR</td>
      <td>4</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Antonio Gates</td>
      <td>Cameron Brate</td>
      <td>Chi Shing</td>
      <td>LAC</td>
      <td>TE</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Darren Sproles</td>
      <td>Houston</td>
      <td>Chi Shing</td>
      <td>Phi</td>
      <td>RB</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>17</th>
      <td>DeSean Jackson</td>
      <td>DeMarco Murray</td>
      <td>Jake</td>
      <td>TB</td>
      <td>WR</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Denver</td>
      <td>New Orleans</td>
      <td>Sean</td>
      <td>Den</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Dante Pettis</td>
      <td>D.J. Moore</td>
      <td>Sean</td>
      <td>SF</td>
      <td>WR</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Tyrod Taylor</td>
      <td>Mike Wallace</td>
      <td>Jiwei</td>
      <td>Cle</td>
      <td>QB</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Trent Taylor</td>
      <td>Josh Doctson</td>
      <td>Ron</td>
      <td>SF</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Adam Vinatieri</td>
      <td>Graham Gano</td>
      <td>Sean</td>
      <td>Ind</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Brandon Marshall</td>
      <td>Brandon McManus</td>
      <td>Dai</td>
      <td>Sea</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Seattle</td>
      <td>Green Bay</td>
      <td>Ron</td>
      <td>Sea</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Ryan Grant</td>
      <td>Trent Taylor</td>
      <td>Ron</td>
      <td>Ind</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Phillip Dorsett</td>
      <td>Greg Olsen</td>
      <td>Chi Shing</td>
      <td>NE</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Javorius Allen</td>
      <td>Ben Roethlisberger</td>
      <td>Dai</td>
      <td>Bal</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Matt Breida</td>
      <td>Allen Hurns</td>
      <td>Ron</td>
      <td>SF</td>
      <td>RB</td>
      <td>11</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Houston</td>
      <td>Seattle</td>
      <td>Ron</td>
      <td>Hou</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Ben Roethlisberger</td>
      <td>Jared Goff</td>
      <td>Ron</td>
      <td>Pit</td>
      <td>QB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Jalen Richard</td>
      <td>John Ross</td>
      <td>Dai</td>
      <td>Oak</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Jonnu Smith</td>
      <td>Austin Seferian-Jenkins</td>
      <td>Ron</td>
      <td>Ten</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Antonio Callaway</td>
      <td>Cameron Meredith</td>
      <td>Andrew</td>
      <td>Cle</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Will Dissly</td>
      <td>Jordy Nelson</td>
      <td>Chi Shing</td>
      <td>Sea</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Cody Parkey</td>
      <td>Jalen Richard</td>
      <td>Dai</td>
      <td>Chi</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Ryan Fitzpatrick</td>
      <td>Ted Ginn Jr.</td>
      <td>Sean</td>
      <td>TB</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Giovani Bernard</td>
      <td>Ryan Grant</td>
      <td>Ron</td>
      <td>Cin</td>
      <td>RB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Dan Bailey</td>
      <td>Daniel Carlson</td>
      <td>Ryan</td>
      <td>Min</td>
      <td>K</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Jesse James</td>
      <td>Dante Pettis</td>
      <td>Sean</td>
      <td>Pit</td>
      <td>TE</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Jared Goff</td>
      <td>Marcus Mariota</td>
      <td>Matt</td>
      <td>LAR</td>
      <td>QB</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Dallas</td>
      <td>Denver</td>
      <td>Sean</td>
      <td>Dal</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Josh Lambo</td>
      <td>Adam Vinatieri</td>
      <td>Sean</td>
      <td>Jax</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Tyler Boyd</td>
      <td>Jonnu Smith</td>
      <td>Ron</td>
      <td>Cin</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Mark Walton</td>
      <td>Cody Parkey</td>
      <td>Dai</td>
      <td>Cin</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Malcolm Brown</td>
      <td>Brandon Marshall</td>
      <td>Dai</td>
      <td>LAR</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>46</th>
      <td>Frank Gore</td>
      <td>Antonio Gates</td>
      <td>Chi Shing</td>
      <td>Mia</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Cleveland</td>
      <td>Carolina</td>
      <td>Chi Shing</td>
      <td>Cle</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Ted Ginn Jr.</td>
      <td>Michael Gallup</td>
      <td>Doug</td>
      <td>NO</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Ito Smith</td>
      <td>Alex Smith</td>
      <td>Ron</td>
      <td>Atl</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>50</th>
      <td>Nyheim Hines</td>
      <td>Ronald Jones II</td>
      <td>Andrew</td>
      <td>Ind</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Theo Riddick</td>
      <td>Darren Sproles</td>
      <td>Chi Shing</td>
      <td>Det</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Chandler Catanzaro</td>
      <td>Greg Zuerlein</td>
      <td>Jiwei</td>
      <td>TB</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>53</th>
      <td>Matt Prater</td>
      <td>Corey Grant</td>
      <td>Dai</td>
      <td>Det</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>54</th>
      <td>Christian Kirk</td>
      <td>Mark Walton</td>
      <td>Dai</td>
      <td>Ari</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>55</th>
      <td>Ronald Jones II</td>
      <td>Jamison Crowder</td>
      <td>Dai</td>
      <td>TB</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>56</th>
      <td>Jordy Nelson</td>
      <td>Tyrod Taylor</td>
      <td>Jiwei</td>
      <td>Oak</td>
      <td>WR</td>
      <td>7</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>57</th>
      <td>Ryan Tannehill</td>
      <td>Doug Martin</td>
      <td>Doug</td>
      <td>Mia</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>58</th>
      <td>Baker Mayfield</td>
      <td>Austin Ekeler</td>
      <td>Sean</td>
      <td>Cle</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>59</th>
      <td>Chris Ivory</td>
      <td>Spencer Ware</td>
      <td>Jiwei</td>
      <td>Buf</td>
      <td>RB</td>
      <td>5</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>60</th>
      <td>Dallas Goedert</td>
      <td>Jesse James</td>
      <td>Sean</td>
      <td>Phi</td>
      <td>TE</td>
      <td>4</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>61</th>
      <td>Austin Hooper</td>
      <td>Anthony Miller</td>
      <td>Andrew</td>
      <td>Atl</td>
      <td>TE</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>62</th>
      <td>Seattle</td>
      <td>Dallas</td>
      <td>Sean</td>
      <td>Sea</td>
      <td>DEF</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>63</th>
      <td>Jordan Matthews</td>
      <td>Jimmy Garoppolo</td>
      <td>Chi Shing</td>
      <td>Phi</td>
      <td>WR</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>64</th>
      <td>Andy Dalton</td>
      <td>Ito Smith</td>
      <td>Ron</td>
      <td>Cin</td>
      <td>QB</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>65</th>
      <td>Green Bay</td>
      <td>Minnesota</td>
      <td>Dai</td>
      <td>GB</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>66</th>
      <td>Tennessee</td>
      <td>Houston</td>
      <td>Ron</td>
      <td>Ten</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Albert Wilson</td>
      <td>Pierre Garcon</td>
      <td>Doug</td>
      <td>Mia</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>68</th>
      <td>Rhett Ellison</td>
      <td>Tyler Eifert</td>
      <td>Doug</td>
      <td>NYG</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>69</th>
      <td>Vance McDonald</td>
      <td>Rex Burkhead</td>
      <td>Jiwei</td>
      <td>Pit</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>70</th>
      <td>Tavon Austin</td>
      <td>Rishard Matthews</td>
      <td>Ryan</td>
      <td>Dal</td>
      <td>WR,RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>71</th>
      <td>Randy Bullock</td>
      <td>Josh Lambo</td>
      <td>Sean</td>
      <td>Cin</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
  </tbody>
</table>
</div>




```python
# merge transactions and trades. trade will sent index to the index column in the dataframe

trade_df = pd.DataFrame(trade_list)
trade_df.set_index('index', inplace=True)
trade_df.head()
transaction_df = pd.concat([transaction_df, trade_df])
```


```python
# re-order index so that trade appears in the correct order
transaction_df = transaction_df.sort_index()
transaction_df.head(200)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>add</th>
      <th>drop</th>
      <th>ffteam</th>
      <th>nfl_team</th>
      <th>position</th>
      <th>price</th>
      <th>type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0.0</th>
      <td>James White</td>
      <td>Pittsburgh</td>
      <td>Jake</td>
      <td>NE</td>
      <td>RB</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>1.0</th>
      <td>Mason Crosby</td>
      <td>Baker Mayfield</td>
      <td>Matt</td>
      <td>GB</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>2.0</th>
      <td>John Ross</td>
      <td>Dez Bryant</td>
      <td>Dai</td>
      <td>Cin</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>3.0</th>
      <td>Austin Hooper</td>
      <td>Charles Clay</td>
      <td>Dai</td>
      <td>Atl</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>4.0</th>
      <td>Graham Gano</td>
      <td>Donte Moncrief</td>
      <td>Sean</td>
      <td>Car</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>5.0</th>
      <td>Allen Hurns</td>
      <td>LeGarrette Blount</td>
      <td>Ron</td>
      <td>Dal</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>6.0</th>
      <td>Quincy Enunwa</td>
      <td>Denver</td>
      <td>Ron</td>
      <td>NYJ</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>7.0</th>
      <td>Jaylen Samuels</td>
      <td>Austin Hooper</td>
      <td>Dai</td>
      <td>Pit</td>
      <td>RB,TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>8.0</th>
      <td>Geronimo Allison</td>
      <td>Dede Westbrook</td>
      <td>Ron</td>
      <td>GB</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>9.0</th>
      <td>Dede Westbrook</td>
      <td>Matt Prater</td>
      <td>Jiwei</td>
      <td>Jax</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>10.0</th>
      <td>Brandon McManus</td>
      <td>Jaylen Samuels</td>
      <td>Dai</td>
      <td>Den</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>11.0</th>
      <td>Phillip Lindsay</td>
      <td>Matt Breida</td>
      <td>Matt</td>
      <td>Den</td>
      <td>RB</td>
      <td>15</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>12.0</th>
      <td>T.J. Yeldon</td>
      <td>Delanie Walker</td>
      <td>Sean</td>
      <td>Jax</td>
      <td>RB</td>
      <td>11</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>13.0</th>
      <td>Austin Ekeler</td>
      <td>Duke Johnson Jr.</td>
      <td>Sean</td>
      <td>LAC</td>
      <td>RB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>14.0</th>
      <td>Ted Ginn Jr.</td>
      <td>Jameis Winston</td>
      <td>Sean</td>
      <td>NO</td>
      <td>WR</td>
      <td>4</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>15.0</th>
      <td>Antonio Gates</td>
      <td>Cameron Brate</td>
      <td>Chi Shing</td>
      <td>LAC</td>
      <td>TE</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>16.0</th>
      <td>Darren Sproles</td>
      <td>Houston</td>
      <td>Chi Shing</td>
      <td>Phi</td>
      <td>RB</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>17.0</th>
      <td>DeSean Jackson</td>
      <td>DeMarco Murray</td>
      <td>Jake</td>
      <td>TB</td>
      <td>WR</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>18.0</th>
      <td>Denver</td>
      <td>New Orleans</td>
      <td>Sean</td>
      <td>Den</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>19.0</th>
      <td>Dante Pettis</td>
      <td>D.J. Moore</td>
      <td>Sean</td>
      <td>SF</td>
      <td>WR</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>20.0</th>
      <td>Tyrod Taylor</td>
      <td>Mike Wallace</td>
      <td>Jiwei</td>
      <td>Cle</td>
      <td>QB</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>21.0</th>
      <td>Trent Taylor</td>
      <td>Josh Doctson</td>
      <td>Ron</td>
      <td>SF</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>22.0</th>
      <td>Adam Vinatieri</td>
      <td>Graham Gano</td>
      <td>Sean</td>
      <td>Ind</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>23.0</th>
      <td>Brandon Marshall</td>
      <td>Brandon McManus</td>
      <td>Dai</td>
      <td>Sea</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>24.0</th>
      <td>Seattle</td>
      <td>Green Bay</td>
      <td>Ron</td>
      <td>Sea</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>25.0</th>
      <td>Ryan Grant</td>
      <td>Trent Taylor</td>
      <td>Ron</td>
      <td>Ind</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>26.0</th>
      <td>Phillip Dorsett</td>
      <td>Greg Olsen</td>
      <td>Chi Shing</td>
      <td>NE</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>27.0</th>
      <td>Javorius Allen</td>
      <td>Ben Roethlisberger</td>
      <td>Dai</td>
      <td>Bal</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>28.0</th>
      <td>Matt Breida</td>
      <td>Allen Hurns</td>
      <td>Ron</td>
      <td>SF</td>
      <td>RB</td>
      <td>11</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>29.0</th>
      <td>Houston</td>
      <td>Seattle</td>
      <td>Ron</td>
      <td>Hou</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>30.0</th>
      <td>Ben Roethlisberger</td>
      <td>Jared Goff</td>
      <td>Ron</td>
      <td>Pit</td>
      <td>QB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>31.0</th>
      <td>Jalen Richard</td>
      <td>John Ross</td>
      <td>Dai</td>
      <td>Oak</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>32.0</th>
      <td>Jonnu Smith</td>
      <td>Austin Seferian-Jenkins</td>
      <td>Ron</td>
      <td>Ten</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>33.0</th>
      <td>Antonio Callaway</td>
      <td>Cameron Meredith</td>
      <td>Andrew</td>
      <td>Cle</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>34.0</th>
      <td>Will Dissly</td>
      <td>Jordy Nelson</td>
      <td>Chi Shing</td>
      <td>Sea</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>35.0</th>
      <td>Cody Parkey</td>
      <td>Jalen Richard</td>
      <td>Dai</td>
      <td>Chi</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>36.0</th>
      <td>Ryan Fitzpatrick</td>
      <td>Ted Ginn Jr.</td>
      <td>Sean</td>
      <td>TB</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>37.0</th>
      <td>Giovani Bernard</td>
      <td>Ryan Grant</td>
      <td>Ron</td>
      <td>Cin</td>
      <td>RB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>38.0</th>
      <td>Dan Bailey</td>
      <td>Daniel Carlson</td>
      <td>Ryan</td>
      <td>Min</td>
      <td>K</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>39.0</th>
      <td>Jesse James</td>
      <td>Dante Pettis</td>
      <td>Sean</td>
      <td>Pit</td>
      <td>TE</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>40.0</th>
      <td>Jared Goff</td>
      <td>Marcus Mariota</td>
      <td>Matt</td>
      <td>LAR</td>
      <td>QB</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>41.0</th>
      <td>Dallas</td>
      <td>Denver</td>
      <td>Sean</td>
      <td>Dal</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>42.0</th>
      <td>Josh Lambo</td>
      <td>Adam Vinatieri</td>
      <td>Sean</td>
      <td>Jax</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>43.0</th>
      <td>Tyler Boyd</td>
      <td>Jonnu Smith</td>
      <td>Ron</td>
      <td>Cin</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>44.0</th>
      <td>Mark Walton</td>
      <td>Cody Parkey</td>
      <td>Dai</td>
      <td>Cin</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>45.0</th>
      <td>Malcolm Brown</td>
      <td>Brandon Marshall</td>
      <td>Dai</td>
      <td>LAR</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>46.0</th>
      <td>Frank Gore</td>
      <td>Antonio Gates</td>
      <td>Chi Shing</td>
      <td>Mia</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>47.0</th>
      <td>Cleveland</td>
      <td>Carolina</td>
      <td>Chi Shing</td>
      <td>Cle</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>48.0</th>
      <td>Ted Ginn Jr.</td>
      <td>Michael Gallup</td>
      <td>Doug</td>
      <td>NO</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>49.0</th>
      <td>Ito Smith</td>
      <td>Alex Smith</td>
      <td>Ron</td>
      <td>Atl</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>50.0</th>
      <td>Nyheim Hines</td>
      <td>Ronald Jones II</td>
      <td>Andrew</td>
      <td>Ind</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>51.0</th>
      <td>Theo Riddick</td>
      <td>Darren Sproles</td>
      <td>Chi Shing</td>
      <td>Det</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>52.0</th>
      <td>Chandler Catanzaro</td>
      <td>Greg Zuerlein</td>
      <td>Jiwei</td>
      <td>TB</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>53.0</th>
      <td>Matt Prater</td>
      <td>Corey Grant</td>
      <td>Dai</td>
      <td>Det</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>53.3</th>
      <td>Marquise Goodwin</td>
      <td>Adrian Peterson</td>
      <td>Matt</td>
      <td>SF</td>
      <td>WR</td>
      <td>5</td>
      <td>trade</td>
    </tr>
    <tr>
      <th>53.6</th>
      <td>Adrian Peterson</td>
      <td>Marquise Goodwin</td>
      <td>Jake</td>
      <td>WAS</td>
      <td>RB</td>
      <td>12</td>
      <td>trade</td>
    </tr>
    <tr>
      <th>54.0</th>
      <td>Christian Kirk</td>
      <td>Mark Walton</td>
      <td>Dai</td>
      <td>Ari</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>55.0</th>
      <td>Ronald Jones II</td>
      <td>Jamison Crowder</td>
      <td>Dai</td>
      <td>TB</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>56.0</th>
      <td>Jordy Nelson</td>
      <td>Tyrod Taylor</td>
      <td>Jiwei</td>
      <td>Oak</td>
      <td>WR</td>
      <td>7</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>57.0</th>
      <td>Ryan Tannehill</td>
      <td>Doug Martin</td>
      <td>Doug</td>
      <td>Mia</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>58.0</th>
      <td>Baker Mayfield</td>
      <td>Austin Ekeler</td>
      <td>Sean</td>
      <td>Cle</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>59.0</th>
      <td>Chris Ivory</td>
      <td>Spencer Ware</td>
      <td>Jiwei</td>
      <td>Buf</td>
      <td>RB</td>
      <td>5</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>60.0</th>
      <td>Dallas Goedert</td>
      <td>Jesse James</td>
      <td>Sean</td>
      <td>Phi</td>
      <td>TE</td>
      <td>4</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>61.0</th>
      <td>Austin Hooper</td>
      <td>Anthony Miller</td>
      <td>Andrew</td>
      <td>Atl</td>
      <td>TE</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>62.0</th>
      <td>Seattle</td>
      <td>Dallas</td>
      <td>Sean</td>
      <td>Sea</td>
      <td>DEF</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>63.0</th>
      <td>Jordan Matthews</td>
      <td>Jimmy Garoppolo</td>
      <td>Chi Shing</td>
      <td>Phi</td>
      <td>WR</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>64.0</th>
      <td>Andy Dalton</td>
      <td>Ito Smith</td>
      <td>Ron</td>
      <td>Cin</td>
      <td>QB</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>65.0</th>
      <td>Green Bay</td>
      <td>Minnesota</td>
      <td>Dai</td>
      <td>GB</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>66.0</th>
      <td>Tennessee</td>
      <td>Houston</td>
      <td>Ron</td>
      <td>Ten</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>66.3</th>
      <td>Sony Michel</td>
      <td>Geronimo Allison</td>
      <td>Ron</td>
      <td>NE</td>
      <td>RB</td>
      <td>7</td>
      <td>trade</td>
    </tr>
    <tr>
      <th>66.6</th>
      <td>Geronimo Allison</td>
      <td>Sony Michel</td>
      <td>Andrew</td>
      <td>GB</td>
      <td>WR</td>
      <td>0</td>
      <td>trade</td>
    </tr>
    <tr>
      <th>67.0</th>
      <td>Albert Wilson</td>
      <td>Pierre Garcon</td>
      <td>Doug</td>
      <td>Mia</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>68.0</th>
      <td>Rhett Ellison</td>
      <td>Tyler Eifert</td>
      <td>Doug</td>
      <td>NYG</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>69.0</th>
      <td>Vance McDonald</td>
      <td>Rex Burkhead</td>
      <td>Jiwei</td>
      <td>Pit</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>70.0</th>
      <td>Tavon Austin</td>
      <td>Rishard Matthews</td>
      <td>Ryan</td>
      <td>Dal</td>
      <td>WR,RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>71.0</th>
      <td>Randy Bullock</td>
      <td>Josh Lambo</td>
      <td>Sean</td>
      <td>Cin</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
  </tbody>
</table>
</div>




```python
# run through total transaction dataframe and draft dataframe swapping out players who are dropped for players who are added.
# additionally, transacation costs, and player information added to draft dataframe
# if the index matches the draft_df for a particular player, then only the manager's name is swapped. This is hard-coded.
for x, t in transaction_df.iterrows():
    for i, d in draft_df.iterrows():
        if t['type'] == "waiver":
            if d['name'] == t['drop']:
                draft_df.at[i, 'name'] = t['add']
                draft_df.at[i, 'NFLTeam'] = t['nfl_team']
                draft_df.at[i, 'Pos'] = t['position']
                draft_df.at[i, 'cost'] = t['price']
                draft_df.at[i, 'keeper'] = ""
        if t["type"] == "trade":
            trade_index = draft_df.index[draft_df['name']==t['add']]
            draft_df.at[trade_index, "manager"] = t['ffteam']
roster_df = draft_df

```


```python
roster_df.head(200)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NFLTeam</th>
      <th>Pos</th>
      <th>cost</th>
      <th>name</th>
      <th>team</th>
      <th>keeper</th>
      <th>manager</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dal</td>
      <td>QB</td>
      <td>6</td>
      <td>Dak Prescott</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Pit</td>
      <td>WR</td>
      <td>81</td>
      <td>Antonio Brown</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NE</td>
      <td>TE</td>
      <td>64</td>
      <td>Rob Gronkowski</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ind</td>
      <td>WR</td>
      <td>51</td>
      <td>T.Y. Hilton</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>5</th>
      <td>NYG</td>
      <td>RB</td>
      <td>74</td>
      <td>Saquon Barkley</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Buf</td>
      <td>RB</td>
      <td>29</td>
      <td>LeSean McCoy</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Hou</td>
      <td>RB</td>
      <td>28</td>
      <td>Lamar Miller</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Cin</td>
      <td>WR</td>
      <td>67</td>
      <td>A.J. Green</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Atl</td>
      <td>WR</td>
      <td>85</td>
      <td>Julio Jones</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Ari</td>
      <td>WR</td>
      <td>41</td>
      <td>Larry Fitzgerald</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Phi</td>
      <td>RB</td>
      <td>34</td>
      <td>Jay Ajayi</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>13</th>
      <td>KC</td>
      <td>WR</td>
      <td>60</td>
      <td>Tyreek Hill</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>14</th>
      <td>LAC</td>
      <td>WR</td>
      <td>71</td>
      <td>Keenan Allen</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Phi</td>
      <td>WR</td>
      <td>40</td>
      <td>Alshon Jeffery</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>16</th>
      <td>LAR</td>
      <td>WR</td>
      <td>23</td>
      <td>Robert Woods</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>17</th>
      <td>KC</td>
      <td>RB</td>
      <td>79</td>
      <td>Kareem Hunt</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Buf</td>
      <td>WR</td>
      <td>15</td>
      <td>Kelvin Benjamin</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>19</th>
      <td>NE</td>
      <td>WR</td>
      <td>0</td>
      <td>Phillip Dorsett</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Den</td>
      <td>RB</td>
      <td>36</td>
      <td>Royce Freeman</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>21</th>
      <td>NE</td>
      <td>QB</td>
      <td>28</td>
      <td>Tom Brady</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>22</th>
      <td>SF</td>
      <td>RB</td>
      <td>8</td>
      <td>Alfred Morris</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>23</th>
      <td>LAR</td>
      <td>WR</td>
      <td>39</td>
      <td>Brandin Cooks</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>24</th>
      <td>TB</td>
      <td>WR</td>
      <td>2</td>
      <td>DeSean Jackson</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Cin</td>
      <td>RB</td>
      <td>51</td>
      <td>Joe Mixon</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>26</th>
      <td>KC</td>
      <td>TE</td>
      <td>52</td>
      <td>Travis Kelce</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Car</td>
      <td>RB</td>
      <td>67</td>
      <td>Christian McCaffrey</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Phi</td>
      <td>TE</td>
      <td>49</td>
      <td>Zach Ertz</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Chi</td>
      <td>RB</td>
      <td>64</td>
      <td>Jordan Howard</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Atl</td>
      <td>RB</td>
      <td>22</td>
      <td>Tevin Coleman</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>31</th>
      <td>GB</td>
      <td>TE</td>
      <td>23</td>
      <td>Jimmy Graham</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>32</th>
      <td>GB</td>
      <td>QB</td>
      <td>51</td>
      <td>Aaron Rodgers</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Cle</td>
      <td>WR</td>
      <td>38</td>
      <td>Jarvis Landry</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Jax</td>
      <td>DEF</td>
      <td>8</td>
      <td>Jacksonville</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Was</td>
      <td>RB</td>
      <td>12</td>
      <td>Adrian Peterson</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Bal</td>
      <td>WR</td>
      <td>19</td>
      <td>Michael Crabtree</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Ten</td>
      <td>WR</td>
      <td>35</td>
      <td>Corey Davis</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>38</th>
      <td>NO</td>
      <td>RB</td>
      <td>39</td>
      <td>Mark Ingram</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>39</th>
      <td>NE</td>
      <td>WR</td>
      <td>39</td>
      <td>Chris Hogan</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>40</th>
      <td>LAR</td>
      <td>WR</td>
      <td>24</td>
      <td>Cooper Kupp</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Bal</td>
      <td>RB</td>
      <td>56</td>
      <td>Alex Collins</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>42</th>
      <td>TB</td>
      <td>RB</td>
      <td>0</td>
      <td>Ronald Jones II</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>43</th>
      <td>NE</td>
      <td>WR</td>
      <td>19</td>
      <td>Julian Edelman</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Den</td>
      <td>WR</td>
      <td>38</td>
      <td>Demaryius Thomas</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Chi</td>
      <td>TE</td>
      <td>31</td>
      <td>Trey Burton</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>46</th>
      <td>GB</td>
      <td>DEF</td>
      <td>0</td>
      <td>Green Bay</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Sea</td>
      <td>TE</td>
      <td>0</td>
      <td>Will Dissly</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Pit</td>
      <td>TE</td>
      <td>0</td>
      <td>Vance McDonald</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>49</th>
      <td>NO</td>
      <td>QB</td>
      <td>25</td>
      <td>Drew Brees</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>50</th>
      <td>Ten</td>
      <td>RB</td>
      <td>29</td>
      <td>Dion Lewis</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Cle</td>
      <td>RB</td>
      <td>22</td>
      <td>Carlos Hyde</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Car</td>
      <td>QB</td>
      <td>30</td>
      <td>Cam Newton</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>53</th>
      <td>Oak</td>
      <td>RB</td>
      <td>16</td>
      <td>Marshawn Lynch</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>54</th>
      <td>Jax</td>
      <td>WR</td>
      <td>0</td>
      <td>Dede Westbrook</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>55</th>
      <td>TB</td>
      <td>TE</td>
      <td>14</td>
      <td>O.J. Howard</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>56</th>
      <td>Det</td>
      <td>QB</td>
      <td>14</td>
      <td>Matthew Stafford</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>57</th>
      <td>Den</td>
      <td>WR</td>
      <td>18</td>
      <td>Emmanuel Sanders</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>58</th>
      <td>LAR</td>
      <td>DEF</td>
      <td>11</td>
      <td>Los Angeles</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>59</th>
      <td>Mia</td>
      <td>WR</td>
      <td>0</td>
      <td>Albert Wilson</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>60</th>
      <td>NE</td>
      <td>K</td>
      <td>5</td>
      <td>Stephen Gostkowski</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>61</th>
      <td>Cle</td>
      <td>TE</td>
      <td>9</td>
      <td>David Njoku</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>62</th>
      <td>Jax</td>
      <td>RB</td>
      <td>11</td>
      <td>T.J. Yeldon</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>63</th>
      <td>Den</td>
      <td>RB</td>
      <td>15</td>
      <td>Phillip Lindsay</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>64</th>
      <td>LAR</td>
      <td>QB</td>
      <td>2</td>
      <td>Jared Goff</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>65</th>
      <td>Sea</td>
      <td>RB</td>
      <td>25</td>
      <td>Chris Carson</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>66</th>
      <td>NO</td>
      <td>WR</td>
      <td>0</td>
      <td>Ted Ginn Jr.</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Ind</td>
      <td>RB</td>
      <td>0</td>
      <td>Nyheim Hines</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>68</th>
      <td>Was</td>
      <td>TE</td>
      <td>10</td>
      <td>Jordan Reed</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>69</th>
      <td>NYJ</td>
      <td>WR</td>
      <td>14</td>
      <td>Robby Anderson</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>70</th>
      <td>LAC</td>
      <td>DEF</td>
      <td>4</td>
      <td>Los Angeles</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>71</th>
      <td>Oak</td>
      <td>TE</td>
      <td>3</td>
      <td>Jared Cook</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>72</th>
      <td>NYJ</td>
      <td>RB</td>
      <td>14</td>
      <td>Isaiah Crowell</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>73</th>
      <td>NYG</td>
      <td>TE</td>
      <td>0</td>
      <td>Rhett Ellison</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>74</th>
      <td>Car</td>
      <td>WR</td>
      <td>15</td>
      <td>Devin Funchess</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>75</th>
      <td>SF</td>
      <td>TE</td>
      <td>6</td>
      <td>George Kittle</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>76</th>
      <td>Pit</td>
      <td>QB</td>
      <td>0</td>
      <td>Ben Roethlisberger</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Phi</td>
      <td>DEF</td>
      <td>5</td>
      <td>Philadelphia</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>78</th>
      <td>GB</td>
      <td>WR</td>
      <td>0</td>
      <td>Geronimo Allison</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>79</th>
      <td>Atl</td>
      <td>TE</td>
      <td>3</td>
      <td>Austin Hooper</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Sea</td>
      <td>RB</td>
      <td>13</td>
      <td>Rashaad Penny</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>81</th>
      <td>Phi</td>
      <td>WR</td>
      <td>13</td>
      <td>Nelson Agholor</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>82</th>
      <td>Det</td>
      <td>RB</td>
      <td>0</td>
      <td>Theo Riddick</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>83</th>
      <td>Mia</td>
      <td>WR</td>
      <td>5</td>
      <td>DeVante Parker</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>84</th>
      <td>NYJ</td>
      <td>RB</td>
      <td>7</td>
      <td>Bilal Powell</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>85</th>
      <td>Buf</td>
      <td>RB</td>
      <td>5</td>
      <td>Chris Ivory</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>86</th>
      <td>Cle</td>
      <td>QB</td>
      <td>6</td>
      <td>Baker Mayfield</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>87</th>
      <td>Ari</td>
      <td>WR</td>
      <td>0</td>
      <td>Christian Kirk</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>88</th>
      <td>Det</td>
      <td>RB</td>
      <td>10</td>
      <td>Kerryon Johnson</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>89</th>
      <td>TB</td>
      <td>K</td>
      <td>0</td>
      <td>Chandler Catanzaro</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>90</th>
      <td>GB</td>
      <td>RB</td>
      <td>9</td>
      <td>Jamaal Williams</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>91</th>
      <td>Chi</td>
      <td>DEF</td>
      <td>3</td>
      <td>Chicago</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>92</th>
      <td>Ari</td>
      <td>QB</td>
      <td>5</td>
      <td>Josh Rosen</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>93</th>
      <td>NYG</td>
      <td>WR</td>
      <td>4</td>
      <td>Sterling Shepard</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>94</th>
      <td>Bal</td>
      <td>K</td>
      <td>4</td>
      <td>Justin Tucker</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>95</th>
      <td>NE</td>
      <td>RB</td>
      <td>7</td>
      <td>Sony Michel</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Ind</td>
      <td>TE</td>
      <td>4</td>
      <td>Jack Doyle</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>97</th>
      <td>Bal</td>
      <td>DEF</td>
      <td>5</td>
      <td>Baltimore</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>98</th>
      <td>Sea</td>
      <td>DEF</td>
      <td>2</td>
      <td>Seattle</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>99</th>
      <td>SF</td>
      <td>RB</td>
      <td>3</td>
      <td>Jerick McKinnon</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>100</th>
      <td>Pit</td>
      <td>RB</td>
      <td>5</td>
      <td>James Conner</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>101</th>
      <td>NYJ</td>
      <td>WR</td>
      <td>0</td>
      <td>Quincy Enunwa</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>102</th>
      <td>KC</td>
      <td>QB</td>
      <td>3</td>
      <td>Patrick Mahomes</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Cle</td>
      <td>WR</td>
      <td>0</td>
      <td>Antonio Callaway</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>104</th>
      <td>NYJ</td>
      <td>QB</td>
      <td>1</td>
      <td>Sam Darnold</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>105</th>
      <td>Mia</td>
      <td>WR</td>
      <td>3</td>
      <td>Kenny Stills</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>106</th>
      <td>Atl</td>
      <td>K</td>
      <td>2</td>
      <td>Matt Bryant</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>107</th>
      <td>Dal</td>
      <td>WR,RB</td>
      <td>0</td>
      <td>Tavon Austin</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>108</th>
      <td>Ari</td>
      <td>TE</td>
      <td>1</td>
      <td>Ricky Seals-Jones</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>109</th>
      <td>Atl</td>
      <td>QB</td>
      <td>3</td>
      <td>Matt Ryan</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>110</th>
      <td>TB</td>
      <td>WR</td>
      <td>3</td>
      <td>Chris Godwin</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>111</th>
      <td>SF</td>
      <td>RB</td>
      <td>11</td>
      <td>Matt Breida</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>112</th>
      <td>Cin</td>
      <td>RB</td>
      <td>6</td>
      <td>Giovani Bernard</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>113</th>
      <td>NO</td>
      <td>K</td>
      <td>1</td>
      <td>Wil Lutz</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>114</th>
      <td>LAR</td>
      <td>RB</td>
      <td>0</td>
      <td>Malcolm Brown</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>115</th>
      <td>Det</td>
      <td>WR</td>
      <td>5</td>
      <td>Kenny Golladay</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>116</th>
      <td>GB</td>
      <td>K</td>
      <td>0</td>
      <td>Mason Crosby</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>117</th>
      <td>LAC</td>
      <td>WR</td>
      <td>1</td>
      <td>Mike Williams</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>118</th>
      <td>Phi</td>
      <td>K</td>
      <td>2</td>
      <td>Jake Elliott</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>119</th>
      <td>Cle</td>
      <td>RB</td>
      <td>3</td>
      <td>Nick Chubb</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>120</th>
      <td>NE</td>
      <td>RB</td>
      <td>2</td>
      <td>James White</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>121</th>
      <td>GB</td>
      <td>RB</td>
      <td>2</td>
      <td>Aaron Jones</td>
      <td>2 Gurley's 1 Cup</td>
      <td></td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>122</th>
      <td>Mia</td>
      <td>QB</td>
      <td>6</td>
      <td>Ryan Tannehill</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>123</th>
      <td>Bal</td>
      <td>RB</td>
      <td>0</td>
      <td>Javorius Allen</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>124</th>
      <td>NE</td>
      <td>DEF</td>
      <td>1</td>
      <td>New England</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>125</th>
      <td>Ind</td>
      <td>RB</td>
      <td>1</td>
      <td>Jordan Wilkins</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>126</th>
      <td>Atl</td>
      <td>DEF</td>
      <td>1</td>
      <td>Atlanta</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>127</th>
      <td>Pit</td>
      <td>K</td>
      <td>1</td>
      <td>Chris Boswell</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>128</th>
      <td>Atl</td>
      <td>WR</td>
      <td>1</td>
      <td>Calvin Ridley</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>129</th>
      <td>GB</td>
      <td>WR</td>
      <td>1</td>
      <td>Randall Cobb</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>130</th>
      <td>Cin</td>
      <td>K</td>
      <td>0</td>
      <td>Randy Bullock</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>131</th>
      <td>Cin</td>
      <td>QB</td>
      <td>1</td>
      <td>Andy Dalton</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>132</th>
      <td>Car</td>
      <td>RB</td>
      <td>1</td>
      <td>C.J. Anderson</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>133</th>
      <td>Ind</td>
      <td>RB</td>
      <td>1</td>
      <td>Marlon Mack</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>134</th>
      <td>KC</td>
      <td>K</td>
      <td>1</td>
      <td>Harrison Butker</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>135</th>
      <td>Mia</td>
      <td>RB</td>
      <td>0</td>
      <td>Frank Gore</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>136</th>
      <td>Min</td>
      <td>K</td>
      <td>3</td>
      <td>Dan Bailey</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>137</th>
      <td>TB</td>
      <td>QB</td>
      <td>6</td>
      <td>Ryan Fitzpatrick</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>138</th>
      <td>Cin</td>
      <td>WR</td>
      <td>0</td>
      <td>Tyler Boyd</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>139</th>
      <td>Ind</td>
      <td>TE</td>
      <td>1</td>
      <td>Eric Ebron</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>140</th>
      <td>Jax</td>
      <td>QB</td>
      <td>1</td>
      <td>Blake Bortles</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>141</th>
      <td>Cle</td>
      <td>DEF</td>
      <td>0</td>
      <td>Cleveland</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>142</th>
      <td>Phi</td>
      <td>TE</td>
      <td>4</td>
      <td>Dallas Goedert</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>143</th>
      <td>Ten</td>
      <td>DEF</td>
      <td>0</td>
      <td>Tennessee</td>
      <td>Bilbro Naggins</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>144</th>
      <td>Bal</td>
      <td>WR</td>
      <td>1</td>
      <td>John Brown</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>145</th>
      <td>NO</td>
      <td>TE</td>
      <td>1</td>
      <td>Benjamin Watson</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>146</th>
      <td>Ari</td>
      <td>DEF</td>
      <td>1</td>
      <td>Arizona</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>147</th>
      <td>Min</td>
      <td>RB</td>
      <td>1</td>
      <td>Latavius Murray</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>148</th>
      <td>LAC</td>
      <td>WR</td>
      <td>1</td>
      <td>Tyrell Williams</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>149</th>
      <td>SF</td>
      <td>K</td>
      <td>1</td>
      <td>Robbie Gould</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Sea</td>
      <td>WR</td>
      <td>24</td>
      <td>Doug Baldwin</td>
      <td>FirstRoundFlops</td>
      <td>AB</td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>151</th>
      <td>Oak</td>
      <td>WR</td>
      <td>43</td>
      <td>Amari Cooper</td>
      <td>FirstRoundFlops</td>
      <td>AB</td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>152</th>
      <td>Chi</td>
      <td>WR</td>
      <td>5</td>
      <td>Allen Robinson II</td>
      <td>Jake News</td>
      <td>B</td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>153</th>
      <td>NO</td>
      <td>RB</td>
      <td>9</td>
      <td>Alvin Kamara</td>
      <td>Jake News</td>
      <td>B</td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>154</th>
      <td>TB</td>
      <td>WR</td>
      <td>30</td>
      <td>Mike Evans</td>
      <td>Jake News</td>
      <td>AB</td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>155</th>
      <td>Ind</td>
      <td>QB</td>
      <td>5</td>
      <td>Andrew Luck</td>
      <td>Jake News</td>
      <td>B</td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>156</th>
      <td>Min</td>
      <td>WR</td>
      <td>28</td>
      <td>Adam Thielen</td>
      <td>2 Gurley's 1 Cup</td>
      <td>B</td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Min</td>
      <td>RB</td>
      <td>64</td>
      <td>Dalvin Cook</td>
      <td>2 Gurley's 1 Cup</td>
      <td>B</td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>158</th>
      <td>LAR</td>
      <td>RB</td>
      <td>40</td>
      <td>Todd Gurley II</td>
      <td>2 Gurley's 1 Cup</td>
      <td>AB</td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>159</th>
      <td>Sea</td>
      <td>WR</td>
      <td>5</td>
      <td>Tyler Lockett</td>
      <td>Cry me a Philip</td>
      <td>B</td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>160</th>
      <td>KC</td>
      <td>WR</td>
      <td>32</td>
      <td>Sammy Watkins</td>
      <td>Cry me a Philip</td>
      <td>B</td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>161</th>
      <td>Min</td>
      <td>TE</td>
      <td>17</td>
      <td>Kyle Rudolph</td>
      <td>Cry me a Philip</td>
      <td>AB</td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>162</th>
      <td>Sea</td>
      <td>QB</td>
      <td>34</td>
      <td>Russell Wilson</td>
      <td>Cry me a Philip</td>
      <td>B</td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>163</th>
      <td>Min</td>
      <td>WR</td>
      <td>24</td>
      <td>Stefon Diggs</td>
      <td>Bilbro Naggins</td>
      <td>AB</td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>164</th>
      <td>NYG</td>
      <td>WR</td>
      <td>29</td>
      <td>Odell Beckham Jr.</td>
      <td>Bilbro Naggins</td>
      <td>AB</td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>165</th>
      <td>NE</td>
      <td>WR</td>
      <td>6</td>
      <td>Josh Gordon</td>
      <td>Bilbro Naggins</td>
      <td>B</td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Pit</td>
      <td>RB</td>
      <td>66</td>
      <td>Le'Veon Bell</td>
      <td>Bilbro Naggins</td>
      <td>AB</td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Mia</td>
      <td>RB</td>
      <td>17</td>
      <td>Kenyan Drake</td>
      <td>Wu wears the pants!</td>
      <td>B</td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>168</th>
      <td>GB</td>
      <td>WR</td>
      <td>27</td>
      <td>Davante Adams</td>
      <td>Wu wears the pants!</td>
      <td>AB</td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>169</th>
      <td>LAC</td>
      <td>RB</td>
      <td>39</td>
      <td>Melvin Gordon III</td>
      <td>Wu wears the pants!</td>
      <td>AB</td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>170</th>
      <td>Hou</td>
      <td>WR</td>
      <td>56</td>
      <td>DeAndre Hopkins</td>
      <td>Wu wears the pants!</td>
      <td>B</td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>171</th>
      <td>Jax</td>
      <td>WR</td>
      <td>5</td>
      <td>Keelan Cole</td>
      <td>Bye Week</td>
      <td>B</td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>172</th>
      <td>Atl</td>
      <td>RB</td>
      <td>22</td>
      <td>Devonta Freeman</td>
      <td>Bye Week</td>
      <td>AB</td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>173</th>
      <td>Phi</td>
      <td>RB</td>
      <td>5</td>
      <td>Corey Clement</td>
      <td>Bye Week</td>
      <td>B</td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>174</th>
      <td>Det</td>
      <td>WR</td>
      <td>29</td>
      <td>Golden Tate</td>
      <td>Bye Week</td>
      <td>AB</td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>175</th>
      <td>Hou</td>
      <td>QB</td>
      <td>6</td>
      <td>Deshaun Watson</td>
      <td>Nags</td>
      <td>AB</td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Ari</td>
      <td>RB</td>
      <td>18</td>
      <td>David Johnson</td>
      <td>Nags</td>
      <td>AB</td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>177</th>
      <td>NO</td>
      <td>WR</td>
      <td>18</td>
      <td>Michael Thomas</td>
      <td>Hunter Punter</td>
      <td>AB</td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>178</th>
      <td>Min</td>
      <td>QB</td>
      <td>22</td>
      <td>Kirk Cousins</td>
      <td>Hunter Punter</td>
      <td>AB</td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>179</th>
      <td>SF</td>
      <td>WR</td>
      <td>5</td>
      <td>Marquise Goodwin</td>
      <td>Hunter Punter</td>
      <td>B</td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>180</th>
      <td>Dal</td>
      <td>RB</td>
      <td>55</td>
      <td>Ezekiel Elliott</td>
      <td>Hunter Punter</td>
      <td>B</td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>181</th>
      <td>Was</td>
      <td>RB</td>
      <td>5</td>
      <td>Chris Thompson</td>
      <td>Chi ShingT's Team</td>
      <td>B</td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>182</th>
      <td>TB</td>
      <td>RB</td>
      <td>10</td>
      <td>Peyton Barber</td>
      <td>Chi ShingT's Team</td>
      <td>B</td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>183</th>
      <td>Phi</td>
      <td>WR</td>
      <td>2</td>
      <td>Jordan Matthews</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>184</th>
      <td>Hou</td>
      <td>WR</td>
      <td>5</td>
      <td>Will Fuller V</td>
      <td>Chi ShingT's Team</td>
      <td>B</td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>185</th>
      <td>Chi</td>
      <td>RB</td>
      <td>8</td>
      <td>Tarik Cohen</td>
      <td>Thx Ron and Le'Veon</td>
      <td>B</td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>186</th>
      <td>NYG</td>
      <td>TE</td>
      <td>8</td>
      <td>Evan Engram</td>
      <td>Thx Ron and Le'Veon</td>
      <td>B</td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>187</th>
      <td>Ten</td>
      <td>RB</td>
      <td>19</td>
      <td>Derrick Henry</td>
      <td>Thx Ron and Le'Veon</td>
      <td>B</td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>188</th>
      <td>Pit</td>
      <td>WR</td>
      <td>5</td>
      <td>JuJu Smith-Schuster</td>
      <td>Thx Ron and Le'Veon</td>
      <td>B</td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>189</th>
      <td>Oak</td>
      <td>WR</td>
      <td>7</td>
      <td>Jordy Nelson</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>190</th>
      <td>Phi</td>
      <td>QB</td>
      <td>10</td>
      <td>Carson Wentz</td>
      <td>G</td>
      <td>B</td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>191</th>
      <td>Det</td>
      <td>WR</td>
      <td>12</td>
      <td>Marvin Jones Jr.</td>
      <td>G</td>
      <td>B</td>
      <td>Jiwei</td>
    </tr>
  </tbody>
</table>
</div>




```python
# reset the team names so that manager and most recent team name are matching again.
# this is done primarily for the traded player

for i, r in roster_df.iterrows():
    for index, t in manager_df.iterrows():
        if r['manager'] == t['manager']:
            roster_df.at[i, "team"] = t["team_name"]
            
roster_df.head(15)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NFLTeam</th>
      <th>Pos</th>
      <th>cost</th>
      <th>name</th>
      <th>team</th>
      <th>keeper</th>
      <th>manager</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dal</td>
      <td>QB</td>
      <td>6</td>
      <td>Dak Prescott</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Pit</td>
      <td>WR</td>
      <td>81</td>
      <td>Antonio Brown</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NE</td>
      <td>TE</td>
      <td>64</td>
      <td>Rob Gronkowski</td>
      <td>Wu wears the pants!</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ind</td>
      <td>WR</td>
      <td>51</td>
      <td>T.Y. Hilton</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>5</th>
      <td>NYG</td>
      <td>RB</td>
      <td>74</td>
      <td>Saquon Barkley</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Buf</td>
      <td>RB</td>
      <td>29</td>
      <td>LeSean McCoy</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Hou</td>
      <td>RB</td>
      <td>28</td>
      <td>Lamar Miller</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Cin</td>
      <td>WR</td>
      <td>67</td>
      <td>A.J. Green</td>
      <td>Hunter Punter</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Atl</td>
      <td>WR</td>
      <td>85</td>
      <td>Julio Jones</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Ari</td>
      <td>WR</td>
      <td>41</td>
      <td>Larry Fitzgerald</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Phi</td>
      <td>RB</td>
      <td>34</td>
      <td>Jay Ajayi</td>
      <td>Bye Week</td>
      <td></td>
      <td>Doug</td>
    </tr>
    <tr>
      <th>13</th>
      <td>KC</td>
      <td>WR</td>
      <td>60</td>
      <td>Tyreek Hill</td>
      <td>Nags</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>14</th>
      <td>LAC</td>
      <td>WR</td>
      <td>71</td>
      <td>Keenan Allen</td>
      <td>Jake News</td>
      <td></td>
      <td>Sean</td>
    </tr>
  </tbody>
</table>
</div>




```python
# grab the unique team names

ff_team_names = list(manager_df.team_name)
print(ff_team_names)
```

    ["2 Gurley's 1 Cup", 'Bilbro Naggins', 'Bye Week', "Chi ShingT's Team", 'Cry me a Philip', 'FirstRoundFlops', 'G', 'Hunter Punter', 'Jake News', 'Nags', "Thx Ron and Le'Veon", 'Wu wears the pants!']
    


```python
# create a list of all rosters sorted by cost of player for each team

roster_df = roster_df.sort_values(by=['cost'], ascending=False)
roster_df = roster_df[['name', 'NFLTeam', 'Pos', 'cost', 'keeper', 'team', 'manager']]
for ff in ff_team_names:
    print(roster_df.loc[roster_df['team'] == ff])
    print("------------------------------------------------------------------------------------")
```

                     name NFLTeam  Pos  cost keeper              team manager
    157       Dalvin Cook     Min   RB    64      B  2 Gurley's 1 Cup    Matt
    15     Alshon Jeffery     Phi   WR    40         2 Gurley's 1 Cup    Matt
    158    Todd Gurley II     LAR   RB    40     AB  2 Gurley's 1 Cup    Matt
    33      Jarvis Landry     Cle   WR    38         2 Gurley's 1 Cup    Matt
    156      Adam Thielen     Min   WR    28      B  2 Gurley's 1 Cup    Matt
    57   Emmanuel Sanders     Den   WR    18         2 Gurley's 1 Cup    Matt
    63    Phillip Lindsay     Den   RB    15         2 Gurley's 1 Cup    Matt
    68        Jordan Reed     Was   TE    10         2 Gurley's 1 Cup    Matt
    77       Philadelphia     Phi  DEF     5         2 Gurley's 1 Cup    Matt
    179  Marquise Goodwin      SF   WR     5      B  2 Gurley's 1 Cup    Matt
    92         Josh Rosen     Ari   QB     5         2 Gurley's 1 Cup    Matt
    71         Jared Cook     Oak   TE     3         2 Gurley's 1 Cup    Matt
    64         Jared Goff     LAR   QB     2         2 Gurley's 1 Cup    Matt
    121       Aaron Jones      GB   RB     2         2 Gurley's 1 Cup    Matt
    104       Sam Darnold     NYJ   QB     1         2 Gurley's 1 Cup    Matt
    116      Mason Crosby      GB    K     0         2 Gurley's 1 Cup    Matt
    ------------------------------------------------------------------------------------
                        name NFLTeam  Pos  cost keeper            team manager
    27   Christian McCaffrey     Car   RB    67         Bilbro Naggins     Ron
    166         Le'Veon Bell     Pit   RB    66     AB  Bilbro Naggins     Ron
    41          Alex Collins     Bal   RB    56         Bilbro Naggins     Ron
    164    Odell Beckham Jr.     NYG   WR    29     AB  Bilbro Naggins     Ron
    163         Stefon Diggs     Min   WR    24     AB  Bilbro Naggins     Ron
    31          Jimmy Graham      GB   TE    23         Bilbro Naggins     Ron
    111          Matt Breida      SF   RB    11         Bilbro Naggins     Ron
    95           Sony Michel      NE   RB     7         Bilbro Naggins     Ron
    165          Josh Gordon      NE   WR     6      B  Bilbro Naggins     Ron
    112      Giovani Bernard     Cin   RB     6         Bilbro Naggins     Ron
    106          Matt Bryant     Atl    K     2         Bilbro Naggins     Ron
    131          Andy Dalton     Cin   QB     1         Bilbro Naggins     Ron
    76    Ben Roethlisberger     Pit   QB     0         Bilbro Naggins     Ron
    138           Tyler Boyd     Cin   WR     0         Bilbro Naggins     Ron
    101        Quincy Enunwa     NYJ   WR     0         Bilbro Naggins     Ron
    143            Tennessee     Ten  DEF     0         Bilbro Naggins     Ron
    ------------------------------------------------------------------------------------
                       name NFLTeam  Pos  cost keeper      team manager
    5        Saquon Barkley     NYG   RB    74         Bye Week    Doug
    37          Corey Davis     Ten   WR    35         Bye Week    Doug
    12            Jay Ajayi     Phi   RB    34         Bye Week    Doug
    52           Cam Newton     Car   QB    30         Bye Week    Doug
    174         Golden Tate     Det   WR    29     AB  Bye Week    Doug
    172     Devonta Freeman     Atl   RB    22     AB  Bye Week    Doug
    74       Devin Funchess     Car   WR    15         Bye Week    Doug
    122      Ryan Tannehill     Mia   QB     6         Bye Week    Doug
    171         Keelan Cole     Jax   WR     5      B  Bye Week    Doug
    173       Corey Clement     Phi   RB     5      B  Bye Week    Doug
    60   Stephen Gostkowski      NE    K     5         Bye Week    Doug
    70          Los Angeles     LAC  DEF     4         Bye Week    Doug
    99      Jerick McKinnon      SF   RB     3         Bye Week    Doug
    59        Albert Wilson     Mia   WR     0         Bye Week    Doug
    66         Ted Ginn Jr.      NO   WR     0         Bye Week    Doug
    73        Rhett Ellison     NYG   TE     0         Bye Week    Doug
    ------------------------------------------------------------------------------------
                    name NFLTeam  Pos  cost keeper               team    manager
    1      Antonio Brown     Pit   WR    81         Chi ShingT's Team  Chi Shing
    25         Joe Mixon     Cin   RB    51         Chi ShingT's Team  Chi Shing
    38       Mark Ingram      NO   RB    39         Chi ShingT's Team  Chi Shing
    16      Robert Woods     LAR   WR    23         Chi ShingT's Team  Chi Shing
    30     Tevin Coleman     Atl   RB    22         Chi ShingT's Team  Chi Shing
    182    Peyton Barber      TB   RB    10      B  Chi ShingT's Team  Chi Shing
    181   Chris Thompson     Was   RB     5      B  Chi ShingT's Team  Chi Shing
    184    Will Fuller V     Hou   WR     5      B  Chi ShingT's Team  Chi Shing
    102  Patrick Mahomes      KC   QB     3         Chi ShingT's Team  Chi Shing
    183  Jordan Matthews     Phi   WR     2         Chi ShingT's Team  Chi Shing
    127    Chris Boswell     Pit    K     1         Chi ShingT's Team  Chi Shing
    19   Phillip Dorsett      NE   WR     0         Chi ShingT's Team  Chi Shing
    47       Will Dissly     Sea   TE     0         Chi ShingT's Team  Chi Shing
    135       Frank Gore     Mia   RB     0         Chi ShingT's Team  Chi Shing
    82      Theo Riddick     Det   RB     0         Chi ShingT's Team  Chi Shing
    141        Cleveland     Cle  DEF     0         Chi ShingT's Team  Chi Shing
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper             team manager
    10        Julio Jones     Atl   WR    85         Cry me a Philip    Evan
    4         T.Y. Hilton     Ind   WR    51         Cry me a Philip    Evan
    162    Russell Wilson     Sea   QB    34      B  Cry me a Philip    Evan
    160     Sammy Watkins      KC   WR    32      B  Cry me a Philip    Evan
    8        Lamar Miller     Hou   RB    28         Cry me a Philip    Evan
    161      Kyle Rudolph     Min   TE    17     AB  Cry me a Philip    Evan
    69     Robby Anderson     NYJ   WR    14         Cry me a Philip    Evan
    72     Isaiah Crowell     NYJ   RB    14         Cry me a Philip    Evan
    56   Matthew Stafford     Det   QB    14         Cry me a Philip    Evan
    159     Tyler Lockett     Sea   WR     5      B  Cry me a Philip    Evan
    113          Wil Lutz      NO    K     1         Cry me a Philip    Evan
    132     C.J. Anderson     Car   RB     1         Cry me a Philip    Evan
    124       New England      NE  DEF     1         Cry me a Philip    Evan
    139        Eric Ebron     Ind   TE     1         Cry me a Philip    Evan
    144        John Brown     Bal   WR     1         Cry me a Philip    Evan
    146           Arizona     Ari  DEF     1         Cry me a Philip    Evan
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper             team manager
    17        Kareem Hunt      KC   RB    79         FirstRoundFlops   Rajiv
    29      Jordan Howard     Chi   RB    64         FirstRoundFlops   Rajiv
    151      Amari Cooper     Oak   WR    43     AB  FirstRoundFlops   Rajiv
    45        Trey Burton     Chi   TE    31         FirstRoundFlops   Rajiv
    21          Tom Brady      NE   QB    28         FirstRoundFlops   Rajiv
    150      Doug Baldwin     Sea   WR    24     AB  FirstRoundFlops   Rajiv
    36   Michael Crabtree     Bal   WR    19         FirstRoundFlops   Rajiv
    96         Jack Doyle     Ind   TE     4         FirstRoundFlops   Rajiv
    126           Atlanta     Atl  DEF     1         FirstRoundFlops   Rajiv
    117     Mike Williams     LAC   WR     1         FirstRoundFlops   Rajiv
    145   Benjamin Watson      NO   TE     1         FirstRoundFlops   Rajiv
    134   Harrison Butker      KC    K     1         FirstRoundFlops   Rajiv
    140     Blake Bortles     Jax   QB     1         FirstRoundFlops   Rajiv
    148   Tyrell Williams     LAC   WR     1         FirstRoundFlops   Rajiv
    149      Robbie Gould      SF    K     1         FirstRoundFlops   Rajiv
    147   Latavius Murray     Min   RB     1         FirstRoundFlops   Rajiv
    ------------------------------------------------------------------------------------
                       name NFLTeam  Pos  cost keeper team manager
    44     Demaryius Thomas     Den   WR    38           G   Jiwei
    65         Chris Carson     Sea   RB    25           G   Jiwei
    40          Cooper Kupp     LAR   WR    24           G   Jiwei
    51          Carlos Hyde     Cle   RB    22           G   Jiwei
    55          O.J. Howard      TB   TE    14           G   Jiwei
    191    Marvin Jones Jr.     Det   WR    12      B    G   Jiwei
    190        Carson Wentz     Phi   QB    10      B    G   Jiwei
    61          David Njoku     Cle   TE     9           G   Jiwei
    189        Jordy Nelson     Oak   WR     7           G   Jiwei
    85          Chris Ivory     Buf   RB     5           G   Jiwei
    97            Baltimore     Bal  DEF     5           G   Jiwei
    105        Kenny Stills     Mia   WR     3           G   Jiwei
    109           Matt Ryan     Atl   QB     3           G   Jiwei
    48       Vance McDonald     Pit   TE     0           G   Jiwei
    54       Dede Westbrook     Jax   WR     0           G   Jiwei
    89   Chandler Catanzaro      TB    K     0           G   Jiwei
    ------------------------------------------------------------------------------------
                      name NFLTeam  Pos  cost keeper           team manager
    9           A.J. Green     Cin   WR    67         Hunter Punter    Jake
    180    Ezekiel Elliott     Dal   RB    55      B  Hunter Punter    Jake
    28           Zach Ertz     Phi   TE    49         Hunter Punter    Jake
    23       Brandin Cooks     LAR   WR    39         Hunter Punter    Jake
    178       Kirk Cousins     Min   QB    22     AB  Hunter Punter    Jake
    177     Michael Thomas      NO   WR    18     AB  Hunter Punter    Jake
    80       Rashaad Penny     Sea   RB    13         Hunter Punter    Jake
    35     Adrian Peterson     Was   RB    12         Hunter Punter    Jake
    58         Los Angeles     LAR  DEF    11         Hunter Punter    Jake
    84        Bilal Powell     NYJ   RB     7         Hunter Punter    Jake
    0         Dak Prescott     Dal   QB     6         Hunter Punter    Jake
    94       Justin Tucker     Bal    K     4         Hunter Punter    Jake
    120        James White      NE   RB     2         Hunter Punter    Jake
    24      DeSean Jackson      TB   WR     2         Hunter Punter    Jake
    108  Ricky Seals-Jones     Ari   TE     1         Hunter Punter    Jake
    129       Randall Cobb      GB   WR     1         Hunter Punter    Jake
    ------------------------------------------------------------------------------------
                      name NFLTeam  Pos  cost keeper       team manager
    14        Keenan Allen     LAC   WR    71         Jake News    Sean
    3    Leonard Fournette     Jax   RB    60         Jake News    Sean
    39         Chris Hogan      NE   WR    39         Jake News    Sean
    20       Royce Freeman     Den   RB    36         Jake News    Sean
    154         Mike Evans      TB   WR    30     AB  Jake News    Sean
    62         T.J. Yeldon     Jax   RB    11         Jake News    Sean
    153       Alvin Kamara      NO   RB     9      B  Jake News    Sean
    137   Ryan Fitzpatrick      TB   QB     6         Jake News    Sean
    86      Baker Mayfield     Cle   QB     6         Jake News    Sean
    75       George Kittle      SF   TE     6         Jake News    Sean
    155        Andrew Luck     Ind   QB     5      B  Jake News    Sean
    152  Allen Robinson II     Chi   WR     5      B  Jake News    Sean
    142     Dallas Goedert     Phi   TE     4         Jake News    Sean
    110       Chris Godwin      TB   WR     3         Jake News    Sean
    98             Seattle     Sea  DEF     2         Jake News    Sean
    130      Randy Bullock     Cin    K     0         Jake News    Sean
    ------------------------------------------------------------------------------------
                     name NFLTeam    Pos  cost keeper  team manager
    13        Tyreek Hill      KC     WR    60         Nags    Ryan
    26       Travis Kelce      KC     TE    52         Nags    Ryan
    32      Aaron Rodgers      GB     QB    51         Nags    Ryan
    50         Dion Lewis     Ten     RB    29         Nags    Ryan
    6        LeSean McCoy     Buf     RB    29         Nags    Ryan
    176     David Johnson     Ari     RB    18     AB  Nags    Ryan
    53     Marshawn Lynch     Oak     RB    16         Nags    Ryan
    18    Kelvin Benjamin     Buf     WR    15         Nags    Ryan
    34       Jacksonville     Jax    DEF     8         Nags    Ryan
    175    Deshaun Watson     Hou     QB     6     AB  Nags    Ryan
    83     DeVante Parker     Mia     WR     5         Nags    Ryan
    93   Sterling Shepard     NYG     WR     4         Nags    Ryan
    136        Dan Bailey     Min      K     3         Nags    Ryan
    119        Nick Chubb     Cle     RB     3         Nags    Ryan
    128     Calvin Ridley     Atl     WR     1         Nags    Ryan
    107      Tavon Austin     Dal  WR,RB     0         Nags    Ryan
    ------------------------------------------------------------------------------------
                        name NFLTeam  Pos  cost keeper                 team  \
    11      Larry Fitzgerald     Ari   WR    41         Thx Ron and Le'Veon   
    187        Derrick Henry     Ten   RB    19      B  Thx Ron and Le'Veon   
    43        Julian Edelman      NE   WR    19         Thx Ron and Le'Veon   
    7          Philip Rivers     LAC   QB    11         Thx Ron and Le'Veon   
    88       Kerryon Johnson     Det   RB    10         Thx Ron and Le'Veon   
    185          Tarik Cohen     Chi   RB     8      B  Thx Ron and Le'Veon   
    186          Evan Engram     NYG   TE     8      B  Thx Ron and Le'Veon   
    115       Kenny Golladay     Det   WR     5         Thx Ron and Le'Veon   
    100         James Conner     Pit   RB     5         Thx Ron and Le'Veon   
    188  JuJu Smith-Schuster     Pit   WR     5      B  Thx Ron and Le'Veon   
    91               Chicago     Chi  DEF     3         Thx Ron and Le'Veon   
    79         Austin Hooper     Atl   TE     3         Thx Ron and Le'Veon   
    118         Jake Elliott     Phi    K     2         Thx Ron and Le'Veon   
    103     Antonio Callaway     Cle   WR     0         Thx Ron and Le'Veon   
    67          Nyheim Hines     Ind   RB     0         Thx Ron and Le'Veon   
    78      Geronimo Allison      GB   WR     0         Thx Ron and Le'Veon   
    
        manager  
    11   Andrew  
    187  Andrew  
    43   Andrew  
    7    Andrew  
    88   Andrew  
    185  Andrew  
    186  Andrew  
    115  Andrew  
    100  Andrew  
    188  Andrew  
    91   Andrew  
    79   Andrew  
    118  Andrew  
    103  Andrew  
    67   Andrew  
    78   Andrew  
    ------------------------------------------------------------------------------------
                      name NFLTeam  Pos  cost keeper                 team manager
    2       Rob Gronkowski      NE   TE    64         Wu wears the pants!     Dai
    170    DeAndre Hopkins     Hou   WR    56      B  Wu wears the pants!     Dai
    169  Melvin Gordon III     LAC   RB    39     AB  Wu wears the pants!     Dai
    168      Davante Adams      GB   WR    27     AB  Wu wears the pants!     Dai
    49          Drew Brees      NO   QB    25         Wu wears the pants!     Dai
    167       Kenyan Drake     Mia   RB    17      B  Wu wears the pants!     Dai
    81      Nelson Agholor     Phi   WR    13         Wu wears the pants!     Dai
    90     Jamaal Williams      GB   RB     9         Wu wears the pants!     Dai
    22       Alfred Morris      SF   RB     8         Wu wears the pants!     Dai
    125     Jordan Wilkins     Ind   RB     1         Wu wears the pants!     Dai
    133        Marlon Mack     Ind   RB     1         Wu wears the pants!     Dai
    123     Javorius Allen     Bal   RB     0         Wu wears the pants!     Dai
    42     Ronald Jones II      TB   RB     0         Wu wears the pants!     Dai
    46           Green Bay      GB  DEF     0         Wu wears the pants!     Dai
    87      Christian Kirk     Ari   WR     0         Wu wears the pants!     Dai
    114      Malcolm Brown     LAR   RB     0         Wu wears the pants!     Dai
    ------------------------------------------------------------------------------------
    


```python
# Save data as CSV
roster_df.to_csv("current_roster.csv")
```
