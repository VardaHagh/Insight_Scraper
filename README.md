# Analyzing Insight Fellows

I’ve been interested in [Insight Data Science Fellowship](http://insightdatascience.com) for a while and I’m going to apply for their summer 2018 fellowship in Silicon Valley. This is a very prestigious and competitive program, so I wanted to know what my chances of getting into the program are. I did some web scraping to extract data from their [FELLOWS](http://insightdatascience.com/fellows) webpage and did some simple analysis. Here’s a simple explanation about the process:

First, I imported the needed libraries:

```
from bs4 import BeautifulSoup
from collections import OrderedDict
import json
import pandas as pd
import numpy as np
import requests
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
%matplotlib inline
import seaborn as sns
sns.set()
```

I used BeautifulSoup for scraping the webpage:

```
url_to_scrape = 'http://insightdatascience.com/fellows'
r = requests.get(url_to_scrape)
soup = BeautifulSoup(r.text,"html.parser")
```

By inspecting the elements on the Insight webpage, I realized that fellows are stored in lists with 100 elements in each. So I took each list and extracted the info I needed out of them. I also made a dictionary with Name, Title, Company, Project, and Background keys to collect the data:

```
rosters = soup.findAll('div', class_="fellows_list w-dyn-list")
data = {'Name':[],'Title':[],'Company':[],'Project':[],'Background':[], 'Flag':[]}
```
The next step was to extract and assign the right values to each key. There was some missing data on fellows’ background, so I created some flags to filter out those entries that didn’t have any background info:
```
for i,roster in enumerate(rosters):
    fellows = roster.findAll('div', class_="w-clearfix w-dyn-items w-row")
    if len(fellows)!=0:
        fellows = fellows[0].findAll('div', class_='fellow_item w-dyn-item w-col w-col-2')
        for fellow in fellows[:]:

            exception = False
            
            try:
                name = fellow.find('div', class_="tooltip_name").text
                data['Name'].append(name)
            except:
                exception = True
                data['Name'].append('')
            
            title = fellow.find('div', class_="toottip_title").text
            data['Title'].append(title)
            
            try:
                company = fellow.find('div', class_="tooltip_company").text
                data['Company'].append(company)
            except: 
                exception = True
                data['Company'].append('')
            
            project = fellow.find('div', class_="tooltip_project").text
            data['Project'].append(project)
            
            background = fellow.find('div', class_="tooltip_background").text
            if len(background.split(','))==3:
                data['Background'].append(background)
            else:
                exception = True
                data['Background'].append(background)
            
            if exception:
                data['Flag'].append(1)
            else:
                data['Flag'].append(0)
```
Then I made a Pandas data-frame out of my dictionary values and split the background data into three sub-categories, including the major, the degree, and the university that each fellow is affiliated with:

```
columns = ['Name','Title','Company','Project','Background','Flag']
df = pd.DataFrame(data,columns=columns)
df = df[df['Flag']!=1]
background_split = df['Background'].apply(lambda x: pd.Series(x.split(',')))
background_split.rename(columns={0:'Major',1:'University',2:'Degree'},inplace=True)
background_split = background_split[['Major','University','Degree']]
df.drop('Background',1,inplace=True)
df = pd.concat([df,background_split],axis=1)
df['Degree'] = df['Degree'].replace({r'[^\x00-\x7F]+':'',r'\n':''}, regex=True, inplace=False)
df['Name'] = df['Name'].replace({r'[^\x00-\x7F]+':'',r'\n':''}, regex=True, inplace=False)
```
 
The final data frame looks like this:
<div align="center">
<img src="https://vgy.me/OiLUOS.png" alt="OiLUOS.png" height="320px">
</div> 

> Now the fun begins! 
First I wanted to know what the distribution of degrees for the accepted fellows looks like. Here are the results:
```
def clean_text(row):
    # return the list of decoded cell in the Series instead 
    return row.encode('ascii', 'ignore').strip().decode('utf-8')

df['Degree'] = df['Degree'].apply(lambda x: clean_text(x))

fig,ax = plt.subplots(1,1,figsize=(10,10))
df['Degree'].value_counts()[:7].plot(kind='barh',ax=ax,cmap = 'plasma')
ax.set_xlabel('Number')
ax.set_ylabel('Title')
plt.tight_layout()
plt.show(fig)
```
<div align="center">
<img src="https://vgy.me/sNTxSm.png" alt="sNTxSm.png" height="500px">
</div> 

It seems like most of their fellows have got the fellowship right after their Ph.D. This is good news for me, since I’m planning to apply right after my Ph.D. as well.

Then I wanted to know how applicants with a degree in physics were represented:
```
fig,ax = plt.subplots(1,1,figsize=(10,10))
participants = df['Major'].value_counts()
mask = participants > 5 # majors with more than 5 participants
participants[mask].plot(kind='barh',ax=ax, cmap = 'plasma')
plt.tight_layout()
plt.show(fig)
```
<div align="center">
<img src="https://vgy.me/s6cKc6.png" alt="s6cKc6.png" height="500px">
</div> 


Woohoo, PHYSICS ROCKS! It seems like a large proportion of Insight fellows, had their Ph.D. degree in Physics. By looking at the bar chart, we can see that many of them have indicated that their degree was in physics. But some were more specific and also included their field of research in physics, which still counts as physics. The This gives me a lot of hope, as it’s clear that the majority of Insight fellows have been physicists so far.

I also looked at the most represented schools. Unfortunately, Arizona State University is not one of them!

```
fig,ax = plt.subplots(1,1,figsize=(10,10))
df['University'].value_counts(ascending=False)[0:50].plot(kind='barh',ax=ax, cmap = 'plasma')
plt.tight_layout()
plt.show(fig)
```
<div align="center">
<img src="https://vgy.me/EBhUgA.png" alt="EBhUgA.png" height="500px">
</div> 

The last chart that I looked into was the list of companies where the insight fellows have ended up. This looks very promising, since some of the companies I love including Facebook, Google, Microsoft, and IBM are in the top 30:

```
fig,ax = plt.subplots(1,1,figsize=(10,10))
df['Company'].value_counts()[1:31].plot(kind='barh',ax=ax, cmap = 'plasma')
plt.tight_layout()
plt.show(fig)
```
<div align="center">
<img src="https://vgy.me/K1C3VQ.png" alt="K1C3VQ.png" height="500px">
</div> 

Things look good so far. I am really looking forward to submitting my application in the next few days and I hope I can get in.

## Future Work
I would like to look into the projects of all fellows and see if I can see a pattern between their project title and the companies that have hired them. I would like to maximize my chance of getting employed by one of companies I like, so in case I get the Insight Data Science Fellowship for Summer 2018, I’ll invest some time to see if there is a correlation between the types of projects fellows work on and the companies that would hire them:

```
results = set()
df[df['Company']=='Google'][['Project','Major']]
```
<div align="center">
<img src="https://vgy.me/vbhgko.png" alt="vbhgko.png" height="200px">
</div> 

All of the code is included in my IPython notebook above. There is also a cool heat map of Majors and Degrees in the notebook!
