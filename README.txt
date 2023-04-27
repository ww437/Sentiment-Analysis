DSCI 511 - Term Project - Nintendo Switch Sentiment Analysis

Team Members:
    Kelsey Muckelbauer
    Michael Fenton
    Will Wu
    Ishtiaq Shahriar

Repository:
	https://github.com/MF80/DSCI511-TermProject

Files:
	README
	DSCI511 Project.ipynb
	data.zip
	data\B084DDDNRP_FULL.json
	data\B07VJRZ62R_FULL.json
	data\B07VGRJDFY_FULL.json
	Presentation

RapidAPI query limit:
	Our account for Rapid API allows for 3500 requests.  Through development and testing, we have used about 1600 of them.  You should have enough queries to re-populate the whole dataset but that is rather time consuming.  The steps on how to do this is detailed below in 'To fetch new data'.

RapidAPI subscription expiration:
	The provided RapidAPI key/subscription will expire on or around June 16, 2022.

RapidAPI service issues:
	During our final passes while preparing our presentation, RapidAPI server started wrongfully returning errors that the provided ASINs are invalid.  This is not accurate and retrying the service later may yield previously expected results. 

How to run .ipnyb
	Load Google Colab environment:
	If running in the Google Colab environment set loadGoogleColabEnv = True else set loadGoogleColabEnv = False.
	This will prevent runtime errors if running in an environment without the Google Drive.


	Map data directory:
	There is a nbdir variable that is the file path to the data directory.  If your local path is different, you can change it here.
	

	To keep existing data:
	The data directory is preloaded with all reviews for the three variants of the Nintendo Switch system.  Each variant's reviews are stored in a separate file.  Running the notebook as-is will check if the review files already exist.  It will then read reviews from the files then parse and process them accordingly.

	This will yield the most conclusive results (because it's the full data set) and should closely match the statistics provided in the report.

	To fetch new data:
	To (re)fetch data, simply delete (or rename) the existing files in the data directory and re-run the notebook.  The current configuration will only pull 20 pages of reviews per console variant for the sake of time.  You can increase or decrease the number of requests per product by changing the variables first_page_number and last_page_number.  Each review page contains up to 10 individual reviews.

	Our process limits the number of fetches up to last known page of reviews per product so blank requests are not queried.  If you want to pull the full review set, set last_page_number >= 325 (the high review page of the three products).

	When fetching data "Fetching page number: #" will be printed for every fifth request to provide feed back but limit spamming.


	Reloading full data set if deleted:
	The full data set provided is also stored in data.zip to quickly restore the original files if needed.


	Looking at processed data:
	The processed data for the training set stored in a dictionaries of counters called training_data_set
	The data for testing is kept in a list called test_data_set.

	Print statements are within the code to view the structures.


Code sections:
	Section 1:
	This section of code queries RapidAPI for reviews of three different Nintendo Switch products.  Reviews for each product are downloaded and compiled into a .json file for that product.  The following in an example of the data structure for a review.
		{
	      "id": "R1GC2P7OQBVZ2H",
	      "asin": {
	        "original": "B07VJRZ62R",
	        "variant": "B07VJRZ62R"
	      },
	      "review_data": "Reviewed in the United States on April 3, 2020",
	      "date": {
	        "date": "April 3, 2020",
	        "unix": 1585879200
	      },
	      "name": "DNA B",
	      "rating": 1,
	      "title": "Good Console at $299.Junk at This Price Gouge",
	      "review": "I like the Switch Console and even with its quirks, at $299 it's fairly priced and hopefully will be again when the demand goes down.It's nowhere nearly as powerful as what Playstation and Xbox offer but it's so much more flexible that it can get away with a $300 price tag.$450 though?No, not worth it at all.",
	      "verified_purchase": false
	    }
	
	If a compiled review .json file already exists in the data directory, the process for that product is skipped.  Other error handling is used to a) not exceed the known max review page number for a product, b) not record results from bad or timed out requests, and c) break the request loop if we start to receive blank review pages (signifying the end of reviews).


	Section 2:
	This section reads and processes the saved .json files from the data directory.  For the products designated to the training data set, each review is categorized based on the associated star rating: good (4-5 stars), neutral (3 stars), bad (1-2 stars).  Each word in the title text and review text, is checked against a common word list (comprised of the 100 most used English words plus some other common words).  If the word is not a common word it is stored into counters for good, neutral, or bad categorizes.  Titles and reviews are stored separately so we can later process them individually.

	The data for the test set does not need to be parsed into a counter like the training data.  Instead, we are going to analyze the reviews: rating, title, review text, later in the process.


	Section 3:
	The third part of the process is to run some Naives Bayes algorithms against our data to check if our model and/or data are sufficient to accurately predict sentiment for this limited case.

	We first try a multi-classification analysis using positive, negative, and mediocre (neutral) sentiments.  For each review in the test data set, we tokenize the words and compare them to the model data.  If the word appears in a bad review, the word's occurence ratio (log of the number of times the word is counted / the number of total words) is added to an aggregate sum of bad likelihood.  The same is done for neutral and positive reviews.  The log of the prior (percentage of overall good, bad, or neutrals reviews) is also factored into the likelihood sum.  We then compare the three calculated likelihoods where the largest score (most likely) outcome is our estimated sentiment.  

	Since we know the ratings for the data in the test set, we then determine if our model produced a true positive, false positive, true neutral, or false neutral result.  Statistical results are calculated for the full test set (precision, recall, f-measure, and accuracy).

	We repeat the same process but instead remove the neutral classification and only classify positive or negative sentiments.


	Results:
	The results show that while the two-classification process works well, it is more difficult to predict sentiment when neutral reviews are considered.

	Multi-classification results
		Precision: 0.46741963509991313
		Recall: 0.46741963509991313
		F-measure: 0.46741963509991313
		Accuracy: 0.6449464233999421

	Two-classification results
		Precision: 0.9024621212121212
		Recall: 0.9587525150905433
		F-measure: 0.9297560975609755
		Accuracy: 0.8784810126582279

