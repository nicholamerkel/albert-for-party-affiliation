# Tweet Classification via Sentiment Analysis

### Political Party Classification of Tweets via Google’s ALBERT Natural Language Model

##### via [gaganmanku96's Albert for Sentiment Analysis](https://github.com/gaganmanku96/Albert-Sentiment-Analysis)

## Preparing Dataset
##### `twitter-scraper` and Gathering Tweets
`twitter-scraper` (cloned from: [github repo](https://github.com/bisguzar/twitter-scraper) + modifications):
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

Below are the chosen government officials' twitter

Democrats:
Andrew Cuomo ([@NYGovCuomo](https://twitter.com/NYGovCuomo))
Barack Obama ([@BarackObama](https://twitter.com/BarackObama))
Bernie Sanders ([@SenSanders](https://twitter.com/SenSanders))
Chuck Schumer ([@SenSchumer](https://twitter.com/SenSchumer))
Joe Biden ([@JoeBiden](https://twitter.com/JoeBiden))
Nancy Pelosi ([@SpeakerPelosi](https://twitter.com/SpeakerPelosi))


and Republicans:
Ben Carson ([@realBenCarson](https://twitter.com/realBenCarson))
Donald Trump ([@realDonaldTrump](https://twitter.com/realDonaldTrump))
Marco Rubio ([@MarcoRubio](https://twitter.com/MarcoRubio))
Mike Pence ([@Mike_Pence](https://twitter.com/Mike_Pence))
Mitt Romney ([@MittRomney](https://twitter.com/MittRomney))
Scott Walker ([@ScottWalker](https://twitter.com/ScottWalker))

Tweets were scraped on 03/23/20 with the exception of Republicans Ben Carson and Scott Walker, whose tweets were scraped on 03/25/20.

(Previously used data from: [Kaggle’s Democrat Vs. Republican Tweets](https://www.kaggle.com/kapastor/democratvsrepublicantweets))


#### Combining Tweet into Comprehensive Dataset for Fine-Tuning
As noted, `twitter-scraper`'s `get_tweets()` implements the functionality for scraping tweets from one twitter profile. As we will see below, fine-tuning requires one .tsv file with full fine-tuning corpus.
To do so:
1. Manually gather tweets from each file in `twitter_scraper\scrubbed_tweets` and copy into one .xlsx file
2. Use online converter to convert the resulting file of above .xlsx file to .tsv
  - I used [this one](https://products.groupdocs.app/conversion/xlsx-to-tsv)
3. Move .tsv file into `Albert-Sentiment-Analysis\data` and rename to `train.tsv`
  - `data` = name specified in `data_dir` in fine-tuning step


## Fine Tuning

Cloned from: [gaganmanku96's Albert for Sentiment Analysis](https://github.com/gaganmanku96/Albert-Sentiment-Analysis)

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
![](https://github.com/nicholamerkel/albert_sentiment_analysis/blob/master/Albert-Sentiment-Analysis/images/trump%203:16%20fake%20news.png "trump fake news")
<br/><br/><br/>
![](https://github.com/nicholamerkel/albert_sentiment_analysis/blob/master/Albert-Sentiment-Analysis/images/trumo%203:10%20crazy%20bernie.png "trump crazy bernie")
<br/><br/><br/>
![](https://github.com/nicholamerkel/albert_sentiment_analysis/blob/master/Albert-Sentiment-Analysis/images/trump%203:10%20corona.png "trump corona")
<br/><br/><br/>
![](https://github.com/nicholamerkel/albert_sentiment_analysis/blob/master/Albert-Sentiment-Analysis/images/pelosi%203:11%20corona.png "pelosi corona")
<br/><br/><br/>
![](https://github.com/nicholamerkel/albert_sentiment_analysis/blob/master/Albert-Sentiment-Analysis/images/pelosi%202:27%20gun.png "pelosi gun")


### Actual Results
###### 0 for Democrat, 1 for Republican

tweet | tweeter (link) | topic | actual | predicted | correct? | confidence
------- | :----:| :----: | :---: | :---: | :---: | :----:
Transgender people everywhere deserve to live in dignity and security. Together, we will end hatred and bigotry towards trans Americans and build a nation based on love, justice and civil rights. #TransDayOfVisibility | [@BernieSanders](https://twitter.com/BernieSanders/status/1245079639258812417)| lgbtq |0	| 0	| Y	| 0.8406
Americans deserve better than a health care system where people are terrified and need treatment, but are afraid to go to the doctor or emergency room because they cannot afford the bill. They deserve Medicare for All.	| [@BernieSanders](https://twitter.com/BernieSanders/status/1245029155089133569)	| health care | 0 | 0 | Y | 0.88369								
We need to bail out workers, not corporations. | [@BernieSanders](https://twitter.com/BernieSanders/status/1241551152258387969) | corp. | 0 | 0 | Y | 0.84622		
Not surprisingly, the Republican plan for the coronavirus pandemic is totally inadequate. It benefits the rich and large corporations, creates desperation for poor and working families and goes nowhere near far enough to address the health and economic crises we're facing. | [@BernieSanders](https://twitter.com/BernieSanders/status/1241086398960226307)	| corp. | 0 | 	0 |	Y |	0.88926		
On day one we'll restore the DACA program for the 1.8 million young people who are eligible. We will we end ICE raids and family separations. We will have a humane, sensible immigration policy supported by the American people. #DemDebate |[@BernieSanders](https://twitter.com/BernieSanders/status/1239360515937271809) | immigration | 0 |	0 |	Y |	0.87774					
Money that goes to bail out big companies should come with serious long-term change: a $15 minimum wage, at least one seat for workers on their board of directors, & no union-busting. And CEOs should face civil & criminal penalties for violating these terms. |	[@SenWarren](https://twitter.com/SenWarren/status/1245084087704051713) | corp. | 0 |	0 |	Y |	0.88823		
This time around, by cancelling student debt payments for millions, we will fix the mistake that still holds back a generation of people and dragged down our economy, and create a real, grassroots stimulus to help see us through this crisis. | [@SenWarren](https://twitter.com/SenWarren/status/1240654722484375553) | student debt | 0 | 0 | Y |	0.87841		
It is disgusting that the Trump Admin is knowingly putting migrants in harm's way, at risk of being kidnapped, by continuing its #RemainInMexico policy. Let’s be clear: This policy makes no one safer. It is just furthering the Admin’s discriminatory, xenophobic agenda. |	[@SenatorMendez / RT @SenWarren](https://twitter.com/SenatorMenendez/status/1237856580466425856) |	immigration |	0 |	0	| Y |	0.72588				
President Trump & his Big Oil buddies want to allow offshore drilling in more than 90% of our coastal waters. We can’t put coastal communities at risk of another devastating oil spill. We need to build a clean energy future, protect families, & tackle the #ClimateCrisis head-on. |	[@SenWarren](https://twitter.com/SenWarren/status/1233776187547308035) | oil industry / climate |	0	| 0 | Y	| 0.88588			
Trump’s new acting intelligence director is an unqualified hack willing to risk our national security to sell out to foreign powers. My bill to #EndCorruptionNow  would ban Americans like Richard Grenell from getting paid to lobby for foreign govts. | [@SenWarren](https://twitter.com/SenWarren/status/1232745435166584837) | corruption / foreign govts. | 0 | 0 | Y | 0.88869
I mean, that's fine, but I'm not sure what Biden has to offer beyond vague statements and botched basement interviews	| [@BenShapiro](https://twitter.com/benshapiro/status/1245448978373603328) | biden | 1 | 1 | Y | 0.83639			
Feminists need to stop making the Coronavirus a gender issue like Helen Lewis in her article for
@TheAtlantic. Men are dying at higher rates than women, and we are all just trying to survive. | [@classicallyabby / RT @BenShapiro](https://twitter.com/classicallyabby/status/1245442162340368384)	| feminism | 1 | 0 | N | 0.75995
'#COVIDー19 kills more men than women. But today’s gender activists are so addicted to the “women-are-victims” narrative, nothing can open their hearts &minds to human reality. Gender ideology is a powedul drug. It robs true believers of reason & mercy. Read this thread & weep. https://twitter.com/heatherbarr1/status/1238056470777868294 |	[@CHSommers / RT @BenShapiro](https://twitter.com/CHSommers/status/1245397912378642442)	| feminism	| 1 |	1 |	Y |	0.7881			
Life is a right. In fact, unlike abortion, which is not a right, life is specified directly in the Declaration of Independence, as well as the Fifth and Fourteenth Amendments, and remains the most fundamental right of all. #AbortionIsAWomansRight	| [@BenShapiro](https://twitter.com/benshapiro/status/1129038284162834432) | abortion |	1 |	1 |	Y |	0.73817													
"GOP: Okay, we'll pass this insanely large spending bill because we have to stop the imminent collapse of the American economy.
Democrats: Sounds good.\n Pelosi: I WANT WINDMILLS AND CORPORATE DIVERSITY QUOTAS\nDemocrats: WE WANT WINDMILLS AND CORPORATE DIVERSITY QUOTAS" |[ @BenShapiro](https://twitter.com/benshapiro/status/1242152881022623744)	| covid |	1 |	0 |	N |	0.86141			
Republicans are working to help the American people. Democrats are politicizing this pandemic to advance a radical agenda.	| [@NewtGingrich](https://twitter.com/newtgingrich/status/1242535079810826246) | covid	| 1 | 0 |	N |	0.88494
"“It’s strange out here. It’s like what the US would be like in 2021 If Bernie was in charge.”
@ChrisPlanteShow"	| [@NewtGingrich](https://twitter.com/newtgingrich/status/1239908092403474433)	| bernie	| 1 |	1	| Y |	0.83383													



### Discussion of Results

Fine-tuned model seems to be better suited for accurately predicting Democrat rather than Republican tweets.
Need to look more into `train.tsv` and include tweet samples from more/better representation of Republican and Democratic leaders.

## Resources

* [Senitment Analysis using ALBERT](https://towardsdatascience.com/sentiment-analysis-using-albert-938eb)
* [Google’s ALBERT](https://github.com/google-research/ALBERT)
* [Twitter Scraper](https://github.com/bisguzar/twitter-scraper)
