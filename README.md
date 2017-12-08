# 2017_NFL_Spreads_Data
I have created a dataframe to analyze Vegas' predictions on NFL games in 2017.  It involves both teams, the spread (from Westgate), and if the projected team (it does not matter if they covered the spread, just if they won)

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

teams = ['49ers', 'Bears', 'Bengals', 'Bills', 'Broncos', 'Browns', 'Bucs', 'Cardinals', 'Chargers', 'Chiefs', 'Colts', 'Cowboys', 'Dolphins', 'Eagles', 'Falcons', 'Giants', 'Jaguars', 'Jets', 'Lions', 'Packers', 'Panthers', 'Patriots', 'Raiders', 'Rams', 'Ravens', 'Redskins', 'Saints', 'Seahawks', 'Steelers', 'Texans', 'Titans', 'Vikings'] 

#import data
nfl = pd.read_csv('NFL_vs_spread_2017.csv')

#label columns
nfl.columns = ['Week', 'Away Team', 'Away Score', 'Home Team', 'Home Score', 'Spread', 'H/A', 'Correct', 'Over/Under']

#Determine if games hit the over or under and make a column for it
total_score = nfl['Away Score'] + nfl['Home Score']
nfl['Total Score'] = total_score
nfl['O/U'] = np.where((nfl['Total Score'] > nfl['Over/Under']), 'Over' or (nfl['Total Score'] < nfl['Over/Under']), 'Under')

#Give the counts of all the spreads
nfl_count = nfl.groupby(nfl['Spread']).count()

#Find all the averages for each spread
nfl_percentages = nfl.groupby(nfl['Spread']).mean()

################## LOSES ##################
#Returns dataframe for all games not predicted correctly
loses = nfl.loc[nfl['Correct'] == 0]
#print loses

#seperated the loses by the spreads
loses_by_spread_totals = loses.groupby(loses['Spread']).count()
loses_by_spread_mean = loses.groupby(loses['Spread']).mean()
#print loses_by_spread_totals
#print loses_by_spread_mean

################## WINS ##################
#all games that resulted in straight up wins
wins = nfl.loc[nfl['Correct'] == 1]
#print wins

#seperated the wins by the spreads
wins_by_spread_totals = wins.groupby(wins['Spread']).count()
wins_by_spread_mean = wins.groupby(wins['Spread']).mean()
#print wins_by_spread_totals
#print wins_by_spread_mean

#New dataframe with wins and loses counts and averages
wins_loses = pd.concat([wins_by_spread_totals['Correct'], 
loses_by_spread_totals['Correct'], nfl_count['Correct'], nfl_percentages['Correct']], axis = 1, keys = ['Wins', 'Loses', 'Total', 'Win %'])
#print wins_loses

#Check out all of a teams games this year
#user_team = str(raw_input('Which team would you like to see? '))
#team_record = nfl[(nfl['Home Team'] == user_team) | (nfl['Away Team'] == user_team)]
#print
#print user_team, team_record['Correct'].mean()
#print
#print team_record

percents = []

###Check average correctness for all teams
for team in teams:
    team_info = nfl[(nfl['Home Team'] == team) | (nfl['Away Team'] == team)]
    percents.append(team_info['Correct'].mean())
    #print team, team_info['Correct'].mean()
    
#Dataframe of the all teams and their winning percentage
teams_percent = pd.DataFrame([teams, percents]).transpose()
teams_percent.columns = ['Team', 'Winning %']
teams_percent = teams_percent.set_index(['Team'])
#print teams_percent

#Bar graph of each teams and their percentage of being correct
#x = range(len(teams))
#plt.bar(x, teams_percent['Winning %'], color = 'blue')
#plt.xticks(x, teams, rotation = 'vertical')
#plt.yticks([0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0])
#plt.xlabel('Team')
#plt.ylabel('Percent Correctly Predicted')
#plt.title('Vegas Spreads Winning Percetage by Team')
#plt.grid(True)
#plt.show()

##### Projected Winning Team Won
home_team_fav = nfl.loc[nfl['H/A'] == 'H']
home_team_percent = pd.DataFrame([home_team_fav['Home Team'], home_team_fav['Correct']]).transpose()
home_team_percent.columns = ['Team', 'Correct']
away_team_fav = nfl.loc[nfl['H/A'] == 'A']
away_team_percent = pd.DataFrame([away_team_fav['Away Team'], away_team_fav['Correct']]).transpose()
away_team_percent.columns = ['Team', 'Correct']

favorites = [home_team_percent, away_team_percent]

fav = pd.concat(favorites)
team_fav = pd.DataFrame(fav.groupby(fav['Team']).sum())
team_fav_count = pd.DataFrame(fav.groupby(fav['Team']).count())

combine_fav = [team_fav, team_fav_count]
winners_percent = pd.concat(combine_fav, axis = 1)
winners_percent.columns = ['Wins', 'Favored']
winners_percent['Percent Won'] = winners_percent['Wins']/winners_percent['Favored']

##### Projected Losing Team Lost
home_team_fav_loser = nfl.loc[nfl['H/A'] == 'H']
home_team_percent_loser = pd.DataFrame([home_team_fav_loser['Away Team'], home_team_fav_loser['Correct']]).transpose()
home_team_percent_loser.columns = ['Team', 'Correct']
away_team_fav_loser = nfl.loc[nfl['H/A'] == 'A']
away_team_percent_loser = pd.DataFrame([away_team_fav_loser['Home Team'], away_team_fav_loser['Correct']]).transpose()
away_team_percent_loser.columns = ['Team', 'Correct']

favorites_loser = [home_team_percent_loser, away_team_percent_loser]

fav_loser = pd.concat(favorites_loser)
team_fav_loser = pd.DataFrame(fav_loser.groupby(fav_loser['Team']).sum())
team_fav_count_loser = pd.DataFrame(fav_loser.groupby(fav_loser['Team']).count())

combine_fav_loser = [team_fav_loser, team_fav_count_loser]
winners_percent_loser = pd.concat(combine_fav_loser, axis = 1)
winners_percent_loser.columns = ['Loses', 'Not Favored']
winners_percent_loser['Percent Lost'] = winners_percent_loser['Loses']/winners_percent_loser['Not Favored']

finals = [winners_percent, winners_percent_loser]
final = pd.concat(finals, axis = 1)


#### Check probabilities
favored_team = str(raw_input('Favored Team: '))
underdog_team = str(raw_input('Opponent: '))

probability = final['Percent Won'][favored_team] * final['Percent Lost'][underdog_team]
print probability
