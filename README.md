# Political Classification via Sentiment Analysis

### Political Party Affiliation Classifications of Tweets and Headlines via Google’s ALBERT Natural Language Model

## Preparing Dataset
##### `twitter-scraper` and Gathering Tweets
`twitter-scraper` (forked from: [github repo](https://github.com/bisguzar/twitter-scraper) + modifications):
- scrapes tweets from specified twitter profiles
- scraped tweets are put into scrubbed_tweets directory (xlsx file) along with specified party affiliation
  - each scraped profile are put into own xlsx file (**not combined .tsv file fine-tuning requires**)

to run:
```
python3
>>> from twitter_scraper import get_tweets
>>> for tweet in get_tweets('realDonaldTrump', 1):
...      print(tweet)
...
```

`get_tweets()` takes arguments:
- twitter handle (required)
- party affiliation (required)
  - 0 for democrat, 1 for republican
- numTweets (optional; default value 300)
  - specifies number of tweets to be scrubbed

I chose to scrape tweets from prominent government officials listed below:

Democrats:
Barack Obama ([@BarackObama](https://twitter.com/BarackObama))
Joe Biden ([@JoeBiden](https://twitter.com/JoeBiden))
Nancy Pelosi ([@SpeakerPelosi](https://twitter.com/SpeakerPelosi))

and Republicans:
Ben Carson ([@realBenCarson](https://twitter.com/realBenCarson))
Donald Trump ([@realDonaldTrump](https://twitter.com/realDonaldTrump))
Scott Walker ([@ScottWalker](https://twitter.com/ScottWalker))

(Previous versions used data from: [Kaggle’s Democrat Vs. Republican Tweets](https://www.kaggle.com/kapastor/democratvsrepublicantweets))


#### Combining Tweet into Comprehensive Dataset for Fine-Tuning
As noted, `twitter-scraper`'s `get_tweets()` implements the functionality for scraping tweets from one twitter profile. As we will see below, fine-tuning requires one .tsv file with full fine-tuning corpus.
To do so:
1. Manually gather tweets from each file in `twitter_scraper\scrubbed_tweets` and copy into one .xlsx file
2. Use online converter to convert the resulting file of above .xlsx file to .tsv
  - I used [this one](https://products.groupdocs.app/conversion/xlsx-to-tsv)
3. Move .tsv file into `Albert-Sentiment-Analysis\data` and rename to `train.tsv`
  - `data` = name specified in `data_dir` in fine-tuning step


## Fine Tuning

`Albert-Sentiment-Anaylsis` forked from: [gaganmanku96's Albert for Sentiment Analysis](https://github.com/gaganmanku96/Albert-Sentiment-Analysis)

Provides fine tuning on pre-trained ALBERT model (`run_glue.py`) + functionality to perform
predictions (`api.py`)

To Fine-Tune:

```
python3 run_glue.py --data_dir data --model_type albert --model_name_or_path albert-large-v2 --output_dir output --task_name sst-2 --do_train
```
Required parameters:

- `data_dir`: Directory where data is stored
  - `train.tsv` resides in data directory.
  - `train.tsv` has two columns: ”Tweets” and ”Party” with 0’s and 1’s representing
    Democrats and Republicans, respectively (from Preparing Dataset step)
- `model_type`: Albert; Natural Language Model to use for fine-tuning
- `model_name_or_path`: Variant of Albert you want to use
  - Models range from base to xxlarge (along with different versions)
  - More information of models is available here or on the official [Google ALBERT repository](https://github.com/google-research/ALBERT)
- `output_dir`: Directory to store fine-tuned model (must be empty)
- `do_train`: Because we are training the model

## Predictions

1. Set name of folder where model files are stored. I.e. in initialization of SentimentAnalyzer
    class, set `path` equal to the path indicated in `output_dir` of fine-training command. In case
    above: `path='output'`.
2. Run `api.py` file:
    Set text on line 89 of `api.py` to the tweet to predict on and run `api.py` directly.

```
python3 api.py
```

OR via interactive shell. Shown in green with sample tweet from President Trump

```
python
>>> from api import SentimentAnalyzer
>>> classifier = SentimentAnalyzer ()
>>> print(classifier.predict(’Amazing how the Fake News never covers this. No Interest
on Student Loans. The Dems are just talk!’))
```

## Results

Results after fine tuning with cleaned `train.tsv` file of 5000 Democratic and Republic
tweets each.

Below are few examples of the tweets used for prediction...
<br/><br/><br/>
<!-- ![](https://github.com/nicholamerkel/albert_sentiment_analysis/blob/master/Albert-Sentiment-Analysis/images/trump%203:16%20fake%20news.png "trump fake news")
<br/><br/><br/>
![](https://github.com/nicholamerkel/albert_sentiment_analysis/blob/master/Albert-Sentiment-Analysis/images/trumo%203:10%20crazy%20bernie.png "trump crazy bernie")
<br/><br/><br/>
![](https://github.com/nicholamerkel/albert_sentiment_analysis/blob/master/Albert-Sentiment-Analysis/images/trump%203:10%20corona.png "trump corona")
<br/><br/><br/>
![](https://github.com/nicholamerkel/albert_sentiment_analysis/blob/master/Albert-Sentiment-Analysis/images/pelosi%203:11%20corona.png "pelosi corona")
<br/><br/><br/>
![](https://github.com/nicholamerkel/albert_sentiment_analysis/blob/master/Albert-Sentiment-Analysis/images/pelosi%202:27%20gun.png "pelosi gun") -->


### Actual Results

0 for Democrat, 1 for Republican...

Tweet | Actual | Predicted | Confidence | Correct?
----- | :----: | :-------: | :--------: | :------:
<!-- “Can’t believe they are not going after Schumer for the threats he made to our cherished United States Supreme Court, and our two great Justices. If a Republican did that, there would be an endless price to pay. Pathetic!” -@realDonaldTrump | 1 | 0 | 0.7085 | N
“Amazing how the Fake News never covers this. No Interest on Student Loans. The Dems are just talk!” -@realDonaldTrump | 1 | 1 | 0.70596 | Y
“I am fully prepared to use the full power of the Federal Government to deal with our current challenge of the CoronaVirus!" -@realDonaldTrump | 1 | 0 | 0.69494 | N
“Someone needs to tell the Democrats in Congress that CoronaVirus doesn’t care what party you are in. We need to protect ALL Americans!" -@realDonaldTrump | 1 | 0 | 0.54928 | N
“Our CoronaVirus Team has been doing a great job. Even Democrat governors have been VERY complimentary!" -@realDonaldTrump | 1 | 1 | 0.64473 | Y
“Going to be a BAD day for Crazy Bernie!" -@realDonaldTrump | 1 | 1 |  0.68758 | Y
“President @realDonaldTrump took decisive action to put the health of the American people first.“ -@Mike_Pence | 1 | 0 | 0.63431 | N
“It’s clear: the safety, security, and health of the American people remains President Trump’s top priority." -@GOP | 1 | 0 | 0.72733 | N
“@realDonaldTrump's Admin has taken unprecedented steps to protect the health of Americans. <br/> -They declared a public health emergency in January to bolster response efforts </br> -POTUS signed into law more than $8 billion to fund response efforts" -@GOP | 1 | 0 | 0.69607 | N
“Large employers and corporations must step up to the plate and offer paid sick leave and paid family & medical leave to their workers. Both now as we fight the #coronavirus and in the years to come. #COVIDー19" -@SpeakerPelosi | 0 | 0 | 0.72822 | Y
“We cannot fight coronavirus effectively unless everyone who needs to be tested knows they can get tested for free. We cannot slow an outbreak     when workers are stuck w/ the terrible choice of staying home to avoid spreading illness & the paycheck their family can’t afford to lose." -@SpeakerPelosi | 0 | 0 | 0.72171 | Y
“One year ago, the House passed bipartisan background checks legislation to #EndGunViolence. At 2 pm ET, House & Senate Democrats will come together to call on Leader McConnell to finally bring a vote on this effort to save lives." -@SpeakerPelosi | 0 | 0 | 0.73133 | Y -->

### Discussion of Results

Fine-tuned model seems to be better suited for accurately predicting Democrat rather than Republican tweets.
Need to look more into `train.tsv` and include tweet samples from more/better representation of Republican and Democratic leaders.

## Resources

* [Senitment Analysis using ALBERT](https://towardsdatascience.com/sentiment-analysis-using-albert-938eb)
* [Google’s ALBERT](https://github.com/google-research/ALBERT)
* [Twitter Scraper](https://github.com/bisguzar/twitter-scraper)
