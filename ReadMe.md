

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
for ff in ff_team_names:
    print(roster_df.loc[roster_df['team'] == ff])
    print("------------------------------------------------------------------------------------")
```

        NFLTeam  Pos  cost              name                 team keeper
    0       Dal   QB     6      Dak Prescott  Naggers w/ Attitude       
    9       Cin   WR    67        A.J. Green  Naggers w/ Attitude       
    23      LAR   WR    39     Brandin Cooks  Naggers w/ Attitude       
    24       TB   WR     2    DeSean Jackson  Naggers w/ Attitude       
    28      Phi   TE    49         Zach Ertz  Naggers w/ Attitude       
    58      LAR  DEF    11       Los Angeles  Naggers w/ Attitude       
    80      Sea   RB    13     Rashaad Penny  Naggers w/ Attitude       
    84      NYJ   RB     7      Bilal Powell  Naggers w/ Attitude       
    94      Bal    K     4     Justin Tucker  Naggers w/ Attitude       
    108     Car   TE     0        Greg Olsen  Naggers w/ Attitude       
    120      NE   RB     2       James White  Naggers w/ Attitude       
    129      GB   WR     1      Randall Cobb  Naggers w/ Attitude       
    177      NO   WR    18    Michael Thomas  Naggers w/ Attitude     AB
    178     Min   QB    22      Kirk Cousins  Naggers w/ Attitude     AB
    179      SF   WR     5  Marquise Goodwin  Naggers w/ Attitude      B
    180     Dal   RB    55   Ezekiel Elliott  Naggers w/ Attitude      B
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost             name               team keeper
    1       Pit   WR    81    Antonio Brown  Chi ShingT's Team       
    16      LAR   WR    23     Robert Woods  Chi ShingT's Team       
    19       NE   WR     0  Phillip Dorsett  Chi ShingT's Team       
    25      Cin   RB    51        Joe Mixon  Chi ShingT's Team       
    30      Atl   RB    22    Tevin Coleman  Chi ShingT's Team       
    38       NO   RB    39      Mark Ingram  Chi ShingT's Team       
    47      Sea   TE     0      Will Dissly  Chi ShingT's Team       
    82      Phi   RB     3   Darren Sproles  Chi ShingT's Team       
    102      KC   QB     3  Patrick Mahomes  Chi ShingT's Team       
    127     Pit    K     1    Chris Boswell  Chi ShingT's Team       
    135     Mia   RB     0       Frank Gore  Chi ShingT's Team       
    141     Cle  DEF     0        Cleveland  Chi ShingT's Team       
    181     Was   RB     5   Chris Thompson  Chi ShingT's Team      B
    182      TB   RB    10    Peyton Barber  Chi ShingT's Team      B
    183      SF   QB     5  Jimmy Garoppolo  Chi ShingT's Team      B
    184     Hou   WR     5    Will Fuller V  Chi ShingT's Team      B
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost               name          team keeper
    2        NE   TE    64     Rob Gronkowski  crushiNG RON       
    22       SF   RB     8      Alfred Morris  crushiNG RON       
    42      Was   WR    20    Jamison Crowder  crushiNG RON       
    46      Min  DEF     9          Minnesota  crushiNG RON       
    49       NO   QB    25         Drew Brees  crushiNG RON       
    81      Phi   WR    13     Nelson Agholor  crushiNG RON       
    87      Cin   RB     0        Mark Walton  crushiNG RON       
    90       GB   RB     9    Jamaal Williams  crushiNG RON       
    114     LAR   RB     0      Malcolm Brown  crushiNG RON       
    123     Bal   RB     0     Javorius Allen  crushiNG RON       
    125     Ind   RB     1     Jordan Wilkins  crushiNG RON       
    133     Ind   RB     1        Marlon Mack  crushiNG RON       
    167     Mia   RB    17       Kenyan Drake  crushiNG RON      B
    168      GB   WR    27      Davante Adams  crushiNG RON     AB
    169     LAC   RB    39  Melvin Gordon III  crushiNG RON     AB
    170     Hou   WR    56    DeAndre Hopkins  crushiNG RON      B
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost               name          team keeper
    3       Jax   RB    60  Leonard Fournette  Kamara Sutra       
    14      LAC   WR    71       Keenan Allen  Kamara Sutra       
    20      Den   RB    36      Royce Freeman  Kamara Sutra       
    39       NE   WR    39        Chris Hogan  Kamara Sutra       
    62      Jax   RB    11        T.J. Yeldon  Kamara Sutra       
    75       SF   TE     6      George Kittle  Kamara Sutra       
    86      LAC   RB     6      Austin Ekeler  Kamara Sutra       
    98      Dal  DEF     1             Dallas  Kamara Sutra       
    110      TB   WR     3       Chris Godwin  Kamara Sutra       
    130     Jax    K     0         Josh Lambo  Kamara Sutra       
    137      TB   QB     6   Ryan Fitzpatrick  Kamara Sutra       
    142     Pit   TE     2        Jesse James  Kamara Sutra       
    152     Chi   WR     5  Allen Robinson II  Kamara Sutra      B
    153      NO   RB     9       Alvin Kamara  Kamara Sutra      B
    154      TB   WR    30         Mike Evans  Kamara Sutra     AB
    155     Ind   QB     5        Andrew Luck  Kamara Sutra      B
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost              name             team keeper
    4       Ind   WR    51       T.Y. Hilton  Cry me a Philip       
    8       Hou   RB    28      Lamar Miller  Cry me a Philip       
    10      Atl   WR    85       Julio Jones  Cry me a Philip       
    56      Det   QB    14  Matthew Stafford  Cry me a Philip       
    69      NYJ   WR    14    Robby Anderson  Cry me a Philip       
    72      NYJ   RB    14    Isaiah Crowell  Cry me a Philip       
    113      NO    K     1          Wil Lutz  Cry me a Philip       
    124      NE  DEF     1       New England  Cry me a Philip       
    132     Car   RB     1     C.J. Anderson  Cry me a Philip       
    139     Ind   TE     1        Eric Ebron  Cry me a Philip       
    144     Bal   WR     1        John Brown  Cry me a Philip       
    146     Ari  DEF     1           Arizona  Cry me a Philip       
    159     Sea   WR     5     Tyler Lockett  Cry me a Philip      B
    160      KC   WR    32     Sammy Watkins  Cry me a Philip      B
    161     Min   TE    17      Kyle Rudolph  Cry me a Philip     AB
    162     Sea   QB    34    Russell Wilson  Cry me a Philip      B
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost                name      team keeper
    5       NYG   RB    74      Saquon Barkley  Bye Week       
    12      Phi   RB    34           Jay Ajayi  Bye Week       
    37      Ten   WR    35         Corey Davis  Bye Week       
    52      Car   QB    30          Cam Newton  Bye Week       
    59       SF   WR    15       Pierre Garcon  Bye Week       
    60       NE    K     5  Stephen Gostkowski  Bye Week       
    66      Dal   WR    14      Michael Gallup  Bye Week       
    70      LAC  DEF     4         Los Angeles  Bye Week       
    73      Cin   TE     5        Tyler Eifert  Bye Week       
    74      Car   WR    15      Devin Funchess  Bye Week       
    99       SF   RB     3     Jerick McKinnon  Bye Week       
    122     Oak   RB     1         Doug Martin  Bye Week       
    171     Jax   WR     5         Keelan Cole  Bye Week      B
    172     Atl   RB    22     Devonta Freeman  Bye Week     AB
    173     Phi   RB     5       Corey Clement  Bye Week      B
    174     Det   WR    29         Golden Tate  Bye Week     AB
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost              name  team keeper
    6       Buf   RB    29      LeSean McCoy  Nags       
    13       KC   WR    60       Tyreek Hill  Nags       
    18      Buf   WR    15   Kelvin Benjamin  Nags       
    26       KC   TE    52      Travis Kelce  Nags       
    32       GB   QB    51     Aaron Rodgers  Nags       
    34      Jax  DEF     8      Jacksonville  Nags       
    50      Ten   RB    29        Dion Lewis  Nags       
    53      Oak   RB    16    Marshawn Lynch  Nags       
    83      Mia   WR     5    DeVante Parker  Nags       
    93      NYG   WR     4  Sterling Shepard  Nags       
    107     Ten   WR     1  Rishard Matthews  Nags       
    119     Cle   RB     3        Nick Chubb  Nags       
    128     Atl   WR     1     Calvin Ridley  Nags       
    136     Min    K     3        Dan Bailey  Nags       
    175     Hou   QB     6    Deshaun Watson  Nags     AB
    176     Ari   RB    18     David Johnson  Nags     AB
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost                 name                 team keeper
    7       LAC   QB    11        Philip Rivers  Thx Ron and Le'Veon       
    11      Ari   WR    41     Larry Fitzgerald  Thx Ron and Le'Veon       
    43       NE   WR    19       Julian Edelman  Thx Ron and Le'Veon       
    67       TB   RB    23      Ronald Jones II  Thx Ron and Le'Veon       
    79      Chi   WR     7       Anthony Miller  Thx Ron and Le'Veon       
    88      Det   RB    10      Kerryon Johnson  Thx Ron and Le'Veon       
    91      Chi  DEF     3              Chicago  Thx Ron and Le'Veon       
    95       NE   RB     7          Sony Michel  Thx Ron and Le'Veon       
    100     Pit   RB     5         James Conner  Thx Ron and Le'Veon       
    103     Cle   WR     0     Antonio Callaway  Thx Ron and Le'Veon       
    115     Det   WR     5       Kenny Golladay  Thx Ron and Le'Veon       
    118     Phi    K     2         Jake Elliott  Thx Ron and Le'Veon       
    185     Chi   RB     8          Tarik Cohen  Thx Ron and Le'Veon      B
    186     NYG   TE     8          Evan Engram  Thx Ron and Le'Veon      B
    187     Ten   RB    19        Derrick Henry  Thx Ron and Le'Veon      B
    188     Pit   WR     5  JuJu Smith-Schuster  Thx Ron and Le'Veon      B
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost              name              team keeper
    15      Phi   WR    40    Alshon Jeffery  2 Gurley's 1 Cup       
    33      Cle   WR    38     Jarvis Landry  2 Gurley's 1 Cup       
    35      Was   RB    12   Adrian Peterson  2 Gurley's 1 Cup       
    57      Den   WR    18  Emmanuel Sanders  2 Gurley's 1 Cup       
    63      Den   RB    15   Phillip Lindsay  2 Gurley's 1 Cup       
    64      LAR   QB     2        Jared Goff  2 Gurley's 1 Cup       
    68      Was   TE    10       Jordan Reed  2 Gurley's 1 Cup       
    71      Oak   TE     3        Jared Cook  2 Gurley's 1 Cup       
    77      Phi  DEF     5      Philadelphia  2 Gurley's 1 Cup       
    92      Ari   QB     5        Josh Rosen  2 Gurley's 1 Cup       
    104     NYJ   QB     1       Sam Darnold  2 Gurley's 1 Cup       
    116      GB    K     0      Mason Crosby  2 Gurley's 1 Cup       
    121      GB   RB     2       Aaron Jones  2 Gurley's 1 Cup       
    156     Min   WR    28      Adam Thielen  2 Gurley's 1 Cup      B
    157     Min   RB    64       Dalvin Cook  2 Gurley's 1 Cup      B
    158     LAR   RB    40    Todd Gurley II  2 Gurley's 1 Cup     AB
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost              name             team keeper
    17       KC   RB    79       Kareem Hunt  FirstRoundFlops       
    21       NE   QB    28         Tom Brady  FirstRoundFlops       
    29      Chi   RB    64     Jordan Howard  FirstRoundFlops       
    36      Bal   WR    19  Michael Crabtree  FirstRoundFlops       
    45      Chi   TE    31       Trey Burton  FirstRoundFlops       
    96      Ind   TE     4        Jack Doyle  FirstRoundFlops       
    117     LAC   WR     1     Mike Williams  FirstRoundFlops       
    126     Atl  DEF     1           Atlanta  FirstRoundFlops       
    134      KC    K     1   Harrison Butker  FirstRoundFlops       
    140     Jax   QB     1     Blake Bortles  FirstRoundFlops       
    145      NO   TE     1   Benjamin Watson  FirstRoundFlops       
    147     Min   RB     1   Latavius Murray  FirstRoundFlops       
    148     LAC   WR     1   Tyrell Williams  FirstRoundFlops       
    149      SF    K     1      Robbie Gould  FirstRoundFlops       
    150     Sea   WR    24      Doug Baldwin  FirstRoundFlops     AB
    151     Oak   WR    43      Amari Cooper  FirstRoundFlops     AB
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost                 name     team keeper
    27      Car   RB    67  Christian McCaffrey  K-K-Dai       
    31       GB   TE    23         Jimmy Graham  K-K-Dai       
    41      Bal   RB    56         Alex Collins  K-K-Dai       
    76      Pit   QB     0   Ben Roethlisberger  K-K-Dai       
    78       GB   WR     0     Geronimo Allison  K-K-Dai       
    101     NYJ   WR     0        Quincy Enunwa  K-K-Dai       
    106     Atl    K     2          Matt Bryant  K-K-Dai       
    111      SF   RB    11          Matt Breida  K-K-Dai       
    112     Cin   RB     6      Giovani Bernard  K-K-Dai       
    131     Was   QB     1           Alex Smith  K-K-Dai       
    138     Cin   WR     0           Tyler Boyd  K-K-Dai       
    143     Hou  DEF     0              Houston  K-K-Dai       
    163     Min   WR    24         Stefon Diggs  K-K-Dai     AB
    164     NYG   WR    29    Odell Beckham Jr.  K-K-Dai     AB
    165      NE   WR     6          Josh Gordon  K-K-Dai      B
    166     Pit   RB    66         Le'Veon Bell  K-K-Dai     AB
    ------------------------------------------------------------------------------------
        NFLTeam  Pos  cost              name team keeper
    40      LAR   WR    24       Cooper Kupp    G       
    44      Den   WR    38  Demaryius Thomas    G       
    48       NE   RB    29      Rex Burkhead    G       
    51      Cle   RB    22       Carlos Hyde    G       
    54      Jax   WR     0    Dede Westbrook    G       
    55       TB   TE    14       O.J. Howard    G       
    61      Cle   TE     9       David Njoku    G       
    65      Sea   RB    25      Chris Carson    G       
    85       KC   RB     1      Spencer Ware    G       
    89      LAR    K     5     Greg Zuerlein    G       
    97      Bal  DEF     5         Baltimore    G       
    105     Mia   WR     3      Kenny Stills    G       
    109     Atl   QB     3         Matt Ryan    G       
    189     Cle   QB     1      Tyrod Taylor    G       
    190     Phi   QB    10      Carson Wentz    G      B
    191     Det   WR    12  Marvin Jones Jr.    G      B
    ------------------------------------------------------------------------------------
    
