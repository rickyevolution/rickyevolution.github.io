---
layout: post
title: How Billboard champions are made
---

I apologise for the messy blog last week and I hope it can be done better! Therefore in this week's blog there would be a mini walk through to demonstrate how one can use Python and its very powerful modules to perform data analysis! The dataset that we will be using today is the Billboard Hot 100 data back in 2000. The Billboard chart is the standard for music industry in the United States. Let’s dive in and explore the data!

<h2>Step 1: Load the data and do some preliminary look around!</h2>

{% highlight python %}

import pandas as pd #Pandas is a very powerful module which handles data as a data frame
import numpy as np #Fast and convenient modules for numerical operrations
import scipy as sp #Similar function as numpy more statistics focus
import matplotlib.pyplot as plt #plotting tool for python
import seaborn as sns #A nicer looking plotting tool
from datetime import timedelta #This moduel deals with time data
%matplotlib inline

billboard = pd.read_csv('../assets/billboard.csv')
billboard.head()

{% endhighlight %}


![Billboard.csv head](http://res.cloudinary.com/dexpzle9i/image/upload/v1477219542/Screen_Shot_2016-10-23_at_11.44.27_zun5vw.png)

We start off by importing all the modules that will be (could be) helpful during the analysis. Then the targeted csv file is read in. The head() operation shows the top 5 entries of a data frame. Looking at the result, we can tell that each row represents an individual track. There are 83 columns.  All the entries are self-explanatory apart from the ones that have the format ‘x”n”.week’.  These columns represents the ranking of the track in the ‘n’th week. It is very important to note that this is a relative measure since the tracks entered the board at different dates. It is usually very convenient to use the following operation to gain more insight from a data frame.

{% highlight python %}
billboard.info()
billboard.describe()
{% endhighlight %}

It will provide us with the information such as column name, non-null entries and data type. However for the datasets with larger amount of columns, it would return a very long list which is not desirable. A similar result was also observed with the .describe() operation. The .describe() operation returns a summary statistics for numerical columns (mean, median, )

Jupyter has trouble printing all the columns out at once. Therefore the dataframe was divided into groups of 10 and 1 group is printed at a time. The key things to look for here are any abnormal values in min, max where they should be integers bigger than 0 and less than 100.


{% highlight python %}
for i in np.linspace(0,80,9):
    print billboard.describe().iloc[:,i:i+10]
    print'-------------------------------'
{% endhighlight %}

After the inspections it seems that all the contained values are reasonable. However giving the data a deepe thought, one might wonder whether all the tracks exists on the board continuously. Is it possible for a track to be dropped from the board and return? The following code returns a list of boolean values where a False indicates a track has made a come back.

{% highlight python %}
weeks = billboard.iloc[:,7:]
weeks['nncount'] = weeks.notnull().sum(axis=1)
continuous_lst = []
for i in range(weeks.shape[0]):
    ind = weeks.nncount[i]
    continuous_lst.append(all(weeks.iloc[i,:ind-1].notnull())) #end of list is ind-1 because we have introduced column 'nncount'
print continuous_lst
{% endhighlight %}

It is found out that there were 9 tracks that successfully made a come back!

Here I am going to introduce another operation that is very useful for summary analysis. It is the .value_counts() operation. This operation can be seen as a pivot table showing the counts for each unique value in the series. The following are examples where we can use .value_ counts()!

{% highlight python %}
billboard['track'].value_counts() #To find out whether track names are unique
billboard['genre'].value_counts() #To see how are the music genre distributed
billboard['artist.inverted'].value_counts() #To see how many entries per artist
{% endhighlight %}

Here is a brief summary of the data set:

After the investigation, we can see that there are 317 rows and 83 columns in the data. Out of them there are 6 columns with a data type 'object' (Which would very likely be strings).  It is important to note that the date columns namely 'date entered' and 'date peaked' can possibly be transformed to datetime values for further analysis. The 'time' column can also be transformed into integer to investigate the relationship between the length of a track and its popularity. It is also worthwhile to note that there are two tracks both named "Where I Wanna Be" and it is important not to mix them up. There are a total of 10 genres of music and Rock is the most popular. There are 228 unique artist/groups involved in the dataset. Jay-z has the most track entries of 5. From week 66 onwards all the columns are empty which means no tracks has stayed in the chart for more than 65 weeks. Therefore the extra columns should be removed. The track that stayed the longest is Higher performed by Creed. Finally it is very easy to be tricked by the results returned by .info() operation. This is because it is shown that the number of non-null values decreases with the week which is very normal. However under a more detailed investigation it appears that those numbers were not composed of the same group of tracks everytime. For example track 'a', 'b', 'c' are both in the board at week 1 (count = 3). Track 'a' dropped out in week 2 (count =2). In the third week track a comes back but track 'b' and 'c' drops out (count =1). This behaviour can potentially complicates the analysis and catch out the less careful analyst.

<h2>Step 2: Clean the data and turn it into something more useful!</h2>

Although the data seems interesting, it must first be cleaned before using! Cleaning a data set speeds up the analysis process and lowers the chances of error occuring. It is mentioned that in our dataset there are quite a few empty columns. Therefore it would be nice to get rid of them

{% highlight python %}
billboard = billboard.iloc[:,:72] #get rid of the empty columns by slicing
{% endhighlight %}

It is also important to note that some columns have names that are either unnecessarily long or consists of a '.' character. The former makes the column difficult to undersntad/read and the later might cause Python to exhibit unpredictable behaviour due to the dot notation. Therefore it is also a good practise to rename colums to as simple as possible. If spaces are need between words, an underscore '_' character can be used.

{% highlight python %}
billboard = billboard.rename(columns={'artist.inverted':'artist', 
				  'date.entered':'date_entered',
				  'date.peaked':'date_peaked'})
{% endhighlight %}

As mentioned in step 1, we would like to look at the relationship between a track's length and its other properties. However the track length is currently in a format of mm:ss as a string. It is not very easy to use so a function is written to convert that into number of seconds as integers. The time column would not be very useful either after the conversion so it will be dropped. The same case also applies for 'date_entered' and 'date_peaked'. Therefore their data types are transformed from strings into datetime which allows us to measure date difference as timedeltas. It would also be nice to create an extra column where we store the month of 'date_entered'. This would allow us to conveniently investigate any cyclic behaviour that might exsist.

{% highlight python %}
def time_con(str):
    lst = str.split(':')
    secs = int(lst[0])*60 + int(lst[1])
    return secs
billboard['time_in_seconds'] = billboard.time.apply(time_con)
billboard.drop(['time'], axis =1, inplace = True)
billboard['datetime_entered'] = pd.to_datetime(billboard['date_entered'])
billboard['datetime_peaked'] = pd.to_datetime(billboard['date_peaked'])
billboard.drop(['date_entered','date_peaked'], axis =1, inplace = True)
billboard['month_entered'] = billboard['datetime_entered'].apply(lambda x:x.month)
{% endhighlight %}

The huge number of columns in the original dataframe is not optimal for some data analysis such as time series.Using the .melt() function in Pandas, one can pivot the weekly ranking data into rows. This operation changes the 65 non-empty ranking columns into 2 columns namely 'week' and 'rank'. There will now be multiple entries for each track, one for each week on the Billboard rankings. The following code describes the operations that prepare and perform the melt.

{% highlight python %}
# Preperation
def re_name(x):
    if (len(x)==10) & (x[0]=='x'):
        return int(x[1:3])
    elif (x[0]=='x'):
        return int(x[1])
    else:
        return x
billboard.rename(columns=lambda x: re_name(x), inplace=True)

#Actual melt and clean up
melt_value_list = list(billboard.columns)[4:-6] #The columns to transform
melt_index_list = list(billboard.columns)[:4] + list(billboard.columns)[-6:] #The columns to keep
billboard_melted = pd.melt(billboard, id_vars = melt_index_list, value_vars= melt_value_list)
billboard_melted.rename(columns = {'variable':'week', 'value':'rank'}, inplace=True) #Rename columns
billboard_melted = billboard_melted[billboard_melted['rank'].notnull()] #Remove redundant columns
# Create a column to show the current week
dt = timedelta(days=7)
billboard_melted['week'] = billboard_melted['week'].astype(int)
billboard_melted['datetime_current'] = billboard_melted['datetime_entered'] + (billboard_melted['week']-1)*dt
billboard_melted['week'] = billboard_melted['week'].astype(int)
billboard_melted['genre'] = billboard_melted['genre'].astype('category')
{% endhighlight %}

We now have 2 data frames. It would be nice to keep the original dataframe too as different dataframes are better suited at different type of analysis. Therefore pivot tables are generated from the melted dataframe and merged back into the original dataframe. Pivot tables are very useful tools in data analysis as it allows us to compute statistics for different groupings. In our case they are used to find the number of times a track appears in the melted dataframe (i.e. How long did it stay in the dataframe in total) and the minimum rank number (i.e. the highest rank). We have discovered that there are 2 tracks that have the same name so we have to be very careful when performing the pivot table analysis. One way to solve this problem is to create an new column named 'artist_track' which combines the strins from the artist column and the track column so every item in that column would be unique. For our purpose, the original 'track' column will be dropped. A column called 'total_weeks_in' is introduced to give information of how long has the track stayed in the chart. An example of the new columns are showed below.

![Billboard extra columns](http://res.cloudinary.com/dexpzle9i/image/upload/v1477243815/Screen_Shot_2016-10-23_at_18.29.36_yecgk4.png)

<h2>Step 3: Visualisations to spot interesting trends!</h2>

Having cleaned the data, it is now time to do some plotting! I would like to know what factors are likely to be correlated with how successful the track is (High rank and long staying). The following factors will be investigated:
- Track title length (really?)
- Track length
- Genre
- Number of days to peak (some kind of momentum)
- Months entered

First lets look at how the our targets (maximum rank and number of weeks in chart). It can be seen that there are quite a lot of tracks (nearly 40) made it to the top bin which is on the left hand side. Then the rest are distributed quite evenly across the spectrum. The staying of the tracks are also interesting. It should be expected that as time goes by the number of tracks staying in the chart would drop. It is surprising to see that there is a very obvious peak at the bin around 20-23. That means most tracks dropped out at their early 20s(week).

![Billboard max rank distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477245701/Screen_Shot_2016-10-23_at_19.00.54_ee4rfm.png)
![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477244956/Screen_Shot_2016-10-23_at_18.48.41_zfbffs.png)

Now we proceed to look at how the different variables and their effect on track popularity. 

<h4>Track title length</h4>

![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477251597/Screen_Shot_2016-10-23_at_19.36.25_qystc3.png)
![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477251808/Screen_Shot_2016-10-23_at_20.42.56_kud0qe.png)

The median track title length is 3 words and the marjority of tracks have a title length of 2 to 4. However when we look at the bar chart, we would discover that there seems to be little correlation between track title length and how successful the track is. The most we can say is that tracks titles with an odd number length seems to be rnak higher than the others. However we would need further investigations to prove that

<h4>Track length</h4>

![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477252191/Screen_Shot_2016-10-23_at_20.48.29_xfvfci.png)

![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477252201/Screen_Shot_2016-10-23_at_20.49.26_zsgldg.png)

From the above plots we can see that most trakcs last less than 5 minutes. In terms of rank there seems to be little correlation (but the excpetionally long ones i.e. over 7 minutes seems to be doing pretty well). It seems that if a track would like to stay longer in the chart, it'd be better for to last fewer than 6 minutes. This is because the longer few could not last over 20 weeks. Speaking of which this scatter plot show us a very interesting pattern. There is a huge cluster around the 20 minutes mark which we described earlier. This mark forms a very recognisable line in the plot. Therefore it can be suspected that something artificial is going on.The explanation can be found in <a href = 'https://en.wikipedia.org/wiki/Billboard_Hot_100'>this wikipedia page</a> which stated that: 

"a song is permanently moved to "recurrent status" if it has spent 20 weeks on the Hot 100 and fallen below position number 50. Additionally, descending songs are removed from the chart if ranking below number 25 after 52 weeks.Exceptions are made to re-releases and sudden resurgence in popularity of tracks that have taken a very long time to gain mainstream success. These rare cases are handled on a case-by-case basis and ultimately determined by Billboard's chart managers and staff."

<h4>Genre</h4>

![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477253143/Screen_Shot_2016-10-23_at_21.05.15_sqs5t3.png)

![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477253143/Screen_Shot_2016-10-23_at_21.05.04_t1lvmh.png)

It is obvious even from a glance that rock music dominated the chart in 2000. The number of rock music, having a count of over 130, almost doubled the second place - country. Some music types such as gospel, jazz and reggae
were almost non exsistent. In terms of ranking, the few jazz entrants did significantly better than other types of music averaging within top 10. The other trend to look out for is the relatively poor performance of R&B with a below average rank and below average in-chart time.

<h4>Number of days to peak</h4>

![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477254012/Screen_Shot_2016-10-23_at_21.19.46_lzs5cs.png)

![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477253936/Screen_Shot_2016-10-23_at_21.18.03_ql6jib.png)

This variable seems to be able to shows more correlation with how well the track performs. From the histogram we know that the majority of the tracks took less than 100 days to reach their peak. This is a reasonable observation as we know that many of them will be gone by about 140 days (20 weeks). It seems that the higher reaching tracks usually takes longer (although not by much) to reach the top. Therefore this momentum could be an indicator of whether a track would reach the very top. More interestingly the number of days to peak correlates really well with how long the track stays in the chart. This is kind of trivial because obvious the grow in one can only also result in a grow in the other. However if we think about it there is always a chance that a track drops straight out of the chart once its peaked and this plot can be used to argue against such proposal.

<h4>Month entered</h4>

![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477255242/Screen_Shot_2016-10-23_at_21.39.46_u11cut.png)

![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477255242/Screen_Shot_2016-10-23_at_21.40.07_kvup34.png)

The number of tracks entering the chart for each different month is very similar. The performance though varies by a larger degree. The entries from April, May and October had a reached a higher rank on average and also stayed in the chart for longer.

<h3>The targets against each other...</h3>

![Billboard duration distribution](http://res.cloudinary.com/dexpzle9i/image/upload/v1477255669/Screen_Shot_2016-10-23_at_21.47.25_lcxedm.png)

These 2 variables related relatively well as expected. Popular tracks tend to reach a higher rank and stay in the chart for longer. However there is an outlier at the bottom left corner. This track reached within top 10 (7th place to be exact) but only existed in the chart for 5 weeks. This is a jazz track (which matches our observation well) by Kenny G called Auld Lang Syne (The Millenium Mix). Its behavious could be due to the fact that the song itself is very old but well known. Therefore with a good performance it can catch people's attention. However the hype was soon over and excitment didn't last long.

<h2>The problem statement?</h2>

From the EDA and plottings in previous sections, we were able to demonstrate the (lack of) relaionship between music tracks properties. It is also interesting to discover that there were some tracks that were able to make a come back after dropping out of the chart. This is not a common characteristic which is worth more investigation. Therefore the problem statement here would be to find out whether these resurrected tracks are different to the others in terms of: 
- maximum ranks
- duration

<h4>Procedure</h4>
- First look at the data as a whole. Since we are looking at data from 2000 only it is safe to assume that our data is the population.
- The null hypothesis would be that they are the same. Alternate hypothesis would be that they are different!
- In order to determine the difference between sample and population, a t-test will be used since although we have the population mean and standard deviation, the sample size is small (under 30)
- Set a proper alpha level
- Find out the mean and standard deviation of max rank for the population
- Find out the number of come-back tracks (this would serve as the sample size)
- Due to Central Limit Theorem, the smple mean would be equal to the population mean
- The sample standard deviation would be the sqrt(sum of squared deviation/sample size -1)
- Find the mean for the come back group
- Find the difference of the means and divide by (sample standard deviation)/sqrt(sample size)
- Determine degrees of freedom
- Find the p-value using a t-table and determine whether to reject or accept the null hypothesis
- Repeat process with duration in the chart

The code used is shown in the below snippet. Note that the same analysis was carried out for the duration in chart analysis too.
{% highlight python %}
max_rank_population_mean = billboard['max_rank'].mean()
max_rank_population_std = billboard['max_rank'].std()
come_back_index = [i for i in range(len(continuous_lst)) if not continuous_lst[i]]
sample_size = len(come_back_index)
come_back_max_rank_list = [billboard['max_rank'][i] for i in come_back_index]
sum_of_sq_deviation = np.sum([(i - max_rank_population_mean)**2 for i in come_back_max_rank_list])
sample_standard_deviation = (sum_of_sq_deviation/(sample_size-1))**0.5
come_back_max_rank_list_mean = np.mean(come_back_max_rank_list)
t_stat = (come_back_max_rank_list_mean - max_rank_population_mean) / (sample_standard_deviation/(len(come_back_index)**0.5))
{% endhighlight %}

Since this is not a very critical scientific test, an alpha level of 0.05 is good enough. The degrees of freedom is 10 - 1 =9. After consulting the t-table it is determined that the t-statistic is 0.505. Since our aplpha level is 0.05, it requires a t-statistics of >2.262 or <-2.262 to reject the null hypothesis. In conclusion it is reasonable to accept the null hypothesis and say that the maximum rank of the come-back tracks are reasonably similar to the population. For duration we have a t-statistics of 1.066. It is also not possible for us to declare that the come-back tracks have a significantly different total staying period comparing to the whole population. Since the sample size is so small, it is very difficult to reject the null hypothesis. Therefore it would be very beneficial to include more samples from other years in order to further investigate the properties of these special tracks that made a come back.