Data Dictionary:
	Amazon request data:

	ID: 				Amazon ID of customer leaving the review
	ASIN: 				Amazon Standard Identification Number: A unique ID for each product or product version listed on Amazon.
	ASIN-ORIGINAL:		ASIN of the product that current review is originally for.
	ASIN-variant:		ASIN of other product variants that current review wasn't written for but could be applied to
	REVIEW_DATA:		Location and date of review
	DATE-DATE:			Date of review
	DATE-UNIX:			Date of review using UNIX date scheme
	NAME:				Name of the reviewer
	RATING:				Star rating, from 1 (lowest) to 5 (highest), of the product given by the reviewer
	TITLE:				Title of the review provided by the reviewer
	REVIEW:				Written review text from the reviewer for this product.
	VERIFIED_PURCHASE:	Whether the reviewer has purchased product (per Amazon).

Challenges and limitations:
	We initially tried to access reviews directly through Amazon's APIs.  To access their APIs, one must create an AWS account then have it verified and approved for free API access.  The approval process was prolonged or offline and we were never able to fully activate our account for API access.  We opted to use RapidAPI instead enough though it is a paid service.  The ease of registration and use was worth the low tier subscription cost for our current scope of this project.

	We also intended to pull reviews for the latest Playstation and Xbox consoles but, due to limited availability and price fixing, neither console is for sales through Amazon and, therefore, not listed for reviews.  We decided to change our scope from using three different gaming console system to simply using one system and different variants of it.  This would also provide more cohesion in the review language and therefore more reliable results: comparing apples to apples.

	A new challenge which appeared while finalizing code and presentation, RapidAPIs environment started to report the error: "Can't find product reviews. Please make sure that you provide a correct ASIN and country(if not US)."  The ASINs are correct in the notebook and we are attributing this to service issues from RapidAPI.

	Limitations within the program can come from our limited number of total reviews for building the training set and testing against.  Between the three versions of Nintendo Switch available on Amazon, though there are over 115,000 ratings, there are only about 6,000 written reviews.  Writing a review for a product takes more time and effort compared to only choosing a star rating; we expected this but not such a large disparity in the numbers.  To resolve this issue, other product types could be picked but the same trend is likely.

	Other limitations come from using the RapidAPI interface.  This is a subscription based service where the lowest, cheapest, tier only allows for 3500 queries a month.  Though this is enough requests for our project, rescoping to larger review sets may require higher tiered subscriptions.

	An algorthimic challenge we ran into was that our multi-classification algorithm struggles to identify good, bad, or neutral reviews where the two-classification method works well.  We believe that it's easier to differentiate language from two ends of the spectrum but, by incorporating neutral language, it blurs the hard lines of extreme sentiments.  Neutral reviews will likely have more overlap to good and bad reviews than simply good vs bad reviews would alone.  

Distribution and continued worked:
	This project will be available on GitHub at https://github.com/MF80/DSCI511-TermProject for any future work considered.  The API key will be removed from the source code though and those continuing the work would have to obtain one through RapidAPI.

	Future work can include: larger/different data sets, work identifying words with the highest variance (those with the highest relative weights), analyzing the difference between reviews from verified purchasers and non-verified purchasers or reviews from different regions.  Other work can address better analysis of neutral data and the classification of neutral sentiments.


