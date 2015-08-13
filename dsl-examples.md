### Testing syntax:

```python
import helpers

def to_be_objective(tweet):
    subjectivity = helpers.get_subjectiviy_from_spanish(tweet['text'])
    return subjectivity < 0.4

def not_about_traffic(tweet):
    lower_words = helpers.get_lowercase_tweet_words(tweet)
    keywords = ['carretera','trafico','tráfico','coche']
    return 'atasco' in lower_words and set(lower_words).intersection(set(keywords))


@failure >> terminal
@check 'all spanish tweets are objective':
    all_tweets = stream('Twitter').filter(helpers.is_tweet)
    spanish_tweets = all_tweets.filter(helpers.in_spanish)
    expect(spanish_tweets)(to_be_objective)


@failure >> slack/developers
@check 'no one in madrid talks about traffic':
    madrid_tweets = stream('TwitterMadrid').filter(helpers.is_tweet)
    expect(madrid_tweets)(not_about_traffic)

```

```python
import helpers

def has_less_than_two_emojis(tweet):
    return helpers.count_emoji(tweet['text']) <= 2

def original_tweet_has_less_than_50_retweets(tweet):
    return tweet['retweeted_status']['retweet_count'] <= 50

def hashtag_at_most_twelve_letters_long(tweet):
    return all(len(h['text']) <= 12 for h in tweet['entities']['hashtags'])


tweets = stream('twitter').filter(helpers.is_tweet)

@failure >> trello/qa
@check 'all retweeted tweets should have less than 50 retweets':
    retweeted_tweets = tweets.filter(helpers.is_retweet)
    expect(retweeted_tweets)(original_tweet_has_less_than_50_retweets)


@failure >> terminal
@check 'all japanese tweets should have less than 2 emojis':
    tweets_in_japanese = tweets.filter(helpers.is_japanese_tweet)
    expect(tweets_in_japanese)(has_less_than_two_emojis)


@failure >> slack/theboss
@check 'all spanish tweets should have hashtags shorter that 12 characters':
    spanish_hashtags = tweets.filter(helpers.is_spanish_tweet).filter(helpers.has_hashtags)
    expect(spanish_hashtags)(hashtag_at_most_twelve_letters_long)
```

### Watching syntax

When the original syntax doesn't match with your use case, you can use this other one:

```python
import helpers

def subjective_tweets(tweet):
    subjectivity = helpers.get_subjectiviy_from_spanish(tweet['text'])
    return subjectivity > 0.4

def about_traffic(tweet):
    lower_words = helpers.get_lowercase_tweet_words(tweet)
    keywords = ['carretera','trafico','tráfico','coche']
    return 'atasco' in lower_words and set(lower_words).intersection(set(keywords))


@match >> terminal
@watch 'all spanish tweets that are subjective':
    all_tweets = stream('Twitter').filter(helpers.is_tweet)
    spanish_tweets = all_tweets.filter(helpers.in_spanish)
    watch(spanish_tweets)(subjective_tweets)


@match >> slack/developers
@watch 'people in madrid talking about traffic':
    madrid_tweets = stream('TwitterMadrid').filter(helpers.is_tweet)
    watch(madrid_tweets)(about_traffic)

```

```python
import helpers

def has_more_than_two_emojis(tweet):
    return helpers.count_emoji(tweet['text']) > 2

def original_tweet_has_more_than_50_retweets(tweet):
    return tweet['retweeted_status']['retweet_count'] > 50

def hashtag_longer_than_twelve_letters_long(tweet):
    return all(len(h['text']) > 12 for h in tweet['entities']['hashtags'])


tweets = stream('twitter').filter(helpers.is_tweet)

@match >> trello/qa
@watch 'all retweeted tweets with more than 50 retweets':
    retweeted_tweets = tweets.filter(helpers.is_retweet)
    watch(retweeted_tweets)(original_tweet_has_more_than_50_retweets)


@match >> terminal
@watch 'all japanese tweets with more than 2 emojis':
    tweets_in_japanese = tweets.filter(helpers.is_japanese_tweet)
    watch(tweets_in_japanese)(has_more_than_two_emojis)


@match >> slack/theboss
@watch 'all spanish tweets with hashtags longer than 12 characters':
    spanish_hashtags = tweets.filter(helpers.is_spanish_tweet).filter(helpers.has_hashtags)
    watch(spanish_hashtags)(hashtag_longer_than_twelve_letters_long)
```
