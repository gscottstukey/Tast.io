# [Tast.io](http://tast.io): Better Restaurant Recommendations via Word2Vec

### Kenneth Dooley, Zipfian Academy, 1/3/15 - 3/27/15

## Overview
This was my capstone project while attending Zipfian Academy and was motivated by the question: When in a new city, how do you find local restaurants most similar to a favorite restaurant in your hometown?  I used the tools and techniques of data science to measure similarities between restaurants using only Yelp review text and created a content-based recommendation engine, [Tast.io](http://tast.io) ('Tasty-Oh').  This app can be used to find restaurant recommendations in both New York City and San Francisco starting with most US restaurants as input (as long as they have more than 20 Yelp reviews).

##How to Use
In order to use the app, simply lookup a restaurant in the search bar using Google places autocomplete, choose which city you would like to see your results in (searches where the input and outputs are entirely within one city also work), and choose distance based results if desired and you live in either New York or San Francisco (must allow location tracking and not be using a vpn for this to work).  If the restaurant you are using as your search target is not located in either NY or SF, you will need to wait up to a minute after hitting the submit button while the engine scrapes your restaurant's reviews on Yelp.

## Dataset
My dataset consisted of Yelp restaurant reviews for all restaurants in New York City and San Francisco with more than 50 reviews as of March 2015. Using Yelp's [search API](https://www.yelp.com/developers/documentation/v2/search_api) I received metadata for each restaurant in both cities.  I then scraped the URLs for each restaurant in order to get the text of their reviews.  This review html data was then further parsed to find the text of each review.  These data collection steps took approximately two days to run using two separate AWS instances.

### Dataset Scope
The primary dataset for modeling consisted of all reviews for 11,000 restaurants across both New York and San Francisco with more than 50 reviews each. These were filtered from an even larger set of restaurants, however, which had any number of reviews; for each restaurant, I was able to scrape the full text for each review. The most recent reviews were used up to a limit of 500.

## Modeling Similarities
Each restaurant was first modeled as a 'document', which is a string created from the concatenation of the texts from their most recent (up to) 500 reviews.  I built a clean-tokenize-TFIDF-Word2Vec-Doc2Vec pipeline to create vectors for each restaurant from which cosine similarities could be calculated.  A few different methods of creating these vectors were tried before the final pipeline was chosen through a limited A/B testing framework.
<br><br>
1. Clean - Removed punctuation, symbols, and stopwords.  Performed stemming.  Found frequent word pairs to treat as bi-grams.
<br><br>
2. Tokenize - Converted the cleaned strings of words into a list of separate n-grams (1 and 2-grams, in this case)
<br><br>
3. TFIDF - Compiled a vocabulary of all tokens occuring more than once in the corpus which I then used to create a bag-of-words representation for each document.  This was then used to build a TFIDF weighted Document-Token feature matrix.
<br><br>
4. Word2Vec - Trained a Word2Vec skip-gram model using all sentences which occured throughout the entire restaurant document corpus.  Word2Vec yielded a dense 300-dimensional vector for each word in the vocabulary.  These word vectors can be thought of as encoding the semantic 'meaning' of each word.
<br><br>
5. Doc2Vec - Calculated restaurant vectors as the weighted average of the word vectors for all vocabulary words appearing in a given document.  The individual word vector weightings were based on the scaled TFIDF weights for each word in a document. 


### Similarity Modeling Details and Parameters
I originally tried three different methods of creating document vectors for each restaurant:
<br><br>
1) After creating the TFIDF matrix in step 3 above, I reduced its dimensionality from the original vocabulary of 215,000 words and phrases to 300 'topics' using a technique called Latent Semantic Indexing which is based on singular value decomposition, a common matrix factorization technique.  These new vectors were then used to represent each restaurant.
<br><br>
2) The Doc2Vec method described above
<br><br>
3) a combination where the vector from 1) was appended to the vector from 2) for each restaurant.
<br><br>
After performing blind tests of each model with several people knowledgable of the San Francisco and NYC retaurants scenes analyzing their output, the Doc2Vec methodology emerged as the winner.

## Launch
I pickled the matrix of cosine similarities between every pair of restaurant vectors in NY and SF along with lookup dictionaries containing metadata for each restaurant.  Using this similarity matrix, the model is able to quickly lookup the 20 most similar restaurants (in either of those two cities) to the target restaurant.  Restaurants outside of these cities take longer to lookup since their reviews are scraped, parsed and vectorized at run time.   


### Example Searches
One of the interesting results of using this app is that fine distinctions between various subtypes of cuisines can be searched for.  For example, "pizza" is a fairly broad category.  Searches for restaurants like 'Pizza Hut' will return restaurants which are quite similar like Dominos and Papa John's.  However, if we search for a more artisinal wood-fired pizza shop, the results will also skew towards other artisinal wood-fired pizza shops.  This will also work when searching for restaurants that serve thin NY style pizza versus those serving Chicago deep dish pizza.  Other interesting results occur when searching for restaurants with beautiful skyline views of a city.  The results skew towards other restaurants of similar cuisine and price which also offer dramatic views.

## Possible Next Steps
* Add additional cities.
* Increase the model's scope to include bars.
* Explore other topic modeling techniques such as [LDA](http://en.wikipedia.org/wiki/Latent_Dirichlet_allocation) to see if that would result in better restaurant vectors
* Use more robust NLP techniques (lemmatization) in pre-processing the full review texts.
* Explore using the formal Doc2Vec algorithm proposed by [Mikolov and Quoc](http://cs.stanford.edu/~quocle/paragraph_vector.pdf) to create the restaurant vectors.
* Allow the ability to tilt results towards or away from specific keyword vectors.

## Toolkit + Credits
1. [Yelp API](https://www.yelp.com/developers/documentation/v2/search_api) 
2. [MongoDB](http://www.mongodb.org/) - Chosen because my database operations involved more dumping documents in and pulling documents out than creating complex queries.
  * [pymongo](https://github.com/mongodb/mongo-python-driver) - A python driver for MongoDB
3. [BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/) - A python html-parsing library. It makes it much easier to pull out particular elements from a complex webpage.
4. [Requests](http://docs.python-requests.org/en/latest/) - A python library used in scraping tasks for getting webpage html.
4. [Gensim](https://radimrehurek.com/gensim/) - Provides models for a number of natural language processing techniques ... also supports [online machine learning](http://en.wikipedia.org/wiki/Online_machine_learning).
5. [Natural Language Toolkit](http://www.nltk.org/) - Provides support for natural language processing: stopwords lists, word tokenizers, and more!
6. [Flask](http://flask.pocoo.org/) - a python based framework for creating web applications.
7. [Google Place Autocomplete](https://developers.google.com/places/documentation/autocomplete)
8. [Mapbox](https://www.mapbox.com/)
9. [Leaflet](http://leafletjs.com/)
10. [SlickGrid](https://github.com/mleibman/SlickGrid)
11. [Zipfian Academy](http://www.zipfianacademy.com/) - The best Data Science bootcamp. 

## Glossary of Terms
* TFIDF - [Term Frequency - Inverse Document Frequency](http://en.wikipedia.org/wiki/Tf%E2%80%93idf)
* [Word2Vec](https://code.google.com/p/word2vec/)
* [Latent Semantic Indexing](http://en.wikipedia.org/wiki/Latent_semantic_indexing)
