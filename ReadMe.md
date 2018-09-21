

```python
import pandas as pd
from splinter import Browser
from bs4 import BeautifulSoup
import requests
import numpy as np
import time
import json
```


```python
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
draft_df = pd.DataFrame(draft_pick_list)
draft_df['keeper'] = ""
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dal</td>
      <td>QB</td>
      <td>6</td>
      <td>Dak Prescott</td>
      <td>Naggers w/ Attitude</td>
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
    </tr>
    <tr>
      <th>2</th>
      <td>NE</td>
      <td>TE</td>
      <td>64</td>
      <td>Rob Gronkowski</td>
      <td>crushiNG RON</td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Kamara Sutra</td>
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
    </tr>
    <tr>
      <th>5</th>
      <td>NYG</td>
      <td>RB</td>
      <td>74</td>
      <td>Saquon Barkley</td>
      <td>Bye Week</td>
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
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>Thx Ron and Le'Veon</td>
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
    </tr>
    <tr>
      <th>9</th>
      <td>Cin</td>
      <td>WR</td>
      <td>67</td>
      <td>A.J. Green</td>
      <td>Naggers w/ Attitude</td>
      <td></td>
    </tr>
    <tr>
      <th>10</th>
      <td>Atl</td>
      <td>WR</td>
      <td>85</td>
      <td>Julio Jones</td>
      <td>Cry me a Philip</td>
      <td></td>
    </tr>
    <tr>
      <th>11</th>
      <td>Ari</td>
      <td>WR</td>
      <td>41</td>
      <td>Larry Fitzgerald</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
    </tr>
    <tr>
      <th>12</th>
      <td>Phi</td>
      <td>RB</td>
      <td>34</td>
      <td>Jay Ajayi</td>
      <td>Bye Week</td>
      <td></td>
    </tr>
    <tr>
      <th>13</th>
      <td>KC</td>
      <td>WR</td>
      <td>60</td>
      <td>Tyreek Hill</td>
      <td>Nags</td>
      <td></td>
    </tr>
    <tr>
      <th>14</th>
      <td>LAC</td>
      <td>WR</td>
      <td>71</td>
      <td>Keenan Allen</td>
      <td>Kamara Sutra</td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
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
for i, d in draft_df.iterrows():
    for index, keeper in keeper_df.iterrows():
        if d['name'] == keeper['player']:
            draft_df.at[i,'keeper'] = keeper['duration']
            
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dal</td>
      <td>QB</td>
      <td>6</td>
      <td>Dak Prescott</td>
      <td>Naggers w/ Attitude</td>
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
    </tr>
    <tr>
      <th>2</th>
      <td>NE</td>
      <td>TE</td>
      <td>64</td>
      <td>Rob Gronkowski</td>
      <td>crushiNG RON</td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Kamara Sutra</td>
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
    </tr>
    <tr>
      <th>5</th>
      <td>NYG</td>
      <td>RB</td>
      <td>74</td>
      <td>Saquon Barkley</td>
      <td>Bye Week</td>
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
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>Thx Ron and Le'Veon</td>
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
    </tr>
    <tr>
      <th>9</th>
      <td>Cin</td>
      <td>WR</td>
      <td>67</td>
      <td>A.J. Green</td>
      <td>Naggers w/ Attitude</td>
      <td></td>
    </tr>
    <tr>
      <th>10</th>
      <td>Atl</td>
      <td>WR</td>
      <td>85</td>
      <td>Julio Jones</td>
      <td>Cry me a Philip</td>
      <td></td>
    </tr>
    <tr>
      <th>11</th>
      <td>Ari</td>
      <td>WR</td>
      <td>41</td>
      <td>Larry Fitzgerald</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
    </tr>
    <tr>
      <th>12</th>
      <td>Phi</td>
      <td>RB</td>
      <td>34</td>
      <td>Jay Ajayi</td>
      <td>Bye Week</td>
      <td></td>
    </tr>
    <tr>
      <th>13</th>
      <td>KC</td>
      <td>WR</td>
      <td>60</td>
      <td>Tyreek Hill</td>
      <td>Nags</td>
      <td></td>
    </tr>
    <tr>
      <th>14</th>
      <td>LAC</td>
      <td>WR</td>
      <td>71</td>
      <td>Keenan Allen</td>
      <td>Kamara Sutra</td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
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
            try:
                transaction_info = transaction[1].text.splitlines()
                team_info = transaction[2].text.splitlines()
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
                transaction_dict = {
                    "add": transaction_info[1].strip(),
                    "nfl_team": transaction_info[2].strip().split(" - ")[0],
                    "position": transaction_info[2].strip().split(" - ")[1],
                    "price":price,
                    "drop":transaction_info[4].strip(),
                    "ffteam":team_info[3].strip()
                }
                transaction_list.append(transaction_dict)
            except:
                pass
    except:
        pass
```


