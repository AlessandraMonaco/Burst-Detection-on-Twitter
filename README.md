# Burst Detection on Twitter

Twitter's ubiquity, rapidity, cross-platform accessibility and real time nature make it an important source for event detection. This is why having a system for Burst Detection is crucial to enhance situation awareness for decision making, expecially during crisis or natural disasters.

This project is inspired by the paper *"Using Social Media to Enhance Emergency Situation Awareness" by J. Yin, A. Lampert, M. Cameron, B. Robinson and R. Power*, that proposes a complex system to acquire knowledge for emergency response, based on 
- Burst Detection
- Text Classification
- Online Clustering
- Geotagging

In this repository we implement just the Burst Detection module on a stream of tweets that are not disaster-related.

## Methodology

### Data collection
75000 tweets were extracted using Twitter API and Tweepy library. The data were extracted in real time on Friday 22, May from h 15:00 to 24:00. For each tweet we scraped thw tweet id, the text and the hashtags list.Due to the limited number of API calls one can make using a basic and free developer account, (~900 calls every 15 minutes before your access is denied) we used an algorithm that extracts 2,500 tweets per run once every 15 minutes. Therefore, due to the presence of timeouts, the resulting dataset is not exactly a continuous feed of tweets (because some "stopping periods" were required), but a discrete time series (batches of 2500 tweets every 15 minutes) instead.

### Data preprocessing
Tweets were preprocessed as follows:
- the following entities were removed : emojis, punctuation, symbols, numbers, multiple spaces, urls;
- hashtags were treated as terms (i.e. #thisHashtag becomes 'thisHashtag');
- lower casing;
- tokenization with nltk TweetTokenizer;
- english stop words removal;
- lemmatization (WordNetLemmatizer);
- stemming (SnowballStemmer);

### Train-test splitting
- Training set: 15-18 h (27.500 tweets)
- Test set: 18-24 h (47.500 tweets)

### Model building

#### What is a bursty feature?
A **burst** is defined as a word that suddenly occurs frequently in a time window. 

#### Implementation idea
We detected bursty features comparing the actual probability and the expected probability (probability in a random time window), for each feature, taking the paper's suggestion:
>  If the actual probability is noticeably higher than the expected probability of the word, it indicates that this word  exhibits an abnormal behavior in the considered time window, and we thus consider it as a bursty word in that time window.

#### Offline phase : building the Background model
The Background model is needed to produce a vocabulary and to initialize the probabilities. It uses just the training set.
The main steps are:
- Vectorization with a binary count vectorizer;
- Time window splitting with a certain frequency (this is a hyperparameter) : we tried frequencies of 30 sec, 10 min, 20 min,30 min, but for our dataset the best choice was 20 min, ending up with 9 time windows;
- Rescaling time windows with a scaling factor (this is another hyperparameter);
- Computing actual probabilities for each feature, for each time window;
- Computing expected probabilities for each feature

#### Online phase : Detection algorithm
This phase uses the Background model and the test set.
The test set was splitted into 16 windows, using the same window size of the offline phase. Two hyperparameters were empirically chosen:
- the **threshold** : if the difference between expected and actual probabilities is greater than the threshold, the feature is detected as bursty;
- the **sliding feature window size** for dynamic windowing : this is the time after which the background model is recomputed using a new vocabulary and new probabilities; we set it to 3 hours.

#### Evaluation
Since we do not have any labels in the dataset and therefore any ground truth, the evaluation is performed by comparing the results (bursty features) of the online algorithm with the ones detected with an offline algorithm, that considers as vocabulary the features of the entire dataset. This is known as **competitive analysis**.
We defined:
- **true positives** (TP) : bursty features detected by both the online and the offline algorithm;
- **true negatives** (TN) : (non-bursty) features detected by none of the algorithms;
- **false positives** (FP) : features detected as bursty by the online but not by the offline algorithm;
- **false negatives** (FN) : features detected as bursty by the offline but not by the online algorithm;

We used this definitions to compute the following evaluation metrics
- **False alarm rate** = FP / (#bursts detected online)
- **Detection rate** = TP / (#bursts detected offline)

### Results
These are just examples of the results we got:
![image](https://user-images.githubusercontent.com/68104089/108636791-e5382100-7487-11eb-895b-44fa33084934.png)
![image](https://user-images.githubusercontent.com/68104089/108636883-45c75e00-7488-11eb-917b-ba0837b87a28.png)
Basically the detected bursty features revealed the following events:
- **rainonme** and **arianagrande** were detected because on that day the song "Rain on me" by Ariana Grande and Lady Gaga was published on **applemusic**, in the **newmusicdaily** playlist;
- features like **trump**, **president**, **church** and **american** refear to a conference by Trump, helded in that day, where he stated that churches were essential and they should reopen;
- **noacf** is the acronymum of Notes On A Conditional Form, which is the fourth studio album by the english band "The 1975", and it was detected because it was released on that day through Dirty Hit and Polydor Records.

To conclude, we just show some offline plots to understand how a true positive, a false negative and a false positive look like:
![image](https://user-images.githubusercontent.com/68104089/108637201-eff3b580-7489-11eb-8ae7-c89155dd906e.png)
