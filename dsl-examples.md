```python
import helpers

@failure >> terminal
def to_be_objective(tweet):
    subjectivity = helpers.get_subjectiviy_from_spanish(tweet['text'])
    return subjectivity > 0.4

@failure >> terminal
def not_about_traffic(tweet):
    lower_words = helpers.get_lowercase_tweet_words(tweet)
    keywords = ['carretera','trafico','tráfico','coche']
    return 'atasco' in lower_words and set(lower_words).intersection(set(keywords))

all_tweets = stream('Twitter').filter(helpers.is_tweet)
@check 'all spanish tweets are objective':
    spanish_tweets = all_tweets.filter(helpers.in_spanish)
    expect(spanish_tweets)(to_be_objective)

madrid_tweets = stream('TwitterMadrid').filter(helpers.is_tweet)
@check 'no one in madrid talks about traffic':
    expect(madrid_tweets)(not_about_traffic)

```

```python
import helpers

@failure >> terminal
def has_less_than_two_emojis(tweet):
    return helpers.count_emoji(tweet['text']) <= 2

@failure >> terminal
def original_tweet_has_less_than_50_retweets(tweet):
    return tweet['retweeted_status']['retweet_count'] <= 50

@failure >> terminal
def hashtag_at_most_twelve_letters_long(tweet):
    return all(len(h['text']) <= 12 for h in tweet['entities']['hashtags'])



tweets = stream('twitter').filter(helpers.is_tweet)

@check 'all retweeted tweets should have less than 50 retweets':
    retweeted_tweets = tweets.filter(helpers.is_retweet)
    expect(retweeted_tweets)(original_tweet_has_less_than_50_retweets)

@check 'all japanese tweets should have less than 2 emojis':
    tweets_in_japanese = tweets.filter(helpers.is_japanese_tweet)
    expect(tweets_in_japanese)(has_less_than_two_emojis)

@check 'all spanish tweets should have hashtags shorter that 12 characters':
    spanish_hashtags = tweets.filter(helpers.is_spanish_tweet).filter(helpers.has_hashtags)
    expect(spanish_hashtags)(hashtag_at_most_twelve_letters_long)
```
