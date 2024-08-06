# Датасет по premier league

***Импорт библиотек***

import pandas as pd

import numpy as np

from matplotlib import pyplot as plt

import sklearn

import seaborn as sns

import plotly.express as px


## ***Запуск проекта***
df = pd.read_csv("/content/PremierLeague.csv")



## ***Ознакомление с дата сетом***
df.head()

df

df.info()

df.describe()

df.isna().sum()


## ***написание условий***
df[["starting_at"]] = df[["starting_at"]].apply(pd.to_datetime)

def define_winner(r):

    if r["home_team_goals"] > r["away_team_goals"]:
    
        return "home_winner"
        
    elif r["home_team_goals"] < r["away_team_goals"]:
    
        return "away_winner"
        
    else:
    
        return "draw"
df["winner"] = df.apply(define_winner, axis=1)
df


d = {}

for team in df.home_team_name.unique():
    df_home = df[df["home_team_name"] == team]
    df_away = df[df["away_team_name"] == team]
    
    counts_home = df_home.value_counts('winner')
    counts_away = df_away.value_counts('winner')
    

    home_winner = 0 if 'home_winner' not in counts_home else counts_home['home_winner']

    away_winner = 0 if 'away_winner' not in counts_home else counts_home['away_winner']

    home_draw = 0 if 'draw' not in counts_home else counts_home['draw']
    

    win_away = 0 if 'away_winner' not in counts_away else counts_away['away_winner']

    lost_away = 0 if 'home_winner' not in counts_away else counts_away['home_winner']

    away_draw = 0 if 'draw' not in counts_away else counts_away['draw']
    

    GF = df_home.home_team_goals.sum() + df_away.away_team_goals.sum()

    GA = df_home.away_team_goals.sum() + df_away.home_team_goals.sum()

    GD = GF - GA
    
    d[team] = {
               "Played": home_winner + win_away + away_winner + lost_away + home_draw + away_draw,
               "Won": home_winner + win_away,
               "Drawn": home_draw + away_draw,
               "Lost": away_winner + lost_away,
        "GF": GF,
        "GA": GA,
        "GD": GD,
               "Points": (home_winner + win_away)*3 + (home_draw + away_draw)}

def highlight_last_n_rows(n=3):
    def highlight(row):
        return ['background-color: #ee5d5d' if row.name >= 17 else '' for _ in row]

    return highlight

def highlight_champion():
    def highlight(row):
        # Check if the row index is the first row
        return ['background-color: gold' if row.name == 0 else '' for _ in row]

    return highlight
 ## ***Создание таблицы***
df_ranked = pd.DataFrame(d)

df_ranked = df_ranked.T

df_ranked = df_ranked.sort_values("Points", ascending=False).reset_index()

## ***Создание круговой диаграммы***
df_rank = (df_ranked.style
                  .apply(highlight_last_n_rows(), axis=1)
                  .apply(highlight_champion(), axis=1))
df_rank

fig = px.pie(df.value_counts("winner").reset_index(), values='count', names='winner', title='Checkin relation for win', hole=.2)

fig.update_traces(textposition='inside', textinfo='percent+label')

fig.update_layout(showlegend=False)

fig.show()
