---
title: 'Building a Twitter bot using Python and Github Actions'
date: 2023-08-28T01:01:01-07:00
draft: false
tags: [Twitter, Python, Github Actions]
---

A lot of my favourite content on Twitter (or, "X", I guess) these days comes from automated/"bot" accounts,  some of which are [oddly intriguing](https://twitter.com/Unsecured_CCTV). As such, I thought I'd try to set up a Twitter bot to see how easily it could be done. It turns out it's not too tricky as long as you aren't trying to do it in R, as I originally tried. There are many examples of setting up an R Twitter bot using the `rtweet` package in R; however, with Twitter upgrading its API from V1.1 to V2, it appears that `rtweet` no longer works. Hopefully the issues with `rtweet` will be resolved soon. In the meantime, the `tweepy` package in Python offers a simple means of interacting with the new Twitter API.

# Setting up a Twitter account

The first step is to sign up for a Twitter account. You could use an existing account but Twitter requires automated accounts to be labelled as such and you may not want an "automated" tag above all of your personal tweets.

With a basic account set up, we now need to set up a 
[developer account](https://developer.twitter.com/en/portal/dashboard). There are paid tiers for those who want to send a lot of tweets or read tweets, but the free tier (listed at the bottom) allows us to send 1,500 tweets per month, or roughly two tweets per hour.

![TwitterDevSignUp](/images/post6/twitter-dev-sign-up.png)

Once you've logged into the Developer Dashboard, you can create a new project and generate the keys you'll need to automate tweets via Python, in particular, the API Key, Secret API Key, Access Token, Secret Access Token, and Bearer Token.

# Setting up the Github repository

The bot should be hosted in a fresh Github respository. Hosting on Github is advantageous for two reasons. First, we can easily automate scripts in a Github repository to execute according to a schedule using Github Actions (see final section). Second, in certain cases, we may want to host a script publicly while keeping certain components secret. In this case, we  don't want to share our Twitter keys with everyone. Github has a page (Settings -> Secrets and variables -> Actions) where we can assign secret values to variables that can later be called by our scripts.

Click "New repository secret", enter a variable name, and post the key below. You'll need to do this for each of the five keys listed above.

![GithubSecrets](/images/post6/github-secrets.png)


# Python script for tweeting images

Before writing the Python bot I compiled a list of images and saved them to a [.csv file](https://github.com/stevenrossi/fishpoasting/blob/main/URLs.csv) that contained three columns: `img`, `desc`, and `source`, containing the image URL, description, and source, respectively. For this example, I'll be using images of fish and fishing from public archives. I won't go into how I scraped this information, though I may go over that in a later post.

The Python script for tweeting an image from an online source is fairly simple. First, the required packages and Twitter API credentials are loaded, then OAuth is used to authenticate.

```python
import os
import re
import tweepy
import requests
import pandas as pd
import numpy as np

# credentials to access Twitter API
API_KEY = os.environ.get("API_KEY")
API_KEY_SECRET = os.environ.get("API_KEY_SECRET")
BEARER_TOKEN = os.environ.get("BEARER_TOKEN")
ACCESS_TOKEN = os.environ.get("ACCESS_TOKEN")
ACCESS_TOKEN_SECRET = os.environ.get("ACCESS_TOKEN_SECRET")

# create an OAuthHandler instance
client = tweepy.Client(
    BEARER_TOKEN,
    API_KEY,
    API_KEY_SECRET,
    ACCESS_TOKEN,
    ACCESS_TOKEN_SECRET, 
)

# Twitter authentication
tweepy_auth = tweepy.OAuth1UserHandler(
    "{}".format(API_KEY),
    "{}".format(API_KEY_SECRET),
    "{}".format(ACCESS_TOKEN),
    "{}".format(ACCESS_TOKEN_SECRET),
)
tweepy_api = tweepy.API(tweepy_auth)
```

Next, we load the table of images, randomly pick one, and extract the URL, description, and source.

```python
# Read-in table of URLs
allImages = pd.read_csv('URLs.csv')

# Randomly select a row
i = np.random.randint(0,len(allImages),size=1)
z = allImages.iloc[i-1]

# Extract URL, description, source
fish_url = z.iloc[0,0]
fish_desc = z.iloc[0,1]
fish_source = z.iloc[0,2]
```

The next step is to download the image from the URL and upload it to Twitter.


```python
# Download image from URL
img_data = requests.get(fish_url).content
with open("fishpic.jpg", "wb") as handler:
    handler.write(img_data)

post = tweepy_api.simple_upload("fishpic.jpg")
media_id = re.search("media_id=(.+?),", str(post)).group(1)
```

We also need to construct some text to accompany the tweet. Basic tweets cannot exceed 280 characters, so we need to clip and text strings exceeding this limit.

```python
# Construct text from description and source
poast_text = str(fish_desc+'\n'+fish_source)

# Ensure 280 characters max
poast_text_clipped = (poast_text[:280]) if len(poast_text) > 280 else poast_text

```

Finally, some code to actually post the tweet.

```python
# Main function for creating tweet
def main():
    client.create_tweet(text=poast_text_clipped,
                        media_ids=["{}".format(media_id)])

if __name__ == "__main__":
    main()
```

The above Python code was saved to `poast-fish.py`. We can test the script by running `python poast-fish.py` at the command line and seeing if a tweet was successfully posted. If so, we can move onto automating the script.


# Automating with Github Actions

We can use [Github Actions](https://docs.github.com/en/actions) to remotely execute code at regular intervals - all we need to do is save a `YAML` file to the `.github/workflows/` directory of the repository containing [instructions for running the code](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions). The `YAML` file contains two key sections: an `on` section with instructions on when to trigger the workflow and a `jobs` section with instructions on how the workflow should be executed.

Workflows can be scheduled using [cron](https://en.wikipedia.org/wiki/Cron) - here I've specified that the workflow should run every hour on the hour with the argument `'0 * * * *'`. If you don't know how to write cron expressions, you can usually Google "cron" followed by what you want in plain English (e.g, cron every hour on the hour) and you'll get a link to [crontab guru](https://crontab.guru/every-1-hour) with your argument. 

Additionally, if I make a tweak to my bot and push the changes, I want to see the results right away without waiting for the scheduled tweet. To accommodate this, I've also added some lines that will execute the workflow whenever changes have been pushed to the `main` branch.

The job contains four steps. First, we checkout the repository content to a [Github Runner](https://github.com/actions/runner), then we install the version of Python that we need. Next, we install all the packages we need using a [requirements file](https://github.com/stevenrossi/fishpoasting/blob/main/requirements.txt), which ensures we are working with the correct package versions. In the final step, we use `env` to specify the variables that are available to all jobs in the workflow, then we execute the python script that contains our Twitter bot.

```YAML
on:
  schedule:
    - cron: '0 * * * *'
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:

      - name: checkout repo content
        uses: actions/checkout@v2 # checkout the repository content to github runner

      - name: setup python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11.4' # install the python version needed
          
      - name: install python packages
        run: |
          python -m pip install --upgrade pip
          pip3 install -r requirements.txt
          
      - name: execute py script # run poast-fish.py
        env:
          BEARER_TOKEN: ${{ secrets.BEARER_TOKEN }}
          API_KEY: ${{ secrets.API_KEY }}
          API_KEY_SECRET: ${{ secrets.API_KEY_SECRET }}
          ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
          ACCESS_TOKEN_SECRET: ${{ secrets.ACCESS_TOKEN_SECRET }}
        run: python poast-fish.py
```

After pushing this to the `main` branch on Github, the workflow should execute. We can track each execution on the "Actions" tab of the Github repository. If any run fails, we can inspect the output in this tab to see where our workflow failed.

My Twitter bot can be viewed [here](https://twitter.com/fishpoasting). The code is available on [Github](https://github.com/stevenrossi/fishpoasting).




