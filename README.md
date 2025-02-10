# proekt

import pandas as pd
import seaborn as sns
import streamlit as st
import matplotlib.pyplot as plt

df = pd.read_csv('C:/Users/georg/Downloads/Opta_Data_Soccer_Schedule_and_Results_Data_Sample_Examples.xls')

df = df[['REGION','COUNTRY','COMPETITION', 'SEASON', 'DATE_TIME','HOME','AWAY', 'HOME_SCORE', 'AWAY_SCORE','VENUE']]

#display the initial table
from streamlit_dynamic_filters import DynamicFilters


dynamic_filters = DynamicFilters(df, filters=['COUNTRY','HOME', 'AWAY'])

dynamic_filters.display_filters()

dynamic_filters.display_df()



#display the features

st.write('Now lets explore the statistics about goals')
results = df[['HOME_SCORE','AWAY_SCORE']]


diff = (results['HOME_SCORE']-results['AWAY_SCORE'])

diff.describe()

fig = sns.histplot(diff, bins = 13)

st.pyplot(fig.figure)
   
st.write('We can say that our data is normally distributed according to the histogram')


plt.figure()
home = df['HOME_SCORE']
away = df['AWAY_SCORE']

fig1 = sns.histplot(home,bins = 7)
st.pyplot(fig1.figure)
plt.figure()
fig2 = sns.histplot(away,bins = 6)
st.pyplot(fig2.figure)

st.write('Now lets compare. Blue histogram is realted to away goals, while orange to home')
fig1c = sns.histplot(home,bins=7)
st.pyplot(fig1c.figure)
st.write('As we can see away goals are more rare, so home factor really played the role in EPL in season 2021-22')

counts = results.groupby(['HOME_SCORE','AWAY_SCORE']).size().reset_index(name='Count')
fig3 = counts.plot.scatter(x = 'HOME_SCORE',y = 'AWAY_SCORE', c = 'Count', s = 'Count')
st.pyplot(fig3.figure)
plt.figure()
st.write('As we can see the most popular result is 1:1. That is quite surprising as usually the most frequent result is 0:0')


df['RESULT_HOME'] = 1*(df['HOME_SCORE']== df['AWAY_SCORE'])+3*(df['HOME_SCORE'] > df['AWAY_SCORE'])
df['RESULT_AWAY'] = 1*(df['HOME_SCORE']== df['AWAY_SCORE'])+3*(df['HOME_SCORE'] < df['AWAY_SCORE'])


homestd = df[['HOME','RESULT_HOME']].groupby(['HOME']).sum().sort_values(by = 'RESULT_HOME',ascending = False)
awaystd = df[['AWAY','RESULT_AWAY']].groupby(['AWAY']).sum().sort_values(by = 'RESULT_AWAY',ascending = False)


homest = df[['HOME','RESULT_HOME']].rename(columns = {'HOME':'TEAM', 'RESULT_HOME':'RESULT'}).groupby(['TEAM']).sum()
awayst = df[['AWAY','RESULT_AWAY']].rename(columns = {'AWAY':'TEAM', 'RESULT_AWAY':'RESULT'}).groupby(['TEAM']).sum()

plt.figure()
res = homest+awayst
ax0 = sns.barplot(res.sort_values(by = 'RESULT',ascending = False),y="TEAM", x="RESULT", errorbar=None, orient="y")#, hue = 'RESULT', palette = sns.color_palette("magma", as_cmap=True))
ax0.bar_label(ax0.containers[0], fontsize=10);
st.pyplot(ax0.figure)

plt.figure()
ax = sns.barplot(homestd,y="HOME", x="RESULT_HOME", errorbar=None, orient="y", hue = 'RESULT_HOME', palette = sns.color_palette("magma", as_cmap=True))
ax.bar_label(ax.containers[0], fontsize=10);
st.pyplot(ax.figure)
plt.figure()
ax1 = sns.barplot(awaystd,y="AWAY", x="RESULT_AWAY", errorbar=None, orient="y", hue = 'RESULT_AWAY', palette = sns.color_palette("magma", as_cmap=True))
ax1.bar_label(ax1.containers[0], fontsize=10);
st.pyplot(ax1.figure)
plt.figure()

ax2 = sns.barplot((homest-awayst).sort_values(by = 'RESULT',ascending = False),y="TEAM", x="RESULT", errorbar=None, orient="y", hue = 'RESULT', palette = sns.color_palette("coolwarm", as_cmap=True))
ax2.bar_label(ax2.containers[0], fontsize=10);
st.pyplot(ax2.figure)



st.write('Look for results for each team at each month')
df['MONTH'] = pd.to_datetime(df['DATE_TIME'], format="%Y-%m-%d %H:%M:%S.%f").dt.month
df['PLAYED'] = 1
homest1 = df[['HOME','RESULT_HOME','MONTH','PLAYED']].rename(columns = {'HOME':'TEAM', 'RESULT_HOME':'RESULT', 'PLAYED':'GAMES'}).groupby(['TEAM','MONTH'],as_index=False).sum()
awayst1 = df[['AWAY','RESULT_AWAY','MONTH','PLAYED']].rename(columns = {'AWAY':'TEAM', 'RESULT_AWAY':'RESULT','PLAYED':'GAMES'}).groupby(['TEAM','MONTH'],as_index=False).sum()


vertical_concat = pd.concat([homest1,awayst1], axis=0)
final2 = vertical_concat.groupby(['TEAM','MONTH'],as_index=False).sum()



st.write('We can display results for each team (for example, Tottenham), to understand the dynamics')

plt.figure()
fig5, ax = plt.subplots()
fig5 = sns.lineplot(final2[final2['TEAM']=='Arsenal FC'],x='MONTH',y='RESULT',label = 'Tottenham FC',ax=ax)
ax.set_xlim(0,13)
ax.set_xticks(range(0,13))
st.pyplot(fig5.figure)
plt.figure()

