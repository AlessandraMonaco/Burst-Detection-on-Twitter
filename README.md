# Burst Detection on Twitter

Twitter's ubiquity, rapidity, cross-platform accessibility and real time nature make it an important source for event detection. This is why having a system for Burst Detection is crucial to enhance situation awareness for decision making, expecially during crisis or natural disasters.

This project is inspired by the paper "Using Social Media to Enhance Emergency Situation Awareness" by J. Yin, A. Lampert, M. Cameron, B. Robinson and R. Power, that proposes a complex system to acquire knowledge for emergency response, based on 
- Burst Detection
- Text Classification
- Online Clustering
- Geotagging

In this repository we implement just the Burst Detection module on a stream of tweets that are not disaster-related.

## Methodology

### Data collection
75000 tweets were extracted using Twitter API and Tweepy library. The data were extracted in real time on Friday 22, May from h 15:00 to 24:00. For each tweet we scraped thw tweet id, the text and the hashtags list.Due to the limited number of API calls one can make using a basic and free developer account, (~900 calls every 15 minutes before your access is denied) we used an algorithm that extracts 2,500 tweets per run once every 15 minutes. Therefore, due to the presence of timeouts, the resulting dataset is not exactly a continuous feed of tweets (because some "stopping periods" were required), but a discrete time series (batches of 2500 tweets every 15 minutes) instead.

### Model building

#### What is a bursty feature?
