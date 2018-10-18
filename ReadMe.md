

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
print (team_names)
manager_list = []
for x in np.arange(len(team_names)):
    manager = input("Who is the manager for " + team_names[x] + "? ")
    manager_list.append({"team_name": team_names[x], "manager": manager})
manager_df = pd.DataFrame(manager_list)
manager_df.head(12)
```

    ["2 Gurley's 1 Cup", 'Bye Week', "Chi ShingT's Team", 'Cry me a Philip', 'Daized and Confused', 'Douglast', 'FirstRoundFlops', 'Free-Mangina', 'G', 'No bunny no fu fu', 'Seandon', 'Yeebola']
    Who is the manager for 2 Gurley's 1 Cup? Matt
    Who is the manager for Bye Week? Doug
    Who is the manager for Chi ShingT's Team? Chi Shing
    Who is the manager for Cry me a Philip? Evan
    Who is the manager for Daized and Confused? Sean
    Who is the manager for Douglast? Jake
    Who is the manager for FirstRoundFlops? Rajiv
    Who is the manager for Free-Mangina? Ryan
    Who is the manager for G? Jiwei
    Who is the manager for No bunny no fu fu? Andrew
    Who is the manager for Seandon? Dai
    Who is the manager for Yeebola? Ron
    




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
      <td>Doug</td>
      <td>Bye Week</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Chi Shing</td>
      <td>Chi ShingT's Team</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Evan</td>
      <td>Cry me a Philip</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sean</td>
      <td>Daized and Confused</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Jake</td>
      <td>Douglast</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Rajiv</td>
      <td>FirstRoundFlops</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Ryan</td>
      <td>Free-Mangina</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Jiwei</td>
      <td>G</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Andrew</td>
      <td>No bunny no fu fu</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Dai</td>
      <td>Seandon</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Ron</td>
      <td>Yeebola</td>
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
      <td>Douglast</td>
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
      <td>Seandon</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Daized and Confused</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>No bunny no fu fu</td>
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
      <td>Douglast</td>
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
      <td>Douglast</td>
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
      <td>Seandon</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Daized and Confused</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>No bunny no fu fu</td>
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
      <td>Douglast</td>
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
      <td>Douglast</td>
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
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Daized and Confused</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>No bunny no fu fu</td>
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
      <td>Douglast</td>
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
      <td>No bunny no fu fu</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>14</th>
      <td>LAC</td>
      <td>WR</td>
      <td>71</td>
      <td>Keenan Allen</td>
      <td>Daized and Confused</td>
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
      <td>DeSean Jackson</td>
      <td>DeMarco Murray</td>
      <td>Jake</td>
      <td>TB</td>
      <td>WR</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Denver</td>
      <td>New Orleans</td>
      <td>Sean</td>
      <td>Den</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Dante Pettis</td>
      <td>D.J. Moore</td>
      <td>Sean</td>
      <td>SF</td>
      <td>WR</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Tyrod Taylor</td>
      <td>Mike Wallace</td>
      <td>Jiwei</td>
      <td>Cle</td>
      <td>QB</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Trent Taylor</td>
      <td>Josh Doctson</td>
      <td>Ron</td>
      <td>SF</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Adam Vinatieri</td>
      <td>Graham Gano</td>
      <td>Sean</td>
      <td>Ind</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Brandon Marshall</td>
      <td>Brandon McManus</td>
      <td>Dai</td>
      <td>Sea</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Seattle</td>
      <td>Green Bay</td>
      <td>Ron</td>
      <td>Sea</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Ryan Grant</td>
      <td>Trent Taylor</td>
      <td>Ron</td>
      <td>Ind</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Phillip Dorsett</td>
      <td>Greg Olsen</td>
      <td>Chi Shing</td>
      <td>NE</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Javorius Allen</td>
      <td>Ben Roethlisberger</td>
      <td>Dai</td>
      <td>Bal</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Matt Breida</td>
      <td>Allen Hurns</td>
      <td>Ron</td>
      <td>SF</td>
      <td>RB</td>
      <td>11</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Houston</td>
      <td>Seattle</td>
      <td>Ron</td>
      <td>Hou</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Ben Roethlisberger</td>
      <td>Jared Goff</td>
      <td>Ron</td>
      <td>Pit</td>
      <td>QB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Jalen Richard</td>
      <td>John Ross</td>
      <td>Dai</td>
      <td>Oak</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Jonnu Smith</td>
      <td>Austin Seferian-Jenkins</td>
      <td>Ron</td>
      <td>Ten</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Antonio Callaway</td>
      <td>Cameron Meredith</td>
      <td>Andrew</td>
      <td>Cle</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Will Dissly</td>
      <td>Free Agent</td>
      <td>Chi Shing</td>
      <td>Sea</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Cody Parkey</td>
      <td>Jalen Richard</td>
      <td>Dai</td>
      <td>Chi</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Ryan Fitzpatrick</td>
      <td>Ted Ginn Jr.</td>
      <td>Sean</td>
      <td>TB</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Giovani Bernard</td>
      <td>Ryan Grant</td>
      <td>Ron</td>
      <td>Cin</td>
      <td>RB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Dan Bailey</td>
      <td>Daniel Carlson</td>
      <td>Ryan</td>
      <td>Min</td>
      <td>K</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Jesse James</td>
      <td>Dante Pettis</td>
      <td>Sean</td>
      <td>Pit</td>
      <td>TE</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Jared Goff</td>
      <td>Marcus Mariota</td>
      <td>Matt</td>
      <td>LAR</td>
      <td>QB</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Dallas</td>
      <td>Denver</td>
      <td>Sean</td>
      <td>Dal</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Josh Lambo</td>
      <td>Adam Vinatieri</td>
      <td>Sean</td>
      <td>Jax</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Tyler Boyd</td>
      <td>Jonnu Smith</td>
      <td>Ron</td>
      <td>Cin</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Mark Walton</td>
      <td>Cody Parkey</td>
      <td>Dai</td>
      <td>Cin</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Malcolm Brown</td>
      <td>Brandon Marshall</td>
      <td>Dai</td>
      <td>LAR</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Frank Gore</td>
      <td>Antonio Gates</td>
      <td>Chi Shing</td>
      <td>Mia</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Cleveland</td>
      <td>Carolina</td>
      <td>Chi Shing</td>
      <td>Cle</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Greg Olsen</td>
      <td>Ricky Seals-Jones</td>
      <td>Jake</td>
      <td>Car</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Corey Grant</td>
      <td>Free Agent</td>
      <td>Dai</td>
      <td>Jax</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Ted Ginn Jr.</td>
      <td>Michael Gallup</td>
      <td>Doug</td>
      <td>NO</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Ito Smith</td>
      <td>Alex Smith</td>
      <td>Ron</td>
      <td>Atl</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Nyheim Hines</td>
      <td>Ronald Jones II</td>
      <td>Andrew</td>
      <td>Ind</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Theo Riddick</td>
      <td>Darren Sproles</td>
      <td>Chi Shing</td>
      <td>Det</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Chandler Catanzaro</td>
      <td>Greg Zuerlein</td>
      <td>Jiwei</td>
      <td>TB</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Matt Prater</td>
      <td>Corey Grant</td>
      <td>Dai</td>
      <td>Det</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Christian Kirk</td>
      <td>Mark Walton</td>
      <td>Dai</td>
      <td>Ari</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Ronald Jones II</td>
      <td>Jamison Crowder</td>
      <td>Dai</td>
      <td>TB</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Jordy Nelson</td>
      <td>Tyrod Taylor</td>
      <td>Jiwei</td>
      <td>Oak</td>
      <td>WR</td>
      <td>7</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Ryan Tannehill</td>
      <td>Doug Martin</td>
      <td>Doug</td>
      <td>Mia</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Baker Mayfield</td>
      <td>Austin Ekeler</td>
      <td>Sean</td>
      <td>Cle</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Chris Ivory</td>
      <td>Spencer Ware</td>
      <td>Jiwei</td>
      <td>Buf</td>
      <td>RB</td>
      <td>5</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Dallas Goedert</td>
      <td>Jesse James</td>
      <td>Sean</td>
      <td>Phi</td>
      <td>TE</td>
      <td>4</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>46</th>
      <td>Austin Hooper</td>
      <td>Anthony Miller</td>
      <td>Andrew</td>
      <td>Atl</td>
      <td>TE</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Seattle</td>
      <td>Dallas</td>
      <td>Sean</td>
      <td>Sea</td>
      <td>DEF</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Green Bay</td>
      <td>Minnesota</td>
      <td>Dai</td>
      <td>GB</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Tennessee</td>
      <td>Houston</td>
      <td>Ron</td>
      <td>Ten</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>50</th>
      <td>Albert Wilson</td>
      <td>Pierre Garcon</td>
      <td>Doug</td>
      <td>Mia</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Rhett Ellison</td>
      <td>Tyler Eifert</td>
      <td>Doug</td>
      <td>NYG</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Vance McDonald</td>
      <td>Rex Burkhead</td>
      <td>Jiwei</td>
      <td>Pit</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>53</th>
      <td>Tavon Austin</td>
      <td>Rishard Matthews</td>
      <td>Ryan</td>
      <td>Dal</td>
      <td>WR,RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>54</th>
      <td>Randy Bullock</td>
      <td>Josh Lambo</td>
      <td>Sean</td>
      <td>Cin</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>55</th>
      <td>Austin Ekeler</td>
      <td>Malcolm Brown</td>
      <td>Dai</td>
      <td>LAC</td>
      <td>RB</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>56</th>
      <td>Tyler Eifert</td>
      <td>Free Agent</td>
      <td>Andrew</td>
      <td>Cin</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>57</th>
      <td>Mike Davis</td>
      <td>David Njoku</td>
      <td>Jiwei</td>
      <td>Sea</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>58</th>
      <td>D.J. Moore</td>
      <td>Jamaal Williams</td>
      <td>Dai</td>
      <td>Car</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>59</th>
      <td>David Njoku</td>
      <td>Tyler Eifert</td>
      <td>Andrew</td>
      <td>Cle</td>
      <td>TE</td>
      <td>10</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>60</th>
      <td>Taylor Gabriel</td>
      <td>Rashaad Penny</td>
      <td>Jake</td>
      <td>Chi</td>
      <td>WR</td>
      <td>5</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>61</th>
      <td>Keke Coutee</td>
      <td>Sam Darnold</td>
      <td>Matt</td>
      <td>Hou</td>
      <td>WR</td>
      <td>4</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>62</th>
      <td>Cameron Brate</td>
      <td>Ryan Tannehill</td>
      <td>Doug</td>
      <td>TB</td>
      <td>TE</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>63</th>
      <td>Ryan Grant</td>
      <td>Chris Hogan</td>
      <td>Sean</td>
      <td>Ind</td>
      <td>WR</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>64</th>
      <td>Carolina</td>
      <td>Seattle</td>
      <td>Sean</td>
      <td>Car</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>65</th>
      <td>Denver</td>
      <td>Los Angeles</td>
      <td>Doug</td>
      <td>Den</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>66</th>
      <td>Marlon Mack</td>
      <td>Jordan Wilkins</td>
      <td>Dai</td>
      <td>Ind</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Taywan Taylor</td>
      <td>Matt Prater</td>
      <td>Dai</td>
      <td>Ten</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>68</th>
      <td>Austin Hooper</td>
      <td>Alfred Morris</td>
      <td>Dai</td>
      <td>Atl</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>69</th>
      <td>Cincinnati</td>
      <td>Green Bay</td>
      <td>Dai</td>
      <td>Cin</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>70</th>
      <td>Mohamed Sanu</td>
      <td>Ryan Fitzpatrick</td>
      <td>Sean</td>
      <td>Atl</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>71</th>
      <td>Rod Smith</td>
      <td>Marlon Mack</td>
      <td>Dai</td>
      <td>Dal</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>72</th>
      <td>Geoff Swaim</td>
      <td>Albert Wilson</td>
      <td>Doug</td>
      <td>Dal</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>73</th>
      <td>C.J. Uzomah</td>
      <td>Will Dissly</td>
      <td>Chi Shing</td>
      <td>Cin</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>74</th>
      <td>Marquez Valdes-Scantling</td>
      <td>Austin Hooper</td>
      <td>Dai</td>
      <td>GB</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>75</th>
      <td>San Francisco</td>
      <td>Tavon Austin</td>
      <td>Ryan</td>
      <td>SF</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>76</th>
      <td>Ryan Succop</td>
      <td>Chandler Catanzaro</td>
      <td>Jiwei</td>
      <td>Ten</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Corey Grant</td>
      <td>Free Agent</td>
      <td>Rajiv</td>
      <td>Jax</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Houston</td>
      <td>Atlanta</td>
      <td>Rajiv</td>
      <td>Hou</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>79</th>
      <td>Rashaad Penny</td>
      <td>Rod Smith</td>
      <td>Dai</td>
      <td>Sea</td>
      <td>RB</td>
      <td>5</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Austin Hooper</td>
      <td>Robbie Gould</td>
      <td>Rajiv</td>
      <td>Atl</td>
      <td>TE</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>81</th>
      <td>Alfred Blue</td>
      <td>Chris Ivory</td>
      <td>Jiwei</td>
      <td>Hou</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>82</th>
      <td>Brett Maher</td>
      <td>Taywan Taylor</td>
      <td>Dai</td>
      <td>Dal</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>83</th>
      <td>Jameis Winston</td>
      <td>Nelson Agholor</td>
      <td>Dai</td>
      <td>TB</td>
      <td>QB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>84</th>
      <td>Wendell Smallwood</td>
      <td>Ryan Grant</td>
      <td>Sean</td>
      <td>Phi</td>
      <td>RB</td>
      <td>21</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>85</th>
      <td>Marlon Mack</td>
      <td>Giovani Bernard</td>
      <td>Ron</td>
      <td>Ind</td>
      <td>RB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>86</th>
      <td>D'Onta Foreman</td>
      <td>$2  Waiver</td>
      <td>Sean</td>
      <td>Hou</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>87</th>
      <td>Minnesota</td>
      <td>Carolina</td>
      <td>Sean</td>
      <td>Min</td>
      <td>DEF</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>88</th>
      <td>David Moore</td>
      <td>Peyton Barber</td>
      <td>Chi Shing</td>
      <td>Sea</td>
      <td>WR</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>89</th>
      <td>Green Bay</td>
      <td>Cincinnati</td>
      <td>Dai</td>
      <td>GB</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>90</th>
      <td>Greg Zuerlein</td>
      <td>Mason Crosby</td>
      <td>Matt</td>
      <td>LAR</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>91</th>
      <td>Ka'imi Fairbairn</td>
      <td>Brett Maher</td>
      <td>Dai</td>
      <td>Hou</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>92</th>
      <td>Los Angeles</td>
      <td>Tennessee</td>
      <td>Ron</td>
      <td>LAC</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>93</th>
      <td>Sebastian Janikowski</td>
      <td>Greg Zuerlein</td>
      <td>Matt</td>
      <td>Sea</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>94</th>
      <td>Ito Smith</td>
      <td>Rhett Ellison</td>
      <td>Doug</td>
      <td>Atl</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>95</th>
      <td>Seattle</td>
      <td>Denver</td>
      <td>Doug</td>
      <td>Sea</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Mason Crosby</td>
      <td>Robby Anderson</td>
      <td>Evan</td>
      <td>GB</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>97</th>
      <td>Kapri Bibbs</td>
      <td>Marquez Valdes-Scantling</td>
      <td>Dai</td>
      <td>Was</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>98</th>
      <td>Alfred Morris</td>
      <td>Kapri Bibbs</td>
      <td>Dai</td>
      <td>SF</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Kyle Juszczyk</td>
      <td>Ted Ginn Jr.</td>
      <td>Doug</td>
      <td>SF</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>100</th>
      <td>Mitchell Trubisky</td>
      <td>Derrick Henry</td>
      <td>Andrew</td>
      <td>Chi</td>
      <td>QB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>101</th>
      <td>Chester Rogers</td>
      <td>Geoff Swaim</td>
      <td>Doug</td>
      <td>Ind</td>
      <td>WR</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>102</th>
      <td>Greg Zuerlein</td>
      <td>Matt Bryant</td>
      <td>Ron</td>
      <td>LAR</td>
      <td>K</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Giovani Bernard</td>
      <td>Mason Crosby</td>
      <td>Evan</td>
      <td>Cin</td>
      <td>RB</td>
      <td>5</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>104</th>
      <td>Raheem Mostert</td>
      <td>Arizona</td>
      <td>Evan</td>
      <td>SF</td>
      <td>RB</td>
      <td>4</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>105</th>
      <td>Willie Snead IV</td>
      <td>Randall Cobb</td>
      <td>Jake</td>
      <td>Bal</td>
      <td>WR</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>106</th>
      <td>Ricky Seals-Jones</td>
      <td>Quincy Enunwa</td>
      <td>Ron</td>
      <td>Ari</td>
      <td>TE</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>107</th>
      <td>New York</td>
      <td>Seattle</td>
      <td>Doug</td>
      <td>NYJ</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>108</th>
      <td>Indianapolis</td>
      <td>Minnesota</td>
      <td>Sean</td>
      <td>Ind</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>109</th>
      <td>Brett Maher</td>
      <td>Randy Bullock</td>
      <td>Sean</td>
      <td>Dal</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>110</th>
      <td>Adam Vinatieri</td>
      <td>Chris Boswell</td>
      <td>Chi Shing</td>
      <td>Ind</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>111</th>
      <td>Nelson Agholor</td>
      <td>Jordan Matthews</td>
      <td>Chi Shing</td>
      <td>Phi</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>112</th>
      <td>Tre'Quan Smith</td>
      <td>Alfred Morris</td>
      <td>Dai</td>
      <td>NO</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>113</th>
      <td>Buffalo</td>
      <td>Green Bay</td>
      <td>Dai</td>
      <td>Buf</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>114</th>
      <td>Peyton Barber</td>
      <td>Tre'Quan Smith</td>
      <td>Dai</td>
      <td>TB</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>115</th>
      <td>Giorgio Tavecchio</td>
      <td>Sebastian Janikowski</td>
      <td>Matt</td>
      <td>Atl</td>
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
      <td>DeSean Jackson</td>
      <td>DeMarco Murray</td>
      <td>Jake</td>
      <td>TB</td>
      <td>WR</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>1.0</th>
      <td>Denver</td>
      <td>New Orleans</td>
      <td>Sean</td>
      <td>Den</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>2.0</th>
      <td>Dante Pettis</td>
      <td>D.J. Moore</td>
      <td>Sean</td>
      <td>SF</td>
      <td>WR</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>3.0</th>
      <td>Tyrod Taylor</td>
      <td>Mike Wallace</td>
      <td>Jiwei</td>
      <td>Cle</td>
      <td>QB</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>4.0</th>
      <td>Trent Taylor</td>
      <td>Josh Doctson</td>
      <td>Ron</td>
      <td>SF</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>5.0</th>
      <td>Adam Vinatieri</td>
      <td>Graham Gano</td>
      <td>Sean</td>
      <td>Ind</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>6.0</th>
      <td>Brandon Marshall</td>
      <td>Brandon McManus</td>
      <td>Dai</td>
      <td>Sea</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>7.0</th>
      <td>Seattle</td>
      <td>Green Bay</td>
      <td>Ron</td>
      <td>Sea</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>8.0</th>
      <td>Ryan Grant</td>
      <td>Trent Taylor</td>
      <td>Ron</td>
      <td>Ind</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>9.0</th>
      <td>Phillip Dorsett</td>
      <td>Greg Olsen</td>
      <td>Chi Shing</td>
      <td>NE</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>10.0</th>
      <td>Javorius Allen</td>
      <td>Ben Roethlisberger</td>
      <td>Dai</td>
      <td>Bal</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>11.0</th>
      <td>Matt Breida</td>
      <td>Allen Hurns</td>
      <td>Ron</td>
      <td>SF</td>
      <td>RB</td>
      <td>11</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>12.0</th>
      <td>Houston</td>
      <td>Seattle</td>
      <td>Ron</td>
      <td>Hou</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>13.0</th>
      <td>Ben Roethlisberger</td>
      <td>Jared Goff</td>
      <td>Ron</td>
      <td>Pit</td>
      <td>QB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>14.0</th>
      <td>Jalen Richard</td>
      <td>John Ross</td>
      <td>Dai</td>
      <td>Oak</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>15.0</th>
      <td>Jonnu Smith</td>
      <td>Austin Seferian-Jenkins</td>
      <td>Ron</td>
      <td>Ten</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>16.0</th>
      <td>Antonio Callaway</td>
      <td>Cameron Meredith</td>
      <td>Andrew</td>
      <td>Cle</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>17.0</th>
      <td>Will Dissly</td>
      <td>Free Agent</td>
      <td>Chi Shing</td>
      <td>Sea</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>18.0</th>
      <td>Cody Parkey</td>
      <td>Jalen Richard</td>
      <td>Dai</td>
      <td>Chi</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>19.0</th>
      <td>Ryan Fitzpatrick</td>
      <td>Ted Ginn Jr.</td>
      <td>Sean</td>
      <td>TB</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>20.0</th>
      <td>Giovani Bernard</td>
      <td>Ryan Grant</td>
      <td>Ron</td>
      <td>Cin</td>
      <td>RB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>21.0</th>
      <td>Dan Bailey</td>
      <td>Daniel Carlson</td>
      <td>Ryan</td>
      <td>Min</td>
      <td>K</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>22.0</th>
      <td>Jesse James</td>
      <td>Dante Pettis</td>
      <td>Sean</td>
      <td>Pit</td>
      <td>TE</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>23.0</th>
      <td>Jared Goff</td>
      <td>Marcus Mariota</td>
      <td>Matt</td>
      <td>LAR</td>
      <td>QB</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>24.0</th>
      <td>Dallas</td>
      <td>Denver</td>
      <td>Sean</td>
      <td>Dal</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>25.0</th>
      <td>Josh Lambo</td>
      <td>Adam Vinatieri</td>
      <td>Sean</td>
      <td>Jax</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>26.0</th>
      <td>Tyler Boyd</td>
      <td>Jonnu Smith</td>
      <td>Ron</td>
      <td>Cin</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>27.0</th>
      <td>Mark Walton</td>
      <td>Cody Parkey</td>
      <td>Dai</td>
      <td>Cin</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>28.0</th>
      <td>Malcolm Brown</td>
      <td>Brandon Marshall</td>
      <td>Dai</td>
      <td>LAR</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>29.0</th>
      <td>Frank Gore</td>
      <td>Antonio Gates</td>
      <td>Chi Shing</td>
      <td>Mia</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>30.0</th>
      <td>Cleveland</td>
      <td>Carolina</td>
      <td>Chi Shing</td>
      <td>Cle</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>31.0</th>
      <td>Greg Olsen</td>
      <td>Ricky Seals-Jones</td>
      <td>Jake</td>
      <td>Car</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>32.0</th>
      <td>Corey Grant</td>
      <td>Free Agent</td>
      <td>Dai</td>
      <td>Jax</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>33.0</th>
      <td>Ted Ginn Jr.</td>
      <td>Michael Gallup</td>
      <td>Doug</td>
      <td>NO</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>34.0</th>
      <td>Ito Smith</td>
      <td>Alex Smith</td>
      <td>Ron</td>
      <td>Atl</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>35.0</th>
      <td>Nyheim Hines</td>
      <td>Ronald Jones II</td>
      <td>Andrew</td>
      <td>Ind</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>36.0</th>
      <td>Theo Riddick</td>
      <td>Darren Sproles</td>
      <td>Chi Shing</td>
      <td>Det</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>37.0</th>
      <td>Chandler Catanzaro</td>
      <td>Greg Zuerlein</td>
      <td>Jiwei</td>
      <td>TB</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>38.0</th>
      <td>Matt Prater</td>
      <td>Corey Grant</td>
      <td>Dai</td>
      <td>Det</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>39.0</th>
      <td>Christian Kirk</td>
      <td>Mark Walton</td>
      <td>Dai</td>
      <td>Ari</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>40.0</th>
      <td>Ronald Jones II</td>
      <td>Jamison Crowder</td>
      <td>Dai</td>
      <td>TB</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>41.0</th>
      <td>Jordy Nelson</td>
      <td>Tyrod Taylor</td>
      <td>Jiwei</td>
      <td>Oak</td>
      <td>WR</td>
      <td>7</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>42.0</th>
      <td>Ryan Tannehill</td>
      <td>Doug Martin</td>
      <td>Doug</td>
      <td>Mia</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>43.0</th>
      <td>Baker Mayfield</td>
      <td>Austin Ekeler</td>
      <td>Sean</td>
      <td>Cle</td>
      <td>QB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>44.0</th>
      <td>Chris Ivory</td>
      <td>Spencer Ware</td>
      <td>Jiwei</td>
      <td>Buf</td>
      <td>RB</td>
      <td>5</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>45.0</th>
      <td>Dallas Goedert</td>
      <td>Jesse James</td>
      <td>Sean</td>
      <td>Phi</td>
      <td>TE</td>
      <td>4</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>46.0</th>
      <td>Austin Hooper</td>
      <td>Anthony Miller</td>
      <td>Andrew</td>
      <td>Atl</td>
      <td>TE</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>47.0</th>
      <td>Seattle</td>
      <td>Dallas</td>
      <td>Sean</td>
      <td>Sea</td>
      <td>DEF</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>48.0</th>
      <td>Green Bay</td>
      <td>Minnesota</td>
      <td>Dai</td>
      <td>GB</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>49.0</th>
      <td>Tennessee</td>
      <td>Houston</td>
      <td>Ron</td>
      <td>Ten</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>50.0</th>
      <td>Albert Wilson</td>
      <td>Pierre Garcon</td>
      <td>Doug</td>
      <td>Mia</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>51.0</th>
      <td>Rhett Ellison</td>
      <td>Tyler Eifert</td>
      <td>Doug</td>
      <td>NYG</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>52.0</th>
      <td>Vance McDonald</td>
      <td>Rex Burkhead</td>
      <td>Jiwei</td>
      <td>Pit</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>53.0</th>
      <td>Tavon Austin</td>
      <td>Rishard Matthews</td>
      <td>Ryan</td>
      <td>Dal</td>
      <td>WR,RB</td>
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
      <td>Randy Bullock</td>
      <td>Josh Lambo</td>
      <td>Sean</td>
      <td>Cin</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>55.0</th>
      <td>Austin Ekeler</td>
      <td>Malcolm Brown</td>
      <td>Dai</td>
      <td>LAC</td>
      <td>RB</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>56.0</th>
      <td>Tyler Eifert</td>
      <td>Free Agent</td>
      <td>Andrew</td>
      <td>Cin</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>57.0</th>
      <td>Mike Davis</td>
      <td>David Njoku</td>
      <td>Jiwei</td>
      <td>Sea</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>58.0</th>
      <td>D.J. Moore</td>
      <td>Jamaal Williams</td>
      <td>Dai</td>
      <td>Car</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>59.0</th>
      <td>David Njoku</td>
      <td>Tyler Eifert</td>
      <td>Andrew</td>
      <td>Cle</td>
      <td>TE</td>
      <td>10</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>60.0</th>
      <td>Taylor Gabriel</td>
      <td>Rashaad Penny</td>
      <td>Jake</td>
      <td>Chi</td>
      <td>WR</td>
      <td>5</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>61.0</th>
      <td>Keke Coutee</td>
      <td>Sam Darnold</td>
      <td>Matt</td>
      <td>Hou</td>
      <td>WR</td>
      <td>4</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>62.0</th>
      <td>Cameron Brate</td>
      <td>Ryan Tannehill</td>
      <td>Doug</td>
      <td>TB</td>
      <td>TE</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>63.0</th>
      <td>Ryan Grant</td>
      <td>Chris Hogan</td>
      <td>Sean</td>
      <td>Ind</td>
      <td>WR</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>64.0</th>
      <td>Carolina</td>
      <td>Seattle</td>
      <td>Sean</td>
      <td>Car</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>65.0</th>
      <td>Denver</td>
      <td>Los Angeles</td>
      <td>Doug</td>
      <td>Den</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>66.0</th>
      <td>Marlon Mack</td>
      <td>Jordan Wilkins</td>
      <td>Dai</td>
      <td>Ind</td>
      <td>RB</td>
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
      <td>Taywan Taylor</td>
      <td>Matt Prater</td>
      <td>Dai</td>
      <td>Ten</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>68.0</th>
      <td>Austin Hooper</td>
      <td>Alfred Morris</td>
      <td>Dai</td>
      <td>Atl</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>69.0</th>
      <td>Cincinnati</td>
      <td>Green Bay</td>
      <td>Dai</td>
      <td>Cin</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>70.0</th>
      <td>Mohamed Sanu</td>
      <td>Ryan Fitzpatrick</td>
      <td>Sean</td>
      <td>Atl</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>71.0</th>
      <td>Rod Smith</td>
      <td>Marlon Mack</td>
      <td>Dai</td>
      <td>Dal</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>72.0</th>
      <td>Geoff Swaim</td>
      <td>Albert Wilson</td>
      <td>Doug</td>
      <td>Dal</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>73.0</th>
      <td>C.J. Uzomah</td>
      <td>Will Dissly</td>
      <td>Chi Shing</td>
      <td>Cin</td>
      <td>TE</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>74.0</th>
      <td>Marquez Valdes-Scantling</td>
      <td>Austin Hooper</td>
      <td>Dai</td>
      <td>GB</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>75.0</th>
      <td>San Francisco</td>
      <td>Tavon Austin</td>
      <td>Ryan</td>
      <td>SF</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>76.0</th>
      <td>Ryan Succop</td>
      <td>Chandler Catanzaro</td>
      <td>Jiwei</td>
      <td>Ten</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>77.0</th>
      <td>Corey Grant</td>
      <td>Free Agent</td>
      <td>Rajiv</td>
      <td>Jax</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>78.0</th>
      <td>Houston</td>
      <td>Atlanta</td>
      <td>Rajiv</td>
      <td>Hou</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>79.0</th>
      <td>Rashaad Penny</td>
      <td>Rod Smith</td>
      <td>Dai</td>
      <td>Sea</td>
      <td>RB</td>
      <td>5</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>80.0</th>
      <td>Austin Hooper</td>
      <td>Robbie Gould</td>
      <td>Rajiv</td>
      <td>Atl</td>
      <td>TE</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>81.0</th>
      <td>Alfred Blue</td>
      <td>Chris Ivory</td>
      <td>Jiwei</td>
      <td>Hou</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>82.0</th>
      <td>Brett Maher</td>
      <td>Taywan Taylor</td>
      <td>Dai</td>
      <td>Dal</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>83.0</th>
      <td>Jameis Winston</td>
      <td>Nelson Agholor</td>
      <td>Dai</td>
      <td>TB</td>
      <td>QB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>84.0</th>
      <td>Wendell Smallwood</td>
      <td>Ryan Grant</td>
      <td>Sean</td>
      <td>Phi</td>
      <td>RB</td>
      <td>21</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>85.0</th>
      <td>Marlon Mack</td>
      <td>Giovani Bernard</td>
      <td>Ron</td>
      <td>Ind</td>
      <td>RB</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>86.0</th>
      <td>D'Onta Foreman</td>
      <td>$2  Waiver</td>
      <td>Sean</td>
      <td>Hou</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>87.0</th>
      <td>Minnesota</td>
      <td>Carolina</td>
      <td>Sean</td>
      <td>Min</td>
      <td>DEF</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>88.0</th>
      <td>David Moore</td>
      <td>Peyton Barber</td>
      <td>Chi Shing</td>
      <td>Sea</td>
      <td>WR</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>89.0</th>
      <td>Green Bay</td>
      <td>Cincinnati</td>
      <td>Dai</td>
      <td>GB</td>
      <td>DEF</td>
      <td>1</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>90.0</th>
      <td>Greg Zuerlein</td>
      <td>Mason Crosby</td>
      <td>Matt</td>
      <td>LAR</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>91.0</th>
      <td>Ka'imi Fairbairn</td>
      <td>Brett Maher</td>
      <td>Dai</td>
      <td>Hou</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>92.0</th>
      <td>Los Angeles</td>
      <td>Tennessee</td>
      <td>Ron</td>
      <td>LAC</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>93.0</th>
      <td>Sebastian Janikowski</td>
      <td>Greg Zuerlein</td>
      <td>Matt</td>
      <td>Sea</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>94.0</th>
      <td>Ito Smith</td>
      <td>Rhett Ellison</td>
      <td>Doug</td>
      <td>Atl</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>95.0</th>
      <td>Seattle</td>
      <td>Denver</td>
      <td>Doug</td>
      <td>Sea</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>96.0</th>
      <td>Mason Crosby</td>
      <td>Robby Anderson</td>
      <td>Evan</td>
      <td>GB</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>97.0</th>
      <td>Kapri Bibbs</td>
      <td>Marquez Valdes-Scantling</td>
      <td>Dai</td>
      <td>Was</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>98.0</th>
      <td>Alfred Morris</td>
      <td>Kapri Bibbs</td>
      <td>Dai</td>
      <td>SF</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>99.0</th>
      <td>Kyle Juszczyk</td>
      <td>Ted Ginn Jr.</td>
      <td>Doug</td>
      <td>SF</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>100.0</th>
      <td>Mitchell Trubisky</td>
      <td>Derrick Henry</td>
      <td>Andrew</td>
      <td>Chi</td>
      <td>QB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>101.0</th>
      <td>Chester Rogers</td>
      <td>Geoff Swaim</td>
      <td>Doug</td>
      <td>Ind</td>
      <td>WR</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>102.0</th>
      <td>Greg Zuerlein</td>
      <td>Matt Bryant</td>
      <td>Ron</td>
      <td>LAR</td>
      <td>K</td>
      <td>6</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>103.0</th>
      <td>Giovani Bernard</td>
      <td>Mason Crosby</td>
      <td>Evan</td>
      <td>Cin</td>
      <td>RB</td>
      <td>5</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>104.0</th>
      <td>Raheem Mostert</td>
      <td>Arizona</td>
      <td>Evan</td>
      <td>SF</td>
      <td>RB</td>
      <td>4</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>105.0</th>
      <td>Willie Snead IV</td>
      <td>Randall Cobb</td>
      <td>Jake</td>
      <td>Bal</td>
      <td>WR</td>
      <td>3</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>106.0</th>
      <td>Ricky Seals-Jones</td>
      <td>Quincy Enunwa</td>
      <td>Ron</td>
      <td>Ari</td>
      <td>TE</td>
      <td>2</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>107.0</th>
      <td>New York</td>
      <td>Seattle</td>
      <td>Doug</td>
      <td>NYJ</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>108.0</th>
      <td>Indianapolis</td>
      <td>Minnesota</td>
      <td>Sean</td>
      <td>Ind</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>109.0</th>
      <td>Brett Maher</td>
      <td>Randy Bullock</td>
      <td>Sean</td>
      <td>Dal</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>110.0</th>
      <td>Adam Vinatieri</td>
      <td>Chris Boswell</td>
      <td>Chi Shing</td>
      <td>Ind</td>
      <td>K</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>111.0</th>
      <td>Nelson Agholor</td>
      <td>Jordan Matthews</td>
      <td>Chi Shing</td>
      <td>Phi</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>112.0</th>
      <td>Tre'Quan Smith</td>
      <td>Alfred Morris</td>
      <td>Dai</td>
      <td>NO</td>
      <td>WR</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>113.0</th>
      <td>Buffalo</td>
      <td>Green Bay</td>
      <td>Dai</td>
      <td>Buf</td>
      <td>DEF</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>114.0</th>
      <td>Peyton Barber</td>
      <td>Tre'Quan Smith</td>
      <td>Dai</td>
      <td>TB</td>
      <td>RB</td>
      <td>0</td>
      <td>waiver</td>
    </tr>
    <tr>
      <th>115.0</th>
      <td>Giorgio Tavecchio</td>
      <td>Sebastian Janikowski</td>
      <td>Matt</td>
      <td>Atl</td>
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
      <td>Douglast</td>
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
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Daized and Confused</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>No bunny no fu fu</td>
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
      <td>Douglast</td>
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
      <td>No bunny no fu fu</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>14</th>
      <td>LAC</td>
      <td>WR</td>
      <td>71</td>
      <td>Keenan Allen</td>
      <td>Daized and Confused</td>
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
      <td>Free-Mangina</td>
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
      <td>Daized and Confused</td>
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
      <td>TB</td>
      <td>RB</td>
      <td>0</td>
      <td>Peyton Barber</td>
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>23</th>
      <td>LAR</td>
      <td>WR</td>
      <td>39</td>
      <td>Brandin Cooks</td>
      <td>Douglast</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>24</th>
      <td>TB</td>
      <td>WR</td>
      <td>2</td>
      <td>DeSean Jackson</td>
      <td>Douglast</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Car</td>
      <td>RB</td>
      <td>67</td>
      <td>Christian McCaffrey</td>
      <td>Yeebola</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Phi</td>
      <td>TE</td>
      <td>49</td>
      <td>Zach Ertz</td>
      <td>Douglast</td>
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
      <td>Yeebola</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>32</th>
      <td>GB</td>
      <td>QB</td>
      <td>51</td>
      <td>Aaron Rodgers</td>
      <td>Free-Mangina</td>
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
      <td>Free-Mangina</td>
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
      <td>Phi</td>
      <td>RB</td>
      <td>21</td>
      <td>Wendell Smallwood</td>
      <td>Daized and Confused</td>
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
      <td>Yeebola</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>42</th>
      <td>TB</td>
      <td>RB</td>
      <td>0</td>
      <td>Ronald Jones II</td>
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>43</th>
      <td>NE</td>
      <td>WR</td>
      <td>19</td>
      <td>Julian Edelman</td>
      <td>No bunny no fu fu</td>
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
      <td>Buf</td>
      <td>DEF</td>
      <td>0</td>
      <td>Buffalo</td>
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Oak</td>
      <td>WR</td>
      <td>19</td>
      <td>Jordy Nelson</td>
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
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>50</th>
      <td>Ten</td>
      <td>RB</td>
      <td>29</td>
      <td>Dion Lewis</td>
      <td>Free-Mangina</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>54</th>
      <td>Hou</td>
      <td>K</td>
      <td>0</td>
      <td>Ka'imi Fairbairn</td>
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
      <td>NYJ</td>
      <td>DEF</td>
      <td>0</td>
      <td>New York</td>
      <td>Douglast</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>59</th>
      <td>Ind</td>
      <td>WR</td>
      <td>6</td>
      <td>Chester Rogers</td>
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
      <td>Sea</td>
      <td>RB</td>
      <td>0</td>
      <td>Mike Davis</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>62</th>
      <td>Ten</td>
      <td>TE</td>
      <td>23</td>
      <td>Delanie Walker</td>
      <td>Daized and Confused</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>63</th>
      <td>SF</td>
      <td>RB</td>
      <td>8</td>
      <td>Matt Breida</td>
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
      <td>SF</td>
      <td>RB</td>
      <td>0</td>
      <td>Kyle Juszczyk</td>
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
      <td>No bunny no fu fu</td>
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
      <td>Cin</td>
      <td>RB</td>
      <td>5</td>
      <td>Giovani Bernard</td>
      <td>Cry me a Philip</td>
      <td></td>
      <td>Evan</td>
    </tr>
    <tr>
      <th>70</th>
      <td>NYJ</td>
      <td>DEF</td>
      <td>0</td>
      <td>New York</td>
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
      <td>Atl</td>
      <td>RB</td>
      <td>0</td>
      <td>Ito Smith</td>
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
      <td>Daized and Confused</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>76</th>
      <td>Pit</td>
      <td>QB</td>
      <td>0</td>
      <td>Ben Roethlisberger</td>
      <td>Yeebola</td>
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
      <td>Jax</td>
      <td>WR</td>
      <td>5</td>
      <td>Dede Westbrook</td>
      <td>Yeebola</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>79</th>
      <td>TB</td>
      <td>RB</td>
      <td>0</td>
      <td>Peyton Barber</td>
      <td>No bunny no fu fu</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Chi</td>
      <td>WR</td>
      <td>5</td>
      <td>Taylor Gabriel</td>
      <td>Douglast</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>81</th>
      <td>TB</td>
      <td>QB</td>
      <td>0</td>
      <td>Jameis Winston</td>
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>82</th>
      <td>LAC</td>
      <td>DEF</td>
      <td>0</td>
      <td>Los Angeles</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>84</th>
      <td>NYJ</td>
      <td>RB</td>
      <td>7</td>
      <td>Bilal Powell</td>
      <td>Douglast</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>85</th>
      <td>Hou</td>
      <td>RB</td>
      <td>0</td>
      <td>Alfred Blue</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>86</th>
      <td>Cle</td>
      <td>RB</td>
      <td>5</td>
      <td>Duke Johnson Jr.</td>
      <td>Daized and Confused</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>87</th>
      <td>Dal</td>
      <td>WR</td>
      <td>8</td>
      <td>Dez Bryant</td>
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>88</th>
      <td>Det</td>
      <td>RB</td>
      <td>10</td>
      <td>Kerryon Johnson</td>
      <td>No bunny no fu fu</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>89</th>
      <td>Ten</td>
      <td>K</td>
      <td>0</td>
      <td>Ryan Succop</td>
      <td>G</td>
      <td></td>
      <td>Jiwei</td>
    </tr>
    <tr>
      <th>90</th>
      <td>Car</td>
      <td>WR</td>
      <td>0</td>
      <td>D.J. Moore</td>
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>91</th>
      <td>Chi</td>
      <td>DEF</td>
      <td>3</td>
      <td>Chicago</td>
      <td>No bunny no fu fu</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>94</th>
      <td>Bal</td>
      <td>K</td>
      <td>4</td>
      <td>Justin Tucker</td>
      <td>Douglast</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>95</th>
      <td>NE</td>
      <td>RB</td>
      <td>7</td>
      <td>Sony Michel</td>
      <td>No bunny no fu fu</td>
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
      <td>Ind</td>
      <td>DEF</td>
      <td>0</td>
      <td>Indianapolis</td>
      <td>Daized and Confused</td>
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
      <td>No bunny no fu fu</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>101</th>
      <td>Ind</td>
      <td>DEF</td>
      <td>0</td>
      <td>Indianapolis</td>
      <td>Yeebola</td>
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
      <td>No bunny no fu fu</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>104</th>
      <td>Hou</td>
      <td>WR</td>
      <td>4</td>
      <td>Keke Coutee</td>
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
      <td>LAR</td>
      <td>K</td>
      <td>6</td>
      <td>Greg Zuerlein</td>
      <td>Yeebola</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>107</th>
      <td>SF</td>
      <td>DEF</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>108</th>
      <td>Car</td>
      <td>TE</td>
      <td>0</td>
      <td>Greg Olsen</td>
      <td>Douglast</td>
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
      <td>Daized and Confused</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>111</th>
      <td>Det</td>
      <td>RB</td>
      <td>2</td>
      <td>LeGarrette Blount</td>
      <td>Yeebola</td>
      <td></td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>112</th>
      <td>Ind</td>
      <td>RB</td>
      <td>6</td>
      <td>Marlon Mack</td>
      <td>Yeebola</td>
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
      <td>Buf</td>
      <td>TE</td>
      <td>1</td>
      <td>Charles Clay</td>
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>115</th>
      <td>Det</td>
      <td>WR</td>
      <td>5</td>
      <td>Kenny Golladay</td>
      <td>No bunny no fu fu</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>116</th>
      <td>Cle</td>
      <td>QB</td>
      <td>1</td>
      <td>Baker Mayfield</td>
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
      <td>No bunny no fu fu</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>119</th>
      <td>Cle</td>
      <td>RB</td>
      <td>3</td>
      <td>Nick Chubb</td>
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>120</th>
      <td>Pit</td>
      <td>DEF</td>
      <td>1</td>
      <td>Pittsburgh</td>
      <td>Douglast</td>
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
      <td>TB</td>
      <td>TE</td>
      <td>3</td>
      <td>Cameron Brate</td>
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
      <td>Seandon</td>
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
      <td>Sea</td>
      <td>RB</td>
      <td>5</td>
      <td>Rashaad Penny</td>
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>126</th>
      <td>Hou</td>
      <td>DEF</td>
      <td>0</td>
      <td>Houston</td>
      <td>FirstRoundFlops</td>
      <td></td>
      <td>Rajiv</td>
    </tr>
    <tr>
      <th>127</th>
      <td>Ind</td>
      <td>K</td>
      <td>0</td>
      <td>Adam Vinatieri</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>129</th>
      <td>Bal</td>
      <td>WR</td>
      <td>3</td>
      <td>Willie Snead IV</td>
      <td>Douglast</td>
      <td></td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>130</th>
      <td>Jax</td>
      <td>WR</td>
      <td>1</td>
      <td>Donte Moncrief</td>
      <td>Daized and Confused</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>131</th>
      <td>Atl</td>
      <td>RB</td>
      <td>0</td>
      <td>Ito Smith</td>
      <td>Yeebola</td>
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
      <td>Sea</td>
      <td>RB</td>
      <td>5</td>
      <td>Rashaad Penny</td>
      <td>Seandon</td>
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
      <td>TB</td>
      <td>TE</td>
      <td>1</td>
      <td>Cameron Brate</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>137</th>
      <td>TB</td>
      <td>QB</td>
      <td>1</td>
      <td>Jameis Winston</td>
      <td>Daized and Confused</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>138</th>
      <td>Cin</td>
      <td>WR</td>
      <td>0</td>
      <td>Tyler Boyd</td>
      <td>Yeebola</td>
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
      <td>Daized and Confused</td>
      <td></td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>143</th>
      <td>LAC</td>
      <td>DEF</td>
      <td>0</td>
      <td>Los Angeles</td>
      <td>Yeebola</td>
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
      <td>SF</td>
      <td>RB</td>
      <td>4</td>
      <td>Raheem Mostert</td>
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
      <td>Atl</td>
      <td>TE</td>
      <td>6</td>
      <td>Austin Hooper</td>
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
      <td>Daized and Confused</td>
      <td>B</td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>153</th>
      <td>NO</td>
      <td>RB</td>
      <td>9</td>
      <td>Alvin Kamara</td>
      <td>Daized and Confused</td>
      <td>B</td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>154</th>
      <td>TB</td>
      <td>WR</td>
      <td>30</td>
      <td>Mike Evans</td>
      <td>Daized and Confused</td>
      <td>AB</td>
      <td>Sean</td>
    </tr>
    <tr>
      <th>155</th>
      <td>Ind</td>
      <td>QB</td>
      <td>5</td>
      <td>Andrew Luck</td>
      <td>Daized and Confused</td>
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
      <td>Yeebola</td>
      <td>AB</td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>164</th>
      <td>NYG</td>
      <td>WR</td>
      <td>29</td>
      <td>Odell Beckham Jr.</td>
      <td>Yeebola</td>
      <td>AB</td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>165</th>
      <td>NE</td>
      <td>WR</td>
      <td>6</td>
      <td>Josh Gordon</td>
      <td>Yeebola</td>
      <td>B</td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Pit</td>
      <td>RB</td>
      <td>66</td>
      <td>Le'Veon Bell</td>
      <td>Yeebola</td>
      <td>AB</td>
      <td>Ron</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Mia</td>
      <td>RB</td>
      <td>17</td>
      <td>Kenyan Drake</td>
      <td>Seandon</td>
      <td>B</td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>168</th>
      <td>GB</td>
      <td>WR</td>
      <td>27</td>
      <td>Davante Adams</td>
      <td>Seandon</td>
      <td>AB</td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>169</th>
      <td>LAC</td>
      <td>RB</td>
      <td>39</td>
      <td>Melvin Gordon III</td>
      <td>Seandon</td>
      <td>AB</td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>170</th>
      <td>Hou</td>
      <td>WR</td>
      <td>56</td>
      <td>DeAndre Hopkins</td>
      <td>Seandon</td>
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
      <td>Free-Mangina</td>
      <td>AB</td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Ari</td>
      <td>RB</td>
      <td>18</td>
      <td>David Johnson</td>
      <td>Free-Mangina</td>
      <td>AB</td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>177</th>
      <td>NO</td>
      <td>WR</td>
      <td>18</td>
      <td>Michael Thomas</td>
      <td>Douglast</td>
      <td>AB</td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>178</th>
      <td>Min</td>
      <td>QB</td>
      <td>22</td>
      <td>Kirk Cousins</td>
      <td>Douglast</td>
      <td>AB</td>
      <td>Jake</td>
    </tr>
    <tr>
      <th>179</th>
      <td>SF</td>
      <td>WR</td>
      <td>5</td>
      <td>Marquise Goodwin</td>
      <td>Douglast</td>
      <td>B</td>
      <td>Matt</td>
    </tr>
    <tr>
      <th>180</th>
      <td>Dal</td>
      <td>RB</td>
      <td>55</td>
      <td>Ezekiel Elliott</td>
      <td>Douglast</td>
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
      <td>Sea</td>
      <td>WR</td>
      <td>2</td>
      <td>David Moore</td>
      <td>Chi ShingT's Team</td>
      <td></td>
      <td>Chi Shing</td>
    </tr>
    <tr>
      <th>183</th>
      <td>SF</td>
      <td>QB</td>
      <td>5</td>
      <td>Jimmy Garoppolo</td>
      <td>Chi ShingT's Team</td>
      <td>B</td>
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
      <td>No bunny no fu fu</td>
      <td>B</td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>186</th>
      <td>NYG</td>
      <td>TE</td>
      <td>8</td>
      <td>Evan Engram</td>
      <td>No bunny no fu fu</td>
      <td>B</td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>187</th>
      <td>Chi</td>
      <td>QB</td>
      <td>0</td>
      <td>Mitchell Trubisky</td>
      <td>No bunny no fu fu</td>
      <td></td>
      <td>Andrew</td>
    </tr>
    <tr>
      <th>188</th>
      <td>Pit</td>
      <td>WR</td>
      <td>5</td>
      <td>JuJu Smith-Schuster</td>
      <td>No bunny no fu fu</td>
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
      <td>Douglast</td>
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
      <td>Seandon</td>
      <td></td>
      <td>Dai</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Daized and Confused</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>No bunny no fu fu</td>
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
      <td>Douglast</td>
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
      <td>No bunny no fu fu</td>
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
      <td>Free-Mangina</td>
      <td></td>
      <td>Ryan</td>
    </tr>
    <tr>
      <th>14</th>
      <td>LAC</td>
      <td>WR</td>
      <td>71</td>
      <td>Keenan Allen</td>
      <td>Daized and Confused</td>
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

    ["2 Gurley's 1 Cup", 'Bye Week', "Chi ShingT's Team", 'Cry me a Philip', 'Daized and Confused', 'Douglast', 'FirstRoundFlops', 'Free-Mangina', 'G', 'No bunny no fu fu', 'Seandon', 'Yeebola']
    


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
    68        Jordan Reed     Was   TE    10         2 Gurley's 1 Cup    Matt
    63        Matt Breida      SF   RB     8         2 Gurley's 1 Cup    Matt
    77       Philadelphia     Phi  DEF     5         2 Gurley's 1 Cup    Matt
    179  Marquise Goodwin      SF   WR     5      B  2 Gurley's 1 Cup    Matt
    92         Josh Rosen     Ari   QB     5         2 Gurley's 1 Cup    Matt
    104       Keke Coutee     Hou   WR     4         2 Gurley's 1 Cup    Matt
    71         Jared Cook     Oak   TE     3         2 Gurley's 1 Cup    Matt
    121       Aaron Jones      GB   RB     2         2 Gurley's 1 Cup    Matt
    64         Jared Goff     LAR   QB     2         2 Gurley's 1 Cup    Matt
    116    Baker Mayfield     Cle   QB     1         2 Gurley's 1 Cup    Matt
    ------------------------------------------------------------------------------------
                       name NFLTeam  Pos  cost keeper      team manager
    5        Saquon Barkley     NYG   RB    74         Bye Week    Doug
    37          Corey Davis     Ten   WR    35         Bye Week    Doug
    12            Jay Ajayi     Phi   RB    34         Bye Week    Doug
    52           Cam Newton     Car   QB    30         Bye Week    Doug
    174         Golden Tate     Det   WR    29     AB  Bye Week    Doug
    172     Devonta Freeman     Atl   RB    22     AB  Bye Week    Doug
    74       Devin Funchess     Car   WR    15         Bye Week    Doug
    59       Chester Rogers     Ind   WR     6         Bye Week    Doug
    171         Keelan Cole     Jax   WR     5      B  Bye Week    Doug
    173       Corey Clement     Phi   RB     5      B  Bye Week    Doug
    60   Stephen Gostkowski      NE    K     5         Bye Week    Doug
    99      Jerick McKinnon      SF   RB     3         Bye Week    Doug
    122       Cameron Brate      TB   TE     3         Bye Week    Doug
    66        Kyle Juszczyk      SF   RB     0         Bye Week    Doug
    73            Ito Smith     Atl   RB     0         Bye Week    Doug
    70             New York     NYJ  DEF     0         Bye Week    Doug
    ------------------------------------------------------------------------------------
                    name NFLTeam  Pos  cost keeper               team    manager
    1      Antonio Brown     Pit   WR    81         Chi ShingT's Team  Chi Shing
    25         Joe Mixon     Cin   RB    51         Chi ShingT's Team  Chi Shing
    38       Mark Ingram      NO   RB    39         Chi ShingT's Team  Chi Shing
    16      Robert Woods     LAR   WR    23         Chi ShingT's Team  Chi Shing
    30     Tevin Coleman     Atl   RB    22         Chi ShingT's Team  Chi Shing
    47      Jordy Nelson     Oak   WR    19         Chi ShingT's Team  Chi Shing
    181   Chris Thompson     Was   RB     5      B  Chi ShingT's Team  Chi Shing
    183  Jimmy Garoppolo      SF   QB     5      B  Chi ShingT's Team  Chi Shing
    184    Will Fuller V     Hou   WR     5      B  Chi ShingT's Team  Chi Shing
    102  Patrick Mahomes      KC   QB     3         Chi ShingT's Team  Chi Shing
    182      David Moore     Sea   WR     2         Chi ShingT's Team  Chi Shing
    135    Cameron Brate      TB   TE     1         Chi ShingT's Team  Chi Shing
    19   Phillip Dorsett      NE   WR     0         Chi ShingT's Team  Chi Shing
    82       Los Angeles     LAC  DEF     0         Chi ShingT's Team  Chi Shing
    127   Adam Vinatieri     Ind    K     0         Chi ShingT's Team  Chi Shing
    141        Cleveland     Cle  DEF     0         Chi ShingT's Team  Chi Shing
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper             team manager
    10        Julio Jones     Atl   WR    85         Cry me a Philip    Evan
    4         T.Y. Hilton     Ind   WR    51         Cry me a Philip    Evan
    162    Russell Wilson     Sea   QB    34      B  Cry me a Philip    Evan
    160     Sammy Watkins      KC   WR    32      B  Cry me a Philip    Evan
    8        Lamar Miller     Hou   RB    28         Cry me a Philip    Evan
    161      Kyle Rudolph     Min   TE    17     AB  Cry me a Philip    Evan
    72     Isaiah Crowell     NYJ   RB    14         Cry me a Philip    Evan
    56   Matthew Stafford     Det   QB    14         Cry me a Philip    Evan
    69    Giovani Bernard     Cin   RB     5         Cry me a Philip    Evan
    159     Tyler Lockett     Sea   WR     5      B  Cry me a Philip    Evan
    146    Raheem Mostert      SF   RB     4         Cry me a Philip    Evan
    113          Wil Lutz      NO    K     1         Cry me a Philip    Evan
    124       New England      NE  DEF     1         Cry me a Philip    Evan
    139        Eric Ebron     Ind   TE     1         Cry me a Philip    Evan
    132     C.J. Anderson     Car   RB     1         Cry me a Philip    Evan
    144        John Brown     Bal   WR     1         Cry me a Philip    Evan
    ------------------------------------------------------------------------------------
                      name NFLTeam  Pos  cost keeper                 team manager
    14        Keenan Allen     LAC   WR    71         Daized and Confused    Sean
    3    Leonard Fournette     Jax   RB    60         Daized and Confused    Sean
    20       Royce Freeman     Den   RB    36         Daized and Confused    Sean
    154         Mike Evans      TB   WR    30     AB  Daized and Confused    Sean
    62      Delanie Walker     Ten   TE    23         Daized and Confused    Sean
    39   Wendell Smallwood     Phi   RB    21         Daized and Confused    Sean
    153       Alvin Kamara      NO   RB     9      B  Daized and Confused    Sean
    75       George Kittle      SF   TE     6         Daized and Confused    Sean
    86    Duke Johnson Jr.     Cle   RB     5         Daized and Confused    Sean
    152  Allen Robinson II     Chi   WR     5      B  Daized and Confused    Sean
    155        Andrew Luck     Ind   QB     5      B  Daized and Confused    Sean
    142     Dallas Goedert     Phi   TE     4         Daized and Confused    Sean
    110       Chris Godwin      TB   WR     3         Daized and Confused    Sean
    130     Donte Moncrief     Jax   WR     1         Daized and Confused    Sean
    137     Jameis Winston      TB   QB     1         Daized and Confused    Sean
    98        Indianapolis     Ind  DEF     0         Daized and Confused    Sean
    ------------------------------------------------------------------------------------
                    name NFLTeam  Pos  cost keeper      team manager
    9         A.J. Green     Cin   WR    67         Douglast    Jake
    180  Ezekiel Elliott     Dal   RB    55      B  Douglast    Jake
    28         Zach Ertz     Phi   TE    49         Douglast    Jake
    23     Brandin Cooks     LAR   WR    39         Douglast    Jake
    178     Kirk Cousins     Min   QB    22     AB  Douglast    Jake
    177   Michael Thomas      NO   WR    18     AB  Douglast    Jake
    35   Adrian Peterson     Was   RB    12         Douglast    Jake
    84      Bilal Powell     NYJ   RB     7         Douglast    Jake
    0       Dak Prescott     Dal   QB     6         Douglast    Jake
    80    Taylor Gabriel     Chi   WR     5         Douglast    Jake
    94     Justin Tucker     Bal    K     4         Douglast    Jake
    129  Willie Snead IV     Bal   WR     3         Douglast    Jake
    24    DeSean Jackson      TB   WR     2         Douglast    Jake
    120       Pittsburgh     Pit  DEF     1         Douglast    Jake
    58          New York     NYJ  DEF     0         Douglast    Jake
    108       Greg Olsen     Car   TE     0         Douglast    Jake
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper             team manager
    17        Kareem Hunt      KC   RB    79         FirstRoundFlops   Rajiv
    29      Jordan Howard     Chi   RB    64         FirstRoundFlops   Rajiv
    151      Amari Cooper     Oak   WR    43     AB  FirstRoundFlops   Rajiv
    45        Trey Burton     Chi   TE    31         FirstRoundFlops   Rajiv
    21          Tom Brady      NE   QB    28         FirstRoundFlops   Rajiv
    150      Doug Baldwin     Sea   WR    24     AB  FirstRoundFlops   Rajiv
    36   Michael Crabtree     Bal   WR    19         FirstRoundFlops   Rajiv
    149     Austin Hooper     Atl   TE     6         FirstRoundFlops   Rajiv
    96         Jack Doyle     Ind   TE     4         FirstRoundFlops   Rajiv
    147   Latavius Murray     Min   RB     1         FirstRoundFlops   Rajiv
    117     Mike Williams     LAC   WR     1         FirstRoundFlops   Rajiv
    134   Harrison Butker      KC    K     1         FirstRoundFlops   Rajiv
    148   Tyrell Williams     LAC   WR     1         FirstRoundFlops   Rajiv
    140     Blake Bortles     Jax   QB     1         FirstRoundFlops   Rajiv
    145   Benjamin Watson      NO   TE     1         FirstRoundFlops   Rajiv
    126           Houston     Hou  DEF     0         FirstRoundFlops   Rajiv
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper          team manager
    13        Tyreek Hill      KC   WR    60         Free-Mangina    Ryan
    26       Travis Kelce      KC   TE    52         Free-Mangina    Ryan
    32      Aaron Rodgers      GB   QB    51         Free-Mangina    Ryan
    6        LeSean McCoy     Buf   RB    29         Free-Mangina    Ryan
    50         Dion Lewis     Ten   RB    29         Free-Mangina    Ryan
    176     David Johnson     Ari   RB    18     AB  Free-Mangina    Ryan
    53     Marshawn Lynch     Oak   RB    16         Free-Mangina    Ryan
    18    Kelvin Benjamin     Buf   WR    15         Free-Mangina    Ryan
    34       Jacksonville     Jax  DEF     8         Free-Mangina    Ryan
    175    Deshaun Watson     Hou   QB     6     AB  Free-Mangina    Ryan
    83     DeVante Parker     Mia   WR     5         Free-Mangina    Ryan
    93   Sterling Shepard     NYG   WR     4         Free-Mangina    Ryan
    136        Dan Bailey     Min    K     3         Free-Mangina    Ryan
    119        Nick Chubb     Cle   RB     3         Free-Mangina    Ryan
    128     Calvin Ridley     Atl   WR     1         Free-Mangina    Ryan
    107     San Francisco      SF  DEF     0         Free-Mangina    Ryan
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper team manager
    44   Demaryius Thomas     Den   WR    38           G   Jiwei
    65       Chris Carson     Sea   RB    25           G   Jiwei
    40        Cooper Kupp     LAR   WR    24           G   Jiwei
    51        Carlos Hyde     Cle   RB    22           G   Jiwei
    55        O.J. Howard      TB   TE    14           G   Jiwei
    191  Marvin Jones Jr.     Det   WR    12      B    G   Jiwei
    190      Carson Wentz     Phi   QB    10      B    G   Jiwei
    189      Jordy Nelson     Oak   WR     7           G   Jiwei
    97          Baltimore     Bal  DEF     5           G   Jiwei
    109         Matt Ryan     Atl   QB     3           G   Jiwei
    105      Kenny Stills     Mia   WR     3           G   Jiwei
    48     Vance McDonald     Pit   TE     0           G   Jiwei
    54   Ka'imi Fairbairn     Hou    K     0           G   Jiwei
    61         Mike Davis     Sea   RB     0           G   Jiwei
    89        Ryan Succop     Ten    K     0           G   Jiwei
    85        Alfred Blue     Hou   RB     0           G   Jiwei
    ------------------------------------------------------------------------------------
                        name NFLTeam  Pos  cost keeper               team manager
    11      Larry Fitzgerald     Ari   WR    41         No bunny no fu fu  Andrew
    43        Julian Edelman      NE   WR    19         No bunny no fu fu  Andrew
    7          Philip Rivers     LAC   QB    11         No bunny no fu fu  Andrew
    88       Kerryon Johnson     Det   RB    10         No bunny no fu fu  Andrew
    186          Evan Engram     NYG   TE     8      B  No bunny no fu fu  Andrew
    185          Tarik Cohen     Chi   RB     8      B  No bunny no fu fu  Andrew
    115       Kenny Golladay     Det   WR     5         No bunny no fu fu  Andrew
    100         James Conner     Pit   RB     5         No bunny no fu fu  Andrew
    188  JuJu Smith-Schuster     Pit   WR     5      B  No bunny no fu fu  Andrew
    91               Chicago     Chi  DEF     3         No bunny no fu fu  Andrew
    118         Jake Elliott     Phi    K     2         No bunny no fu fu  Andrew
    187    Mitchell Trubisky     Chi   QB     0         No bunny no fu fu  Andrew
    103     Antonio Callaway     Cle   WR     0         No bunny no fu fu  Andrew
    79         Peyton Barber      TB   RB     0         No bunny no fu fu  Andrew
    67          Nyheim Hines     Ind   RB     0         No bunny no fu fu  Andrew
    ------------------------------------------------------------------------------------
                      name NFLTeam  Pos  cost keeper     team manager
    2       Rob Gronkowski      NE   TE    64         Seandon     Dai
    170    DeAndre Hopkins     Hou   WR    56      B  Seandon     Dai
    169  Melvin Gordon III     LAC   RB    39     AB  Seandon     Dai
    168      Davante Adams      GB   WR    27     AB  Seandon     Dai
    49          Drew Brees      NO   QB    25         Seandon     Dai
    167       Kenyan Drake     Mia   RB    17      B  Seandon     Dai
    87          Dez Bryant     Dal   WR     8         Seandon     Dai
    125      Rashaad Penny     Sea   RB     5         Seandon     Dai
    133      Rashaad Penny     Sea   RB     5         Seandon     Dai
    114       Charles Clay     Buf   TE     1         Seandon     Dai
    46             Buffalo     Buf  DEF     0         Seandon     Dai
    42     Ronald Jones II      TB   RB     0         Seandon     Dai
    22       Peyton Barber      TB   RB     0         Seandon     Dai
    81      Jameis Winston      TB   QB     0         Seandon     Dai
    123     Javorius Allen     Bal   RB     0         Seandon     Dai
    90          D.J. Moore     Car   WR     0         Seandon     Dai
    ------------------------------------------------------------------------------------
                        name NFLTeam  Pos  cost keeper     team manager
    27   Christian McCaffrey     Car   RB    67         Yeebola     Ron
    166         Le'Veon Bell     Pit   RB    66     AB  Yeebola     Ron
    41          Alex Collins     Bal   RB    56         Yeebola     Ron
    164    Odell Beckham Jr.     NYG   WR    29     AB  Yeebola     Ron
    163         Stefon Diggs     Min   WR    24     AB  Yeebola     Ron
    31          Jimmy Graham      GB   TE    23         Yeebola     Ron
    95           Sony Michel      NE   RB     7         Yeebola     Ron
    165          Josh Gordon      NE   WR     6      B  Yeebola     Ron
    112          Marlon Mack     Ind   RB     6         Yeebola     Ron
    106        Greg Zuerlein     LAR    K     6         Yeebola     Ron
    78        Dede Westbrook     Jax   WR     5         Yeebola     Ron
    111    LeGarrette Blount     Det   RB     2         Yeebola     Ron
    101         Indianapolis     Ind  DEF     0         Yeebola     Ron
    131            Ito Smith     Atl   RB     0         Yeebola     Ron
    138           Tyler Boyd     Cin   WR     0         Yeebola     Ron
    76    Ben Roethlisberger     Pit   QB     0         Yeebola     Ron
    143          Los Angeles     LAC  DEF     0         Yeebola     Ron
    ------------------------------------------------------------------------------------
    


```python
# Save data as CSV
roster_df.to_csv("current_roster.csv")
```
