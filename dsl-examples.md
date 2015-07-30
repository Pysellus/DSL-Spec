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
