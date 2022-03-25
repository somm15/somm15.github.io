---
layout: single
title:  "Sonos alarm playing the last episode of a podcast"
date:   2022-03-25 21:51:16 +0100
categories: Tech
---

I just wanted to wake up every morning with the last episode of a podcast. I didn't find any solution, so I wrote my own scripts.

## Python code
This is a quick and dirty code to redirect GET requests to the last episode (last_podcast.py)
{% highlight ruby %}
import podcastparser
import urllib.request
import pprint
from flask import Flask,redirect

app = Flask(__name__)

@app.route("/")
def last_podcast():
    feedurl = 'http://radiofrance-podcast.net/podcast09/rss_10009.xml'
    parsed = podcastparser.parse(feedurl, urllib.request.urlopen(feedurl))
    url = parsed['episodes'][0]['enclosures'][0]['url']
    pprint.pprint(url)
    return redirect(url, code=302)

if __name__ == "__main__":
    app.run(host='0.0.0.0')
{% endhighlight %}

## Docker
The Dockerfile:
{% highlight txt %}
FROM python:3

WORKDIR /app

COPY requirements.txt ./
COPY last_podcast.py ./
RUN pip install --no-cache-dir -r requirements.txt

ENTRYPOINT [ "python" ]

CMD [ "last_podcast.py" ]

{% endhighlight %}

And the docker-compose.yml
The Docker file:
{% highlight yaml %}
version: "3.4"
services:
  last_podcast:
    build: .
    restart: unless-stopped
    user: 1001:1001
    ports:
      - 5203:5000
    restart: unless-stopped

{% endhighlight %}

## Run
Just copy the python file, the Dockerfile and the docker compose file in the same folder.
{% highlight bash %}
sudo docker-compose build .
sudo docker-compose up -d
{% endhighlight %}

## Sonos
Now you can just add http://<hostnale>:>5203 as a radio station using the sonos app. 
You can configure an alarm to use this radio station.

Quick & dirty, but it work!