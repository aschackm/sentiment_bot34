

```python
# Dependencies
import tweepy
import time
import json
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
import numpy
from datetime import datetime
from matplotlib import style
style.use('ggplot')
from config import consumer_key, consumer_secret, access_token, access_token_secret
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
analyzer = SentimentIntensityAnalyzer()
```


```python
# Twitter API Keys
consumer_key = consumer_key
consumer_secret = consumer_secret
access_token = access_token
access_token_secret = access_token_secret
```


```python
# Twitter Credentials
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth, parser=tweepy.parsers.JSONParser())
```


```python
def sentiment_plot():
    # Bot can be found at https://twitter.com/sentiment_bot34 (It will most likely be non functional but tweets of graphs will be visable)
    bot_handle = "@sentiment_bot34"
    request_list = []
    print("Running function")
    
    # Search for tweets mentioning bot handle
    public_tweets = api.search(bot_handle, count=20, result_type="recent")
    print("Searching for mentions")
    
    # Extracting id, author, and target user from tweets
    for tweet in public_tweets["statuses"]:

        tweet_id = tweet["id"]
        tweet_author = tweet["user"]["screen_name"]
    
        for mentions in tweet["entities"]["user_mentions"]:

            request_list.append(mentions["screen_name"])
        print("Extracted search info")
        
    # Dropping duplicate requests by converting to set then back
    target_requests = set(request_list)
    target_requests = list(target_requests)
    #print(f"target_requests = {target_requests}")
 
    # Check bot timeline for past analysis to prevent repeated use on same user
    timeline_tweets = api.user_timeline()
    for tweet in timeline_tweets:
        for mentions in tweet["entities"]["user_mentions"]:
            user = (mentions["screen_name"])
            #print(f"user = {user}")
            for target in target_requests:
                #print(target)
                if target == user:
                    target_requests.remove(target)
                    print(f"{user} has already been analyzed")
                elif target == "sentiment_bot34":
                    target_requests.remove(target)
                    print("Skipping analysis on sentiment_bot34 itself")

    print(f"Requested targets: {target_requests}")    
    for target in target_requests:
        print("Beginning analysis")
        target_user = f"@{target}"
        counter = 1
        sentiments = []
        oldest_tweet = None

    # Loop through 25 pages of tweets (total 500 tweets)
        for x in range(25):
            public_tweets = api.user_timeline(target_user, max_id = oldest_tweet)
            #print(f"Getting {target_user}'s tweets")
    # Sentiment analysis
            for tweet in public_tweets:
                results = analyzer.polarity_scores(tweet["text"])
                compound = results["compound"]
                pos = results["pos"]
                neu = results["neu"]
                neg = results["neg"]
                tweets_ago = counter
                #print("Analyzing tweets")
    # Get Tweet ID, subtract 1, and assign to oldest_tweet
                oldest_tweet = tweet['id'] - 1

    # Add sentiments for each tweet into a list
                sentiments.append({"Date": tweet["created_at"], 
                                  "Compound": compound,
                                  "Positive": pos,
                                  "Negative": neu,
                                  "Neutral": neg,
                                  "Tweets Ago": counter})
                counter += 1

    # Convert sentiments to DataFrame
        print("Creating dataframe")
        sentiments_pd = pd.DataFrame.from_dict(sentiments)
        #sentiments_pd.head()

    # Plot of sentiments
        x_vals = sentiments_pd["Tweets Ago"]
        y_vals = sentiments_pd["Compound"]
        plt.plot(x_vals, y_vals, marker="o", linewidth=0.5, alpha=0.8, color="blue")
        now = datetime.now()
        now = now.strftime("%Y-%m-%d %H:%M")
        plt.title("Sentiment Analysis of Tweets \n (" + now + ") for " + target_user)
        plt.xlim([x_vals.max(),x_vals.min()])
        plt.ylim(-1,1)
        plt.ylabel("Tweet Polarity")
        plt.xlabel("Tweets Ago")
        plt.gcf().subplots_adjust(left=0.15)
        plt.savefig(f"{target_user}.png")  
        print("Plotting tweets")
    # Post to Twitter
        try:
            print("Posting to Twitter")
            api.update_with_media(f"{target_user}.png",
                          f"@%s Here is a sentiment analysis of {target_user}" % tweet_author, in_reply_to_status_id=tweet_id)   
        except tweepy.TweepError as e:
            print(e.reason)
            continue

                
```


```python
# Set timer to run every 5 minutes
while(True):
    sentiment_plot()
    time.sleep(300)
```

    Running function
    Searching for mentions
    Extracted search info
    Extracted search info
    Extracted search info
    Extracted search info
    Extracted search info
    AlexSchackmuth has already been analyzed
    Skipping analysis on sentiment_bot34 itself
    guardian has already been analyzed
    nytpolitics has already been analyzed
    nytimes has already been analyzed
    NYTMetro has already been analyzed
    Requested targets: []
    Running function
    Searching for mentions
    Extracted search info
    Extracted search info
    Extracted search info
    Extracted search info
    Extracted search info
    Extracted search info
    Extracted search info
    Extracted search info
    AlexSchackmuth has already been analyzed
    Skipping analysis on sentiment_bot34 itself
    guardian has already been analyzed
    nytpolitics has already been analyzed
    nytimes has already been analyzed
    NYTMetro has already been analyzed
    Requested targets: ['reneauberjonois', 'realGulDukat', 'BarackObama']
    Beginning analysis
    Creating dataframe
    Plotting tweets
    Posting to Twitter
    Beginning analysis
    Creating dataframe
    Plotting tweets
    Posting to Twitter
    Beginning analysis
    Creating dataframe
    Plotting tweets
    Posting to Twitter

