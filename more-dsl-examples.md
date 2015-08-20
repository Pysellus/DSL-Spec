### Testing syntax:

```python
import helpers

def has_a_price(item):
    # your implementation here

def expires_after_tomorrow(item):
    # your implementation here

def has_enough_stock(item):
    # your implementation here


items = stream('my_api').filter(helpers.is_item)


@failure >> trello/qa
@check 'all items must have a price':
    expect(items)(has_a_price)


@failure >> terminal
@check 'all items expiring soon must expire after tomorrow':
    items_expiring_soon = items.filter(helpers.expires_soon)
    expect(items_expiring_soon)(expires_after_tomorrow)


@failure >> slack/alarm
@check 'there is enough stock of all items deliverable to Madrid':
    items_deliverable_to_Madrid = items.filter(helpers.deliverable_to_Madrid)
    expect(items_deliverable_to_Madrid)(has_enough_stock)
```

### Watching syntax

When the original syntax doesn't match with your use case, you can use this other one:

```python
import helpers

def has_more_than_two_emojis(tweet):
    return helpers.count_emoji(tweet['text']) > 2

def original_tweet_has_more_than_50_retweets(tweet):
    return tweet['retweeted_status']['retweet_count'] > 50

def hashtag_longer_than_twelve_letters_long(tweet):
    return all(len(h['text']) > 12 for h in tweet['entities']['hashtags'])


tweets = stream('twitter').filter(helpers.actually_is_tweet)

@match >> trello/qa
@watch 'all retweeted tweets with more than 50 retweets':
    retweeted_tweets = tweets.filter(helpers.is_retweet)
    watch(retweeted_tweets)(original_tweet_has_more_than_50_retweets)


@match >> terminal
@watch 'all japanese tweets with more than 2 emojis':
    tweets_in_japanese = tweets.filter(helpers.is_japanese_tweet)
    watch(tweets_in_japanese)(has_more_than_two_emojis)


@match >> slack/urgent
@watch 'all spanish tweets with hashtags longer than 12 characters':
    spanish_hashtags = tweets.filter(helpers.is_spanish_tweet).filter(helpers.has_hashtags)
    watch(spanish_hashtags)(hashtag_longer_than_twelve_letters_long)
```
