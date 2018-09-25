---
layout: post
title:  "Simple Twitter Sentiment with Elasticsearch"
author: "Dayton"
---


```python
# !pip install textblob
# !pip install elasticsearch
```


```python
from tweepy import Stream
from tweepy import OAuthHandler
from tweepy import StreamListener
import json
from textblob import TextBlob
from elasticsearch import Elasticsearch
```


```python
# Elasticsearch running on localhost:9200
es = Elasticsearch()
```


```python
consumer_key = "your-consumer-key"
consumer_secret = "your-consumer-secret"

access_token = "access-token"
access_token_secret = "access-token-secret"

auth = OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
```


```python
class listener(StreamListener):
    
    def on_data(self, data):
        dict_data = json.loads(data)
        tweet = TextBlob(dict_data['text'])
        sen = tweet.sentiment.polarity
        
        if sen < 0:
            sentiment = 'negative'
        elif sen == 0:
            sentiment = 'neutral'
        else:
            sentiment = 'positve'
        
        es.index(
            index = 'nz-sentiment',
            doc_type = 'test-type',
            body = {
                'author': dict_data['user']['screen_name'],
                'date': dict_data['created_at'],
                'message': dict_data['text'],
                'polarity': sen,
                'subjectivity': tweet.sentiment.subjectivity,
                'sentiment': sentiment
            }
        )
        return True
    
    def on_error(self, status):
        print(status)
```


```python
# Start streaming, no output
stream = Stream(auth, listener())
stream.filter(track=['new zealand'], async=True)
```


```python
# Run cell to stop streaming
stream.disconnect()
```