```python
transaction_df = pd.DataFrame(transaction_list)
transaction_df = transaction_df.reindex(index=transaction_df.index[::-1]).reset_index(drop=True)
transaction_df = transaction_df.drop([48, 49]).reset_index(drop=True)
transaction_df.head(15)
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>James White</td>
      <td>Pittsburgh</td>
      <td>Naggers w/ Attitude</td>
      <td>NE</td>
      <td>RB</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mason Crosby</td>
      <td>Baker Mayfield</td>
      <td>2 Gurley's 1 Cup</td>
      <td>GB</td>
      <td>K</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>John Ross</td>
      <td>Dez Bryant</td>
      <td>crushiNG RON</td>
      <td>Cin</td>
      <td>WR</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Austin Hooper</td>
      <td>Charles Clay</td>
      <td>crushiNG RON</td>
      <td>Atl</td>
      <td>TE</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Graham Gano</td>
      <td>Donte Moncrief</td>
      <td>Kamara Sutra</td>
      <td>Car</td>
      <td>K</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Allen Hurns</td>
      <td>LeGarrette Blount</td>
      <td>K-K-Dai</td>
      <td>Dal</td>
      <td>WR</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Quincy Enunwa</td>
      <td>Denver</td>
      <td>K-K-Dai</td>
      <td>NYJ</td>
      <td>WR</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Jaylen Samuels</td>
      <td>Austin Hooper</td>
      <td>crushiNG RON</td>
      <td>Pit</td>
      <td>RB,TE</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Geronimo Allison</td>
      <td>Dede Westbrook</td>
      <td>K-K-Dai</td>
      <td>GB</td>
      <td>WR</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Dede Westbrook</td>
      <td>Matt Prater</td>
      <td>G</td>
      <td>Jax</td>
      <td>WR</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Brandon McManus</td>
      <td>Jaylen Samuels</td>
      <td>crushiNG RON</td>
      <td>Den</td>
      <td>K</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Phillip Lindsay</td>
      <td>Matt Breida</td>
      <td>2 Gurley's 1 Cup</td>
      <td>Den</td>
      <td>RB</td>
      <td>15</td>
    </tr>
    <tr>
      <th>12</th>
      <td>T.J. Yeldon</td>
      <td>Delanie Walker</td>
      <td>Kamara Sutra</td>
      <td>Jax</td>
      <td>RB</td>
      <td>11</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Austin Ekeler</td>
      <td>Duke Johnson Jr.</td>
      <td>Kamara Sutra</td>
      <td>LAC</td>
      <td>RB</td>
      <td>6</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Ted Ginn Jr.</td>
      <td>Jameis Winston</td>
      <td>Kamara Sutra</td>
      <td>NO</td>
      <td>WR</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
for x, t in transaction_df.iterrows():
    for i, d in draft_df.iterrows():
        if d['name'] == t['drop']:
            draft_df.at[i, 'name'] = t['add']
            draft_df.at[i, 'NFLTeam'] = t['nfl_team']
            draft_df.at[i, 'Pos'] = t['position']
            draft_df.at[i, 'cost'] = t['price']
            draft_df.at[i, 'keeper'] = ""
roster_df = draft_df
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dal</td>
      <td>QB</td>
      <td>6</td>
      <td>Dak Prescott</td>
      <td>Naggers w/ Attitude</td>
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
    </tr>
    <tr>
      <th>2</th>
      <td>NE</td>
      <td>TE</td>
      <td>64</td>
      <td>Rob Gronkowski</td>
      <td>crushiNG RON</td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jax</td>
      <td>RB</td>
      <td>60</td>
      <td>Leonard Fournette</td>
      <td>Kamara Sutra</td>
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
    </tr>
    <tr>
      <th>5</th>
      <td>NYG</td>
      <td>RB</td>
      <td>74</td>
      <td>Saquon Barkley</td>
      <td>Bye Week</td>
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
    </tr>
    <tr>
      <th>7</th>
      <td>LAC</td>
      <td>QB</td>
      <td>11</td>
      <td>Philip Rivers</td>
      <td>Thx Ron and Le'Veon</td>
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
    </tr>
    <tr>
      <th>9</th>
      <td>Cin</td>
      <td>WR</td>
      <td>67</td>
      <td>A.J. Green</td>
      <td>Naggers w/ Attitude</td>
      <td></td>
    </tr>
    <tr>
      <th>10</th>
      <td>Atl</td>
      <td>WR</td>
      <td>85</td>
      <td>Julio Jones</td>
      <td>Cry me a Philip</td>
      <td></td>
    </tr>
    <tr>
      <th>11</th>
      <td>Ari</td>
      <td>WR</td>
      <td>41</td>
      <td>Larry Fitzgerald</td>
      <td>Thx Ron and Le'Veon</td>
      <td></td>
    </tr>
    <tr>
      <th>12</th>
      <td>Phi</td>
      <td>RB</td>
      <td>34</td>
      <td>Jay Ajayi</td>
      <td>Bye Week</td>
      <td></td>
    </tr>
    <tr>
      <th>13</th>
      <td>KC</td>
      <td>WR</td>
      <td>60</td>
      <td>Tyreek Hill</td>
      <td>Nags</td>
      <td></td>
    </tr>
    <tr>
      <th>14</th>
      <td>LAC</td>
      <td>WR</td>
      <td>71</td>
      <td>Keenan Allen</td>
      <td>Kamara Sutra</td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
ff_team_names = roster_df.team.unique()
print(ff_team_names)
```

    ['Naggers w/ Attitude' "Chi ShingT's Team" 'crushiNG RON' 'Kamara Sutra'
     'Cry me a Philip' 'Bye Week' 'Nags' "Thx Ron and Le'Veon"
     "2 Gurley's 1 Cup" 'FirstRoundFlops' 'K-K-Dai' 'G']
    


