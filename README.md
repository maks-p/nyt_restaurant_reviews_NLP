# NYT Restaurant Reviews
Mod 4 NLP Partner Project <br/>
By: Sam Jackson & Maks Pazuniak

## Project Goals
In this project, we explore the capabilities of NLP to analyze New York Times restaurant reviews and attempt to predict the star rating for the subject restaurant of a given article.

## Data
We scraped the NYT [restaurant review search site](https://www.nytimes.com/reviews/dining) for review URLs, restaurant names, star rating, neighborhood, etc. <br/>

We then found & scraped [this site](https://www.nytimes.com/column/restaurant-review), as the list was more comprehensive, including reviews for restaurants that had been reviewed multiple times. <br/>

We scraped each review URL for restaurant review, headline, and any features that we hadn't retrieved in our initial scrape.

**Original Dataset:** 986 Reviews <br/>
**Functional Dataset:** 713 Reviews <br/>
**Reviewed By: Pete Wells, Frank Bruni, Sam Sifton:** 615 Reviews

**Features:**
- Reviewer: Author of article
- Vocabulary: # of unique tokens per article (after removing stopwords and lemmatizing)
- Area: Area of NYC where restaurant is located
- blob_avg_pol: Average polarity of sentences in the article, based on TextBlob
- blob_std_pol: Standard Deviation of polarity of sentences in the article, based on TextBlob
- headline_sent: Sentiment of Headline, based on TextBlob
- TF-IDF of the most polar words (>abs(+/-3))
- Reviewer x Average review polarity interaction
- Reviewer x Vocabulary interaction

**Target:**
- Star Rating: Number of stars given to a restaurant by a reviewer. <br/>
*Note: We aggregated reviews labeled 'Poor', 'Satisfactory', or 'Good' as 0 stars.  We also grouped together reviews with 3 or 4 stars into one category (labeled 3 stars), as there were only a handful of 4 star reviews.*

## Baseline Models
We used TF-IDF and a Random Forest Classifier to come up with a baseline model.  Our baseline had an test-accuracy of .4805.

## EDA
### Reviewers
We elected to look at only the top three reviewers, based on the reviews we scraped.  These three reviewers produced over 80% of the reviews in our "functional" dataset.  

![stars](https://user-images.githubusercontent.com/42282874/58827390-eb7dce00-8610-11e9-99d7-f8177cb5e22d.png)

**Pete Wells:**
- 304 Reviews
- 09/2009 - Present
- 2 Star Rating percentage: 48%
- Average Star Rating: 1.71 Stars

**Frank Bruni:**
- 218 Reviews
- 06/2004 - 08/2009
- 2 Star Rating percentage: 33%
- Average Star Rating: 1.51 Stars
 
**Sam Sifton:**
- 93 Reviews
- 10/2009 - 10/2011
- 2 Star Rating percentage: 35%
- Average Star Rating: 1.50 Stars 

![Reviewers](/png_files/RatingsByReviewer.png)

### Sentiment Analysis 
We tested several methods for assessing the polarity of the articles. VADER, since it was trained on & built for Twitter-style text (short document length, caps and punctuation to boost sentiment, emojis, etc), didn't detect the sentiment of a NYT article as well as some other lexicons, such as TextBlob and Afinn.  We tested models using both TextBlob and Afinn sentiment analysis.  Below, you can see that Afinn is slightly more consistent than TextBlob with expectations - median sentiment increases with star rating. 

![Polarity](/png_files/polarity_scoring.png)

With all models, we evaluated sentiment on a sentence-by-sentence basis, utilizing the average of each review for our final score.

#### Sample VADER Sentiment Output 
```
Hanon, a new udon shop in Williamsburg, Brooklyn, was produced by the union of a Tokyo video-production company and a Japanese manufacturer of unusually thin condoms.
{'neg': 0.0, 'neu': 1.0, 'pos': 0.0, 'compound': 0.0}

The condoms became the subject of a series of advertisements on which the production company worked; in one of them, called “Acts of Love,” dancers in London re-enact, with surprising grace and dignity, the mating rituals of blue-footed boobies, fiddler crabs and other animals.
{'neg': 0.0, 'neu': 0.84, 'pos': 0.16, 'compound': 0.765}

Well, kids, when two companies like each other very much, sometimes they decide to create a new company together.
{'neg': 0.0, 'neu': 0.691, 'pos': 0.309, 'compound': 0.6908}

That is what happened with the production firm and the prophylactics people when, for reasons that are perhaps best not to question, they hit upon the idea of expanding their product line from condoms into noodles.
{'neg': 0.0, 'neu': 0.893, 'pos': 0.107, 'compound': 0.6369}

The restaurant lies across Union Avenue from Kellogg’s Diner.
{'neg': 0.259, 'neu': 0.741, 'pos': 0.0, 'compound': -0.4215}

Its door is marked during business hours by the fluttering white noren curtains.
{'neg': 0.0, 'neu': 1.0, 'pos': 0.0, 'compound': 0.0}

It is the second Hanon location.
{'neg': 0.0, 'neu': 1.0, 'pos': 0.0, 'compound': 0.0}

The first is about 6,700 miles away, in the city of Kamakura, which lies south of Tokyo and is known for soba, not udon.
{'neg': 0.109, 'neu': 0.891, 'pos': 0.0, 'compound': -0.4215}

This gave Hanon the advantage of not competing against any of Japan’s established udon styles, leaving its chef, Takahiro Yanagisawa, free to come up with his own.
{'neg': 0.0, 'neu': 0.825, 'pos': 0.175, 'compound': 0.6486}

Mr. Yanagisawa, who had spent 25 years making sushi before turning to noodles, focused his innovative urges on the dough.
{'neg': 0.0, 'neu': 0.766, 'pos': 0.234, 'compound': 0.6705}

He began by adding wheat germ and bran to the white flour that in udon is typically used alone.
{'neg': 0.1, 'neu': 0.9, 'pos': 0.0, 'compound': -0.25}

The resulting noodle is called zenryufun or whole wheat.
{'neg': 0.0, 'neu': 1.0, 'pos': 0.0, 'compound': 0.0}

The bran and germ are Mr. Yanagisawa’s attempts to make a more healthful udon, but they also add flavor, a mottled color and a slightly rough texture that holds on to the dashi-based broth.
{'neg': 0.0, 'neu': 1.0, 'pos': 0.0, 'compound': 0.0}
```
#### Sample Afinn Sentiment Output
```
Hanon, a new udon shop in Williamsburg, Brooklyn, was produced by the union of a Tokyo video-production company and a Japanese manufacturer of unusually thin condoms.
  Afinn score: 0.0

The condoms became the subject of a series of advertisements on which the production company worked; in one of them, called “Acts of Love,” dancers in London re-enact, with surprising grace and dignity, the mating rituals of blue-footed boobies, fiddler crabs and other animals.
  Afinn score: 6.0

Well, kids, when two companies like each other very much, sometimes they decide to create a new company together.
  Afinn score: 2.0

That is what happened with the production firm and the prophylactics people when, for reasons that are perhaps best not to question, they hit upon the idea of expanding their product line from condoms into noodles.
  Afinn score: 3.0

The restaurant lies across Union Avenue from Kellogg’s Diner.
  Afinn score: 0.0

Its door is marked during business hours by the fluttering white noren curtains.
  Afinn score: 0.0

It is the second Hanon location.
  Afinn score: 0.0

The first is about 6,700 miles away, in the city of Kamakura, which lies south of Tokyo and is known for soba, not udon.
  Afinn score: 0.0

This gave Hanon the advantage of not competing against any of Japan’s established udon styles, leaving its chef, Takahiro Yanagisawa, free to come up with his own.
  Afinn score: 3.0

Mr. Yanagisawa, who had spent 25 years making sushi before turning to noodles, focused his innovative urges on the dough.
  Afinn score: 4.0

He began by adding wheat germ and bran to the white flour that in udon is typically used alone.
  Afinn score: -2.0

The resulting noodle is called zenryufun or whole wheat.
  Afinn score: 0.0

The bran and germ are Mr. Yanagisawa’s attempts to make a more healthful udon, but they also add flavor, a mottled color and a slightly rough texture that holds on to the dashi-based broth.
  Afinn score: 0.0
```

### Icorporating TF-IDF
After running our sentiment analyses, we thought that finding polar words (or n-grams) within reviews may have some predictive value within our model.  We ran TF-IDF on the corpus of reviews, ran sentiment analysis on each n-gram produced by the TF-IDF Vectorizer, and used TF-IDF scores from the most polar (>|+/-3|) n-grams as features in our model. TF-IDF scores indicated how *important* these polar words were within each article, based on how many times they appeared in the article vs how many times they appeared within the whole corpus.  Below is a list of the most polar n-grams:

![PolarWords](/png_files/PolarWords.png)

## Best Model
Our best model predicted the star rating of our test-set of reviews with an accuracy of .536. <br/>
The best model was a Random Forest Classifier with TextBlob sentiment analysis.

![best model](/png_files/best_model.png)

Our biggest issue while modeling was the class imbalance among star ratings.  Attmpting to correct the class imbalance with over-sampling led to even worse results.  Under-sampling would have led to too few data points.  

![imbalance](/png_files/ClassImbalance.png)

## Going Forward
We have so much more work we'd like to do on this project! The power of NLP is immense and we would have loved more time to dive deeper.  
- Cosine Similarity to find semantic similarity between articles (and maybe, therefore, star-rating)
- Topic Modeling: do certain cuisines, dishes, ambiance, decor, location, etc impact the star-rating
- More work/implementation of most important words as features:

<img width="739" alt="Screen Shot 2019-06-03 at 3 30 23 PM" src="https://user-images.githubusercontent.com/42282874/58828936-8deb8080-8614-11e9-86a5-c72150a780d8.png">

