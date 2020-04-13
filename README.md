# Tweet Classification via Sentiment Analysis

##### Political Party Classification of Tweets and Headlines via Google’s ALBERT Natural Language Model

## Preparing Dataset
#### Gathering Tweets via `twitter-scraper`
- scrapes tweets from specified twitter profiles (trimming links/images from tweets)
- scraped tweets are put into `results/twitter/scrubbed_tweets` directory (xlsx file)
  - each scraped profile are put into own xlsx file (**not combined .tsv file fine-tuning requires**)
  - *NOTE: neutral/irrelevant tweets are still admitted.<br>For accuracy of model, manually groom resulting xlsx files, keeping only tweets that reflect respective party affiliation.*


To Run:
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
- numTweets (optional; default value 500)
  - specifies number of tweets to be scrubbed


Below are the government officials' twitters chosen for fine-tuning:
- Barack Obama (D) ([@BarackObama](https://twitter.com/BarackObama))
- Joe Biden (D) ([@JoeBiden](https://twitter.com/JoeBiden))
- Nancy Pelosi (D) ([@SpeakerPelosi](https://twitter.com/SpeakerPelosi))
- Ben Carson (R) ([@realBenCarson](https://twitter.com/realBenCarson))
- Donald Trump (R) ([@realDonaldTrump](https://twitter.com/realDonaldTrump))
- Scott Walker (R) ([@ScottWalker](https://twitter.com/ScottWalker))

Tweets were scraped on 03/23/20 with the exception of Republicans Ben Carson and Scott Walker, whose tweets were scraped on 03/25/20.

---

#### Combining Tweets into Comprehensive Dataset for Fine-Tuning
*First, manually groom each xlsx file, keeping only the tweets that reflect given party affiliation.<br>I chose to keep first 100 party-relevant tweets for each xlsx file/chosen twitter account.*<br><br>
As noted, `twitter-scraper`'s `get_tweets()` implements the functionality for scraping tweets from one twitter profile. Fine-tuning requires one .tsv file of full corpus.<br>
To do so:
<ol>
<li>Manually gather tweets from each file in `twitter_scraper\scrubbed_tweets` and copy into one .xlsx file (after manually grooming)</li>
<li>Convert the resulting file of above .xlsx file to .tsv
  <ul><li>I used [this online converter](https://products.groupdocs.app/conversion/xlsx-to-tsv)</li></ol>
</li>
<li>Move .tsv file into `Albert-Sentiment-Analysis\data` and rename to `train.tsv`
  <ul><li>`data` = name specified in `data_dir` in fine-tuning step</li></ul>
</li></ol>

In given dataset (i.e. `data\train.tsv`), tweets correspond to:
| rows | govt. official |
| :---: | ----- |
| 2 - 101 | [@BarackObama](https://twitter.com/BarackObama) |
| 102 - 201 |[@JoeBiden](https://twitter.com/JoeBiden) |
| 202 - 301 | [@SpeakerPelosi](https://twitter.com/SpeakerPelosi) |
| 302 - 401 | [@realDonaldTrump](https://twitter.com/realDonaldTrump) |
| 402 - 501 | [@ScottWalker](https://twitter.com/ScottWalker) |
| 502-601 | [@realBenCarson](https://twitter.com/realBenCarson) |


## Fine Tuning
#### Fine-Tuning ALBERT Pre-Trained Model on Dataset via `Albert-Sentiment-Analysis`

Provides fine-tuning on pre-trained ALBERT model (`run_glue.py`) + functionality to perform
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
#### Predicting Democrat/Republican Affiliation for Tweets and Headlines via `Albert-Sentiment-Analysis`

1. Set name of folder where model files are stored. I.e. in initialization of SentimentAnalyzer
    class, set `path` equal to the path indicated in `output_dir` of fine-training command. In case
    above: `path='output'`.
2. Run `api.py` file:
    Set text on line 89 of `api.py` to the tweet to predict on and run `api.py` directly.

```
python3 api.py
```

OR via interactive shell. Shown in green with sample tweet from [@realDonaldTrump](https://twitter.com/realDonaldTrump/status/1239201055315025920)

```
python
>>> from api import SentimentAnalyzer
>>> classifier = SentimentAnalyzer ()
>>> print(classifier.predict(’Amazing how the Fake News never covers this. No Interest
on Student Loans. The Dems are just talk!’))
```

## Results
###### \*0 for Democrat, 1 for Republican\*

tweet | tweeter (link) | topic | actual | predicted | correct? | confidence
------- | :----:| :----: | :---: | :---: | :---: | :----:
Transgender people everywhere deserve to live in dignity and security. Together, we will end hatred and bigotry towards trans Americans and build a nation based on love, justice and civil rights. #TransDayOfVisibility | [@BernieSanders](https://twitter.com/BernieSanders/status/1245079639258812417)| lgbtq |0	| 0	| Y	| 0.8406
Americans deserve better than a health care system where people are terrified and need treatment, but are afraid to go to the doctor or emergency room because they cannot afford the bill. They deserve Medicare for All.	| [@BernieSanders](https://twitter.com/BernieSanders/status/1245029155089133569) | health care | 0 | 0 | Y | 0.88369								
We need to bail out workers, not corporations. | [@BernieSanders](https://twitter.com/BernieSanders/status/1241551152258387969) | corp. | 0 | 0 | Y | 0.84622		
Not surprisingly, the Republican plan for the coronavirus pandemic is totally inadequate. It benefits the rich and large corporations, creates desperation for poor and working families and goes nowhere near far enough to address the health and economic crises we're facing. | [@BernieSanders](https://twitter.com/BernieSanders/status/1241086398960226307)	| corp. | 0 | 	0 |	Y |	0.88926		
On day one we'll restore the DACA program for the 1.8 million young people who are eligible. We will we end ICE raids and family separations. We will have a humane, sensible immigration policy supported by the American people. #DemDebate |[@BernieSanders](https://twitter.com/BernieSanders/status/1239360515937271809) | immigration | 0 |	0 |	Y |	0.87774					
Money that goes to bail out big companies should come with serious long-term change: a $15 minimum wage, at least one seat for workers on their board of directors, & no union-busting. And CEOs should face civil & criminal penalties for violating these terms. |	[@SenWarren](https://twitter.com/SenWarren/status/1245084087704051713) | corp. | 0 |	0 |	Y |	0.88823		
This time around, by cancelling student debt payments for millions, we will fix the mistake that still holds back a generation of people and dragged down our economy, and create a real, grassroots stimulus to help see us through this crisis. | [@SenWarren](https://twitter.com/SenWarren/status/1240654722484375553) | student debt | 0 | 0 | Y |	0.87841		
It is disgusting that the Trump Admin is knowingly putting migrants in harm's way, at risk of being kidnapped, by continuing its #RemainInMexico policy. Let’s be clear: This policy makes no one safer. It is just furthering the Admin’s discriminatory, xenophobic agenda. |	[@SenatorMendez / RT @SenWarren](https://twitter.com/SenatorMenendez/status/1237856580466425856) |	immigration |	0 |	0	| Y |	0.72588				
President Trump & his Big Oil buddies want to allow offshore drilling in more than 90% of our coastal waters. We can’t put coastal communities at risk of another devastating oil spill. We need to build a clean energy future, protect families, & tackle the #ClimateCrisis head-on. |	[@SenWarren](https://twitter.com/SenWarren/status/1233776187547308035) | oil industry / climate |	0	| 0 | Y	| 0.88588			
Trump’s new acting intelligence director is an unqualified hack willing to risk our national security to sell out to foreign powers. My bill to #EndCorruptionNow  would ban Americans like Richard Grenell from getting paid to lobby for foreign govts. | [@SenWarren](https://twitter.com/SenWarren/status/1232745435166584837) | corruption / foreign govts. | 0 | 0 | Y | 0.88869
I mean, that's fine, but I'm not sure what Biden has to offer beyond vague statements and botched basement interviews	| [@BenShapiro](https://twitter.com/benshapiro/status/1245448978373603328) | biden | 1 | 1 | Y | 0.83639			
Feminists need to stop making the Coronavirus a gender issue like Helen Lewis in her article for @TheAtlantic. Men are dying at higher rates than women, and we are all just trying to survive. | [@classicallyabby / RT @BenShapiro](https://twitter.com/classicallyabby/status/1245442162340368384)	| feminism | 1 | 0 | N | 0.75995
\#COVIDー19 kills more men than women. But today’s gender activists are so addicted to the “women-are-victims” narrative, nothing can open their hearts &minds to human reality. Gender ideology is a powedul drug. It robs true believers of reason & mercy. Read this thread & weep. |	[@CHSommers / RT @BenShapiro](https://twitter.com/CHSommers/status/1245397912378642442)	| feminism	| 1 |	1 |	Y |	0.7881			
Life is a right. In fact, unlike abortion, which is not a right, life is specified directly in the Declaration of Independence, as well as the Fifth and Fourteenth Amendments, and remains the most fundamental right of all. #AbortionIsAWomansRight	| [@BenShapiro](https://twitter.com/benshapiro/status/1129038284162834432) | abortion |	1 |	1 |	Y |	0.73817													
"GOP: Okay, we'll pass this insanely large spending bill because we have to stop the imminent collapse of the American economy.<br>Democrats: Sounds good.<br>Pelosi: I WANT WINDMILLS AND CORPORATE DIVERSITY QUOTAS<br>Democrats: WE WANT WINDMILLS AND CORPORATE DIVERSITY QUOTAS" |[ @BenShapiro](https://twitter.com/benshapiro/status/1242152881022623744)	| covid |	1 |	0 |	N |	0.86141			
Republicans are working to help the American people. Democrats are politicizing this pandemic to advance a radical agenda.	| [@NewtGingrich](https://twitter.com/newtgingrich/status/1242535079810826246) | covid	| 1 | 0 |	N |	0.88494
"“It’s strange out here. It’s like what the US would be like in 2021 If Bernie was in charge.” @ChrisPlanteShow"	| [@NewtGingrich](https://twitter.com/newtgingrich/status/1239908092403474433)	| bernie	| 1 |	1	| Y |	0.83383													
Biden in last night’s debate rushed to match Sanders on the left. In the middle of a pandemic he would deport no one. He favors tax paid abortion. Police should not report criminal aliens. He would shut down not just fracking but the oil industry.Insanity disguised as a campaign.	| [@NewtGingrich](https://twitter.com/newtgingrich/status/1239447132559597568) | biden / immigration | 1 | 0 | N | 0.86458
Every day this [impeachment trial] goes on, the Democrats look smaller, more political, more destructive of America |	[@NewtGingrich](https://twitter.com/newtgingrich/status/1222928712083132417) | impeachment | 1 | 1 | Y | 0.67411				
Want to kill a robust economy? Then, raise taxes on companies and the middle class. We know this is what Democrats would do, because they have all said it.	| [@NewtGingrich](https://twitter.com/newtgingrich/status/1221865454962003968) | taxes / corp. | 1 | 0 | N | 0.88977


and headlines...
headline | news org. (link) | topic | actual | predicted | correct? | confidence
------- | :----:| :----: | :---: | :---: | :---: | :----:
FDA Relaxes ‘Blood Ban’ For Gay Men, But LGBTQ Advocates Want Bigger Change |	[@HuffPost](https://www.huffpost.com/entry/blood-donations-relaxed-some-gay-bisexual-men_n_5e862395c5b6a94918331371?ncid=tweetlnkushpmg00000067) | lgbtq | 0 | 0 | Y |	0.72859			
Trump Announces CDC Mask Guidelines But Says He Probably Won’t Follow Them | [@HuffPost](https://www.huffpost.com/entry/trump-mask-cdc-guidelines-voluntary_n_5e87b11cc5b6cc1e47753b7e) | trump	| 0 |	1 | N |	0.83786					
Legal Sex Workers And Others In Adult Industry Denied Coronavirus Aid	| [@HuffPost](https://www.huffpost.com/entry/legal-sex-workers-denied-coronavirus-aid_n_5e86287ac5b6d302366ca912) |	sex workers / covid / health care | 0 |	0	| Y	| 0.52759								
Mississippi Governor Declares ‘Confederate Heritage Month’ During Coronavirus Pandemic | [@HuffPost](https://www.huffpost.com/entry/tate-reeves-confederate-mississippi_n_5e8b3d5cc5b6cbaf282cf2e3) |	confederacy | 0	| 1 | N | 0.78153					
U.S. Rapidly Deporting Hundreds Of Migrant Children Under New Coronavirus Rules |	[@HuffPost](https://www.huffpost.com/entry/us-deport-migrant-children-coronavirus-bill_n_5e8d8e46c5b6e1d10a6c30dc) | immigration	| 0	| 0	| Y	| 0.68523								
Abortion Bans Would Force Women To Travel Hundreds More Miles For Care	| [@HuffPost](https://www.huffpost.com/entry/abortion-bans-travel-coronavirus_n_5e8634b3c5b6a949183355e0) |	abortion 	| 0	| 0	| Y	| 0.80104
Abortion Stops a Beating Heart — Unless It Is Being Harvested for Research |	[@amspectator](https://spectator.org/abortion-stops-a-beating-heart-unless-it-is-being-harvested-for-research/)	| abortion	| 1	| 1	| Y	| 0.80441								
The Democrats Go Full Anti Life	 | [@amspectator](https://spectator.org/the-democrats-go-full-anti-life/) |	abortion 	| 1	| 1	| Y	| 0.59681								
‘No Show Joe’ Makes Trump Look Presidential |	[@amspectator](https://spectator.org/no-show-joe-makes-trump-look-presidential/)	| biden	| 1	| 1	| Y	| 0.83361								
National Security in the Time of Pandemic	| [@amspectator](https://spectator.org/national-security-in-the-time-of-pandemic/)	| national security |	1 |	1	| Y |	0.51622								
The Impeachment That Killed Americans |	[@amspectator](https://spectator.org/the-impeachment-that-killed-americans/) | impeachment | 1	| 1	| Y	| 0.54325								
A Day at the Coronavirus Supermarket That Communist Bernie Would Have Loved	| [@amspectator](https://spectator.org/a-day-at-the-coronavirus-supermarket-that-communist-bernie-would-have-loved/)| bernie | 1	| 1	| Y	| 0.83705																



<!-- ### Discussion of Results -->

## Resources
- [Sentiment Analysis using ALBERT](https://github.com/gaganmanku96/Albert-Sentiment-Analysis))
- [Google’s ALBERT](https://github.com/google-research/ALBERT)
- [Twitter Scraper](https://github.com/bisguzar/twitter-scraper)