```python
roster_df = roster_df[['name', 'NFLTeam', 'Pos', 'cost', 'keeper', 'team']]
for ff in ff_team_names:
    print(roster_df.loc[roster_df['team'] == ff])
    print("------------------------------------------------------------------------------------")
```

                     name NFLTeam  Pos  cost keeper                 team
    0        Dak Prescott     Dal   QB     6         Naggers w/ Attitude
    9          A.J. Green     Cin   WR    67         Naggers w/ Attitude
    23      Brandin Cooks     LAR   WR    39         Naggers w/ Attitude
    24     DeSean Jackson      TB   WR     2         Naggers w/ Attitude
    28          Zach Ertz     Phi   TE    49         Naggers w/ Attitude
    58        Los Angeles     LAR  DEF    11         Naggers w/ Attitude
    80      Rashaad Penny     Sea   RB    13         Naggers w/ Attitude
    84       Bilal Powell     NYJ   RB     7         Naggers w/ Attitude
    94      Justin Tucker     Bal    K     4         Naggers w/ Attitude
    108        Greg Olsen     Car   TE     0         Naggers w/ Attitude
    120       James White      NE   RB     2         Naggers w/ Attitude
    129      Randall Cobb      GB   WR     1         Naggers w/ Attitude
    177    Michael Thomas      NO   WR    18     AB  Naggers w/ Attitude
    178      Kirk Cousins     Min   QB    22     AB  Naggers w/ Attitude
    179  Marquise Goodwin      SF   WR     5      B  Naggers w/ Attitude
    180   Ezekiel Elliott     Dal   RB    55      B  Naggers w/ Attitude
    ------------------------------------------------------------------------------------
                    name NFLTeam  Pos  cost keeper               team
    1      Antonio Brown     Pit   WR    81         Chi ShingT's Team
    16      Robert Woods     LAR   WR    23         Chi ShingT's Team
    19   Phillip Dorsett      NE   WR     0         Chi ShingT's Team
    25         Joe Mixon     Cin   RB    51         Chi ShingT's Team
    30     Tevin Coleman     Atl   RB    22         Chi ShingT's Team
    38       Mark Ingram      NO   RB    39         Chi ShingT's Team
    47       Will Dissly     Sea   TE     0         Chi ShingT's Team
    82    Darren Sproles     Phi   RB     3         Chi ShingT's Team
    102  Patrick Mahomes      KC   QB     3         Chi ShingT's Team
    127    Chris Boswell     Pit    K     1         Chi ShingT's Team
    135       Frank Gore     Mia   RB     0         Chi ShingT's Team
    141        Cleveland     Cle  DEF     0         Chi ShingT's Team
    181   Chris Thompson     Was   RB     5      B  Chi ShingT's Team
    182    Peyton Barber      TB   RB    10      B  Chi ShingT's Team
    183  Jimmy Garoppolo      SF   QB     5      B  Chi ShingT's Team
    184    Will Fuller V     Hou   WR     5      B  Chi ShingT's Team
    ------------------------------------------------------------------------------------
                      name NFLTeam  Pos  cost keeper          team
    2       Rob Gronkowski      NE   TE    64         crushiNG RON
    22       Alfred Morris      SF   RB     8         crushiNG RON
    42     Jamison Crowder     Was   WR    20         crushiNG RON
    46           Minnesota     Min  DEF     9         crushiNG RON
    49          Drew Brees      NO   QB    25         crushiNG RON
    81      Nelson Agholor     Phi   WR    13         crushiNG RON
    87         Mark Walton     Cin   RB     0         crushiNG RON
    90     Jamaal Williams      GB   RB     9         crushiNG RON
    114      Malcolm Brown     LAR   RB     0         crushiNG RON
    123     Javorius Allen     Bal   RB     0         crushiNG RON
    125     Jordan Wilkins     Ind   RB     1         crushiNG RON
    133        Marlon Mack     Ind   RB     1         crushiNG RON
    167       Kenyan Drake     Mia   RB    17      B  crushiNG RON
    168      Davante Adams      GB   WR    27     AB  crushiNG RON
    169  Melvin Gordon III     LAC   RB    39     AB  crushiNG RON
    170    DeAndre Hopkins     Hou   WR    56      B  crushiNG RON
    ------------------------------------------------------------------------------------
                      name NFLTeam  Pos  cost keeper          team
    3    Leonard Fournette     Jax   RB    60         Kamara Sutra
    14        Keenan Allen     LAC   WR    71         Kamara Sutra
    20       Royce Freeman     Den   RB    36         Kamara Sutra
    39         Chris Hogan      NE   WR    39         Kamara Sutra
    62         T.J. Yeldon     Jax   RB    11         Kamara Sutra
    75       George Kittle      SF   TE     6         Kamara Sutra
    86       Austin Ekeler     LAC   RB     6         Kamara Sutra
    98              Dallas     Dal  DEF     1         Kamara Sutra
    110       Chris Godwin      TB   WR     3         Kamara Sutra
    130         Josh Lambo     Jax    K     0         Kamara Sutra
    137   Ryan Fitzpatrick      TB   QB     6         Kamara Sutra
    142        Jesse James     Pit   TE     2         Kamara Sutra
    152  Allen Robinson II     Chi   WR     5      B  Kamara Sutra
    153       Alvin Kamara      NO   RB     9      B  Kamara Sutra
    154         Mike Evans      TB   WR    30     AB  Kamara Sutra
    155        Andrew Luck     Ind   QB     5      B  Kamara Sutra
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper             team
    4         T.Y. Hilton     Ind   WR    51         Cry me a Philip
    8        Lamar Miller     Hou   RB    28         Cry me a Philip
    10        Julio Jones     Atl   WR    85         Cry me a Philip
    56   Matthew Stafford     Det   QB    14         Cry me a Philip
    69     Robby Anderson     NYJ   WR    14         Cry me a Philip
    72     Isaiah Crowell     NYJ   RB    14         Cry me a Philip
    113          Wil Lutz      NO    K     1         Cry me a Philip
    124       New England      NE  DEF     1         Cry me a Philip
    132     C.J. Anderson     Car   RB     1         Cry me a Philip
    139        Eric Ebron     Ind   TE     1         Cry me a Philip
    144        John Brown     Bal   WR     1         Cry me a Philip
    146           Arizona     Ari  DEF     1         Cry me a Philip
    159     Tyler Lockett     Sea   WR     5      B  Cry me a Philip
    160     Sammy Watkins      KC   WR    32      B  Cry me a Philip
    161      Kyle Rudolph     Min   TE    17     AB  Cry me a Philip
    162    Russell Wilson     Sea   QB    34      B  Cry me a Philip
    ------------------------------------------------------------------------------------
                       name NFLTeam  Pos  cost keeper      team
    5        Saquon Barkley     NYG   RB    74         Bye Week
    12            Jay Ajayi     Phi   RB    34         Bye Week
    37          Corey Davis     Ten   WR    35         Bye Week
    52           Cam Newton     Car   QB    30         Bye Week
    59        Pierre Garcon      SF   WR    15         Bye Week
    60   Stephen Gostkowski      NE    K     5         Bye Week
    66       Michael Gallup     Dal   WR    14         Bye Week
    70          Los Angeles     LAC  DEF     4         Bye Week
    73         Tyler Eifert     Cin   TE     5         Bye Week
    74       Devin Funchess     Car   WR    15         Bye Week
    99      Jerick McKinnon      SF   RB     3         Bye Week
    122         Doug Martin     Oak   RB     1         Bye Week
    171         Keelan Cole     Jax   WR     5      B  Bye Week
    172     Devonta Freeman     Atl   RB    22     AB  Bye Week
    173       Corey Clement     Phi   RB     5      B  Bye Week
    174         Golden Tate     Det   WR    29     AB  Bye Week
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper  team
    6        LeSean McCoy     Buf   RB    29         Nags
    13        Tyreek Hill      KC   WR    60         Nags
    18    Kelvin Benjamin     Buf   WR    15         Nags
    26       Travis Kelce      KC   TE    52         Nags
    32      Aaron Rodgers      GB   QB    51         Nags
    34       Jacksonville     Jax  DEF     8         Nags
    50         Dion Lewis     Ten   RB    29         Nags
    53     Marshawn Lynch     Oak   RB    16         Nags
    83     DeVante Parker     Mia   WR     5         Nags
    93   Sterling Shepard     NYG   WR     4         Nags
    107  Rishard Matthews     Ten   WR     1         Nags
    119        Nick Chubb     Cle   RB     3         Nags
    128     Calvin Ridley     Atl   WR     1         Nags
    136        Dan Bailey     Min    K     3         Nags
    175    Deshaun Watson     Hou   QB     6     AB  Nags
    176     David Johnson     Ari   RB    18     AB  Nags
    ------------------------------------------------------------------------------------
                        name NFLTeam  Pos  cost keeper                 team
    7          Philip Rivers     LAC   QB    11         Thx Ron and Le'Veon
    11      Larry Fitzgerald     Ari   WR    41         Thx Ron and Le'Veon
    43        Julian Edelman      NE   WR    19         Thx Ron and Le'Veon
    67       Ronald Jones II      TB   RB    23         Thx Ron and Le'Veon
    79        Anthony Miller     Chi   WR     7         Thx Ron and Le'Veon
    88       Kerryon Johnson     Det   RB    10         Thx Ron and Le'Veon
    91               Chicago     Chi  DEF     3         Thx Ron and Le'Veon
    95           Sony Michel      NE   RB     7         Thx Ron and Le'Veon
    100         James Conner     Pit   RB     5         Thx Ron and Le'Veon
    103     Antonio Callaway     Cle   WR     0         Thx Ron and Le'Veon
    115       Kenny Golladay     Det   WR     5         Thx Ron and Le'Veon
    118         Jake Elliott     Phi    K     2         Thx Ron and Le'Veon
    185          Tarik Cohen     Chi   RB     8      B  Thx Ron and Le'Veon
    186          Evan Engram     NYG   TE     8      B  Thx Ron and Le'Veon
    187        Derrick Henry     Ten   RB    19      B  Thx Ron and Le'Veon
    188  JuJu Smith-Schuster     Pit   WR     5      B  Thx Ron and Le'Veon
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper              team
    15     Alshon Jeffery     Phi   WR    40         2 Gurley's 1 Cup
    33      Jarvis Landry     Cle   WR    38         2 Gurley's 1 Cup
    35    Adrian Peterson     Was   RB    12         2 Gurley's 1 Cup
    57   Emmanuel Sanders     Den   WR    18         2 Gurley's 1 Cup
    63    Phillip Lindsay     Den   RB    15         2 Gurley's 1 Cup
    64         Jared Goff     LAR   QB     2         2 Gurley's 1 Cup
    68        Jordan Reed     Was   TE    10         2 Gurley's 1 Cup
    71         Jared Cook     Oak   TE     3         2 Gurley's 1 Cup
    77       Philadelphia     Phi  DEF     5         2 Gurley's 1 Cup
    92         Josh Rosen     Ari   QB     5         2 Gurley's 1 Cup
    104       Sam Darnold     NYJ   QB     1         2 Gurley's 1 Cup
    116      Mason Crosby      GB    K     0         2 Gurley's 1 Cup
    121       Aaron Jones      GB   RB     2         2 Gurley's 1 Cup
    156      Adam Thielen     Min   WR    28      B  2 Gurley's 1 Cup
    157       Dalvin Cook     Min   RB    64      B  2 Gurley's 1 Cup
    158    Todd Gurley II     LAR   RB    40     AB  2 Gurley's 1 Cup
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper             team
    17        Kareem Hunt      KC   RB    79         FirstRoundFlops
    21          Tom Brady      NE   QB    28         FirstRoundFlops
    29      Jordan Howard     Chi   RB    64         FirstRoundFlops
    36   Michael Crabtree     Bal   WR    19         FirstRoundFlops
    45        Trey Burton     Chi   TE    31         FirstRoundFlops
    96         Jack Doyle     Ind   TE     4         FirstRoundFlops
    117     Mike Williams     LAC   WR     1         FirstRoundFlops
    126           Atlanta     Atl  DEF     1         FirstRoundFlops
    134   Harrison Butker      KC    K     1         FirstRoundFlops
    140     Blake Bortles     Jax   QB     1         FirstRoundFlops
    145   Benjamin Watson      NO   TE     1         FirstRoundFlops
    147   Latavius Murray     Min   RB     1         FirstRoundFlops
    148   Tyrell Williams     LAC   WR     1         FirstRoundFlops
    149      Robbie Gould      SF    K     1         FirstRoundFlops
    150      Doug Baldwin     Sea   WR    24     AB  FirstRoundFlops
    151      Amari Cooper     Oak   WR    43     AB  FirstRoundFlops
    ------------------------------------------------------------------------------------
                        name NFLTeam  Pos  cost keeper     team
    27   Christian McCaffrey     Car   RB    67         K-K-Dai
    31          Jimmy Graham      GB   TE    23         K-K-Dai
    41          Alex Collins     Bal   RB    56         K-K-Dai
    76    Ben Roethlisberger     Pit   QB     0         K-K-Dai
    78      Geronimo Allison      GB   WR     0         K-K-Dai
    101        Quincy Enunwa     NYJ   WR     0         K-K-Dai
    106          Matt Bryant     Atl    K     2         K-K-Dai
    111          Matt Breida      SF   RB    11         K-K-Dai
    112      Giovani Bernard     Cin   RB     6         K-K-Dai
    131           Alex Smith     Was   QB     1         K-K-Dai
    138           Tyler Boyd     Cin   WR     0         K-K-Dai
    143              Houston     Hou  DEF     0         K-K-Dai
    163         Stefon Diggs     Min   WR    24     AB  K-K-Dai
    164    Odell Beckham Jr.     NYG   WR    29     AB  K-K-Dai
    165          Josh Gordon      NE   WR     6      B  K-K-Dai
    166         Le'Veon Bell     Pit   RB    66     AB  K-K-Dai
    ------------------------------------------------------------------------------------
                     name NFLTeam  Pos  cost keeper team
    40        Cooper Kupp     LAR   WR    24           G
    44   Demaryius Thomas     Den   WR    38           G
    48       Rex Burkhead      NE   RB    29           G
    51        Carlos Hyde     Cle   RB    22           G
    54     Dede Westbrook     Jax   WR     0           G
    55        O.J. Howard      TB   TE    14           G
    61        David Njoku     Cle   TE     9           G
    65       Chris Carson     Sea   RB    25           G
    85       Spencer Ware      KC   RB     1           G
    89      Greg Zuerlein     LAR    K     5           G
    97          Baltimore     Bal  DEF     5           G
    105      Kenny Stills     Mia   WR     3           G
    109         Matt Ryan     Atl   QB     3           G
    189      Tyrod Taylor     Cle   QB     1           G
    190      Carson Wentz     Phi   QB    10      B    G
    191  Marvin Jones Jr.     Det   WR    12      B    G
    ------------------------------------------------------------------------------------
    
