# Denver Airbnb Data Analysis Project
This is a practicum project for the MSDS program at Regis University. The original datasets were obtained from http://insideairbnb.com/.

![Denver city skyline photo by Erick Todd from Pexels](https://github.com/tsgruman/regis-practicum-denver-airbnb/blob/assets/pexels-erick-todd-6034694.jpg)

*Denver city skyline photo by Erick Todd from Pexels*

# Introduction
When it comes to vacationing, Airbnb has become synonymous with hotels. The company that was founded in 2007 in San Francisco by two people who opened their home to guests has grown to over 7 million listings in over 220 cities worldwide (https://news.airbnb.com/about-us/). The service offers short-term and long-term rentals, ranging from private rooms to sprawling homes. 

However, not all Airbnb listings are created equally. This project will analyze Denver Airbnb listings to determine what makes an ideal Airbnb listing for Denver visitors. This will be accomplished through sentiment analysis and cluster analysis. First, sentiment analysis using TextBlob will be applied to review comments left by visitors. Second, various clustering techniques will be conducted to cluster Denver listings. The goal is to identify clusters with high polarity values and determine which features contribute to those values.

# Tools
This project was completed using Jupyter Notebook and Python. 

Libraries used include:
* pandas - data manipulation
* matplotlib - plotting
* numpy - arrays
* seaborn - plotting
* datetime - date and time manipulation
* nltk - Natural Language Processing
* string - string datatype manipulation
* wordcloud - generating wordclouds
* textblob
* scipy - mathematical functions
  * scipy.cluster.hierarchy
  * scipy.cluster.hierarchy import dendrogram
* sklearn - variety of machine learning tools
  * preprocessing
  * sklearn.preprocessing import StandardScaler
  * sklearn.cluster import KMeans
  * sklearn.cluster import DBSCAN
  * sklearn.cluster import AgglomerativeClustering
  * sklearn.metrics import silhouette_score
  * sklearn.decomposition import PCA
  * sklearn.neighbors import NearestNeighbors
  * sklearn.feature_selection import SelectKBest
  * sklearn.feature_selection import f_classif

# Data
The datasets were downloaded from InsideAirbnb.com. The Denver listings.csv and reviews.csv files last updated in December 2020 were used. 

## Listings Dataset
The listings.csv file is originally comprised of 3500+ rows and 74 features. After data cleaning to remove NA values, drop columns that were not pertinent to the project, and convert data to the correct datatypes, the cleaned file consisted of 2078 rows and 32 features. 

Features consist of:
* id
* name
* description
* host_id
* host_name
* host_since
* host_location
* host_response_time
* host_response_rate
* host_is_superhost
* host_neighbourhood
* host_listings_count
* host_verifications
* host_has_profile_pic
* host_identity_verified
* latitude
* longitude
* property_type
* room_type
* accommodates
* bathrooms
* bedrooms
* beds
* amenities
* price
* minimum_nights_avg_ntm
* maximum_nights_avg_ntm
* number_of_reviews
* review_scores_rating
* instant_bookable
* reviews_per_month

For cluster analysis and following sentiment analysis, listings are further reduced to 17 columns, which includes the addition of the newly created polarity and subjectivity columns from sentiment analysis.

## Reviews Dataset
The reviews.csv file originally consists of 183,442 rows and 6 columns. Very little data cleaning is needed for this file until the sentiment analysis segment, as I only need the comments column. However, after removing rows with null values, the data is reduced to 183,316 rows.

Features consist of:
* listing_id
* id
* date
* reviewer_id
* reviewer_name
* comments

# Sentiment Analysis
## Data cleaning
This consisted of removing punctuation and using the nltk 'words' library to remove non-English words from the comments. Removing these features reduces the change of affecting the sentiment analysis scores.

![image](https://user-images.githubusercontent.com/43609221/109817653-6497c400-7bef-11eb-81b2-f3e7c861aa70.png)

*Progression of cleaning comments and resulting comments_final column*

## TextBlob
[TextBlob](https://textblob.readthedocs.io/en/dev/) is a powerful and simple-to-use library for text processing. For this project, I used TextBlob for sentiment analysis to calculate polarity and subjectivity scores of comments. 

* Polarity = quantification of positive, neutral, or negative sentiment, range from -1 (negative) to 1 (positive)
* Subjectivity = quantification of degree of opinion vs. objective facts, range from 0 (objective) to 1 (subjective)

First, I defined the function to detect polarity using TextBlob to the cleaned comments column. Then, I added the polarity score to the dataset.

```ruby
def detect_pol(comments_final):
    return TextBlob(comments_final).sentiment.polarity
    
reviews_clean['polarity'] = reviews_clean.comments_final.apply(detect_pol)
reviews_clean[['listing_id', 'comments_final', 'polarity']].head(5)
```
![image](https://user-images.githubusercontent.com/43609221/109818782-9b220e80-7bf0-11eb-9d86-ab91f732e56b.png)

*Resulting output with polarity score added.*

I applied the same to subjectivity, defining the function and then adding the subjectivity value to the dataset.

```ruby
def detect_sub(comments_final):
    return TextBlob(comments_final).sentiment.subjectivity

reviews_clean['subjectivity'] = reviews_clean.comments_final.apply(detect_sub)
reviews_clean[['listing_id', 'comments_final', 'polarity', 'subjectivity']].head(5)
```
![image](https://user-images.githubusercontent.com/43609221/109819685-901bae00-7bf1-11eb-9b88-ab0b1cc8d087.png)

*Resulting output with subjectivity score added.*

To gauge the accuracy of the scoring, I printed top comments with the highest and lowest polarity and subjectivity scores.

Sentiment | Lowest | Highest
-------- | -------- | -------
Polarity | ![image](https://user-images.githubusercontent.com/43609221/109820586-6020da80-7bf2-11eb-9981-3802da2d909d.png) | ![image](https://user-images.githubusercontent.com/43609221/109820609-6747e880-7bf2-11eb-9efc-8e6d9df9e567.png)
Subjectivity | ![image](https://user-images.githubusercontent.com/43609221/109820743-847cb700-7bf2-11eb-8fc2-34f50d8aaa1d.png) | ![image](https://user-images.githubusercontent.com/43609221/109820780-8d6d8880-7bf2-11eb-9581-07191d9e26ce.png)

## Join to Listings Data
The goal of the project is to analyze sentiment data with listings details, so I joined the average of the sentiment scores per listing back to the listings dataset.

```ruby
group_reviews = reviews_clean.groupby("listing_id")
mean_reviews = group_reviews.mean()
mean_reviews = mean_reviews.reset_index()
mean_reviews = mean_reviews.drop(['id', 'reviewer_id'], axis=1)
mean_reviews.head(10)
```
![image](https://user-images.githubusercontent.com/43609221/109821189-f5bc6a00-7bf2-11eb-9c5e-e37f4bab7581.png)

*Grouped mean for each listing sentiment values.*

```ruby
listings_sentiment = pd.merge(listings_clean, mean_reviews, left_on='id', right_on='listing_id', how='inner')
listings_sentiment[['id', 'name', 'price', 'polarity', 'subjectivity']].head(5)
```
![image](https://user-images.githubusercontent.com/43609221/109821527-4b911200-7bf3-11eb-82b4-a55bf39fe404.png)

*Merged grouped mean sentiment values to listings table.*

## Results
Analyzing the distribution of sentiment values, it's clear to see most comments left for listings skew positivie polarity and high subjectivity.

![image](https://user-images.githubusercontent.com/43609221/109823325-0d94ed80-7bf5-11eb-947f-dc28228146b3.png) | ![image](https://user-images.githubusercontent.com/43609221/109823511-3cab5f00-7bf5-11eb-8b73-2729874df4f5.png)
---- | ----

Additionally, plotting polarity against a couple features reveals a positive correlation with number of bedrooms and price.
![image](https://user-images.githubusercontent.com/43609221/109823962-a1ff5000-7bf5-11eb-9c65-d0d41ad18af3.png) | ![image](https://user-images.githubusercontent.com/43609221/109824116-c0fde200-7bf5-11eb-9ea7-0fcb65131a3b.png)
----|----

Taking a look at the new dataset's correlation matrix, I can get a general sense of how polarity and subjectivity are correlated with every other feature.

![image](https://user-images.githubusercontent.com/43609221/109875322-8f087200-7c2d-11eb-8b52-e7d064121732.png)

In line with the previously plotted graphs, polarity has a slightly positive correlation with accommodates, bathrooms, bedrooms, bed, and price and a strong positive correlation with overall ratings. There is a slight negative correlation with hosting listings count (the number of listings a host has), latitude, and longitude.

# Cluster Analysis
## Data
The second part of the project began with loading the final listings dataset with sentiment values and removing any non-numerical values. After subsetting only numerical columns, I was left with 2077 rows and 17 columns. I standardized and normalized the data with the intent of applying clustering to the original dataset, standardized data, and normalized data and comparing the results.

* listings_num = original dataset
* standard_cluster = standardized dataset
* normal_cluster = normalized dataset
* cluster_sub = subset data based on feature selection; contains minimum average nights, number of reviews, review scores rating, reviews per month, and polarity

I also performed feature selection using ExtraTreesClassifier and SelectKBest to subset additional data. Again, I compare the models for the full dataset and selected features to compare performance.

There are many clustering techniques available depending on the dataset and goals of clustering. I decided to employ three clustering methods and compare the results: K-means, DBSCAN, and agglomerative.

## K-means Clustering
First, I needed to find the optimal value for k clusters. I employed the Elbow method using the KMeans from the sklearn library to plot for k.

```ruby
distortions = []
K = range(1,10)
for k in K:
    kmeanModel = KMeans(n_clusters=k)
    kmeanModel.fit(listings_num)
    distortions.append(kmeanModel.inertia_)
    
plt.figure(figsize=(10,5))
plt.plot(K, distortions, 'bx-')
plt.xlabel('k')
plt.ylabel('Distortion')
plt.title('Elbow Method for Optimal k')
plt.show()
```
![image](https://user-images.githubusercontent.com/43609221/109827509-07a10b80-7bf9-11eb-866c-794ff04b7241.png)

*Resulting plot for Elbow Method*

Repeating this process for the standardized and normalized datasets, I get the following optimal k clusters:
* Original data: k = 3
* Standardized data: k = 6
* Normalized data: k = 4

After finding k for each dataset, I applied kmeans to the data. I then concatenated the labels back to each dataset to plot labels against various factors.

```ruby
#apply KMeans
kmeans = KMeans(n_clusters = 3).fit(listings_num)
y_means = kmeans.predict(listings_num)

#add labels to original data
labels = pd.DataFrame(kmeans.labels_)

labelListings = pd.concat((listings_num,labels),axis=1)
labelListings = labelListings.rename({0:'labels'}, axis=1)

#print selected columns
labelListings[['id', 'bathrooms', 'bedrooms', 'price', 'polarity', 'labels']].head(10)
```
![image](https://user-images.githubusercontent.com/43609221/109859993-d5a0a100-7c1a-11eb-8207-7d022201b671.png)

*Output of cluster labels added to dataset.*

A pair plot for the entire dataset is difficult to read, so I instead plotted strip plots to compare categorical cluster labels against each column. This gives me an idea of clustering patterns. This [Kaggle project](https://www.kaggle.com/ellecf/visualizing-multidimensional-clusters) was a great resource for plotting these.

```ruby
f, axes = plt.subplots(4, 5, figsize=(20,25), sharex=False)
f.subplots_adjust(hspace=0.2, wspace=0.7)

for i in range(0,len(list(labelListings))-1):
    col = labelListings.columns[i]
    if i < 5:
        ax = sns.stripplot(x=labelListings['labels'], y=labelListings[col].values, 
        jitter=True, ax=axes[0,(i)])
        ax.set_title(col)
    elif i >= 5 and i < 10:
        ax = sns.stripplot(x=labelListings['labels'],y=labelListings[col].values, 
        jitter=True, ax=axes[1,(i-10)])
        ax.set_title(col)
    elif i >= 10 and i < 15:
        ax = sns.stripplot(x=labelListings['labels'], y=labelListings[col].values, 
        jitter=True, ax=axes[2, (i-10)])
        ax.set_title(col)
    elif i >= 15:
        ax = sns.stripplot(x=labelListings['labels'], y = labelListings[col].values, 
        jitter=True, ax=axes[3, (i-15)])
        ax.set_title(col)
```
![image](https://user-images.githubusercontent.com/43609221/109860713-b3f3e980-7c1b-11eb-93a5-2e3f10c82c0d.png)

The full dataset is still a lot for the eyes to read through, so I concatenated the kmeans labels to the subset data (dataset with feature-selected columns only). This resulted in a much easier-to-read output for the variables that feature selection identified as most important.

```ruby
labelListings_sub = pd.concat((cluster_sub,labels),axis=1)
labelListings_sub = labelListings_sub.rename({0:'labels'}, axis=1)

f, axes = plt.subplots(2,3, figsize=(20,20), sharex=False)
f.subplots_adjust(hspace=0.2, wspace=0.7)

for i in range(0,len(list(labelListings_sub))-1):
    col = labelListings_sub.columns[i]
    if i < 3:
        ax = sns.stripplot(x=labelListings_sub['labels'], y=labelListings_sub[col].values, 
        jitter=True, ax=axes[0,(i)])
        ax.set_title(col)
    elif i >= 3:
        ax = sns.stripplot(x=labelListings_sub['labels'],y=labelListings_sub[col].values, 
        jitter=True, ax=axes[1,(i-3)])
        ax.set_title(col)
```
![image](https://user-images.githubusercontent.com/43609221/109860994-0cc38200-7c1c-11eb-885c-247111a9f350.png)

I repeated this with the standardized and normalized data and received the following results:

Dataset | Full Strip Plot | Subset Strip Plot
--- | --- | ---
Original | ![image](https://user-images.githubusercontent.com/43609221/109861445-88bdca00-7c1c-11eb-8710-8648ee65fa9e.png) | ![image](https://user-images.githubusercontent.com/43609221/109861466-8e1b1480-7c1c-11eb-83f7-be1ee5c11536.png)
Standardized | ![image](https://user-images.githubusercontent.com/43609221/109861498-96734f80-7c1c-11eb-92aa-c38181e930b8.png) | ![image](https://user-images.githubusercontent.com/43609221/109861510-9bd09a00-7c1c-11eb-957f-ba2c9b480415.png)
Normalized | ![image](https://user-images.githubusercontent.com/43609221/109861564-ab4fe300-7c1c-11eb-91a2-662e03e48747.png) | ![image](https://user-images.githubusercontent.com/43609221/109861585-b145c400-7c1c-11eb-81ca-ca193fea71bd.png)

## DBSCAN
The second clustering technique I used was DBSCAN (Density-Based Spatial Clustering of Applications with Noise), which is a well-known unsupervised learning method. As its name suggests, it works well for high density data. I used the standardized data for DBSCAN.

The first step was to identify optimal epsilon and minimum samples values for the model. These values are defined by [Maklin (2019)](https://towardsdatascience.com/machine-learning-clustering-dbscan-determine-the-optimal-value-for-epsilon-eps-python-example-3100091cfbc) as:
* epsilon: "Two points are considered neighbors if the distance between the two points is below the threshold epsilon." This is notated as eps in sklearn library.
* minimum samples: "The minimum number of neighbors a given point should have in order to be classified as a core point. It’s important to note that the point itself is included in the minimum number of samples." This is notated as min_samples in sklearn library.

To find optimal values for eps and min_samples, I used the NearestNeighbors function from the sklearn library to calculate the distance between neighbors and plot the results. Similar to the Elbow method, I'm looking to identity the maximum curvature in the plot as my epsilon value.

```ruby
neighbors = NearestNeighbors(n_neighbors=30)
neighbors_fit = neighbors.fit(standard_cluster)
distances, indices = neighbors_fit.kneighbors(standard_cluster)

distances = np.sort(distances, axis=0)
distances = distances[:,1]
plt.plot(distances)
plt.title("Point Distances")
```
![image](https://user-images.githubusercontent.com/43609221/109866741-e48b5180-7c22-11eb-88fa-7f3e44880797.png)

*The max curve seems to be between 2 and 4.* 

With a range of optimal eps values, I iterated dbscan with values between 1 and 7 to see how many clusters each would give me. I then iterated various values for min_samples in case the default value of 5 could be improved.

```ruby
for eps in [1, 3, 5, 7]:
    print("\neps={}".format(eps))
    min_samples=5
    dbscan = DBSCAN(eps=eps, min_samples=min_samples)
    labels=dbscan.fit_predict(standard_cluster)
    print("min_samples= {}".format(min_samples))
    print("Number of clusters: {}".format(len(np.unique(labels))))
    print("Cluster sizes: {}".format(np.bincount(labels + 1)))
```
Output:

![image](https://user-images.githubusercontent.com/43609221/109866922-26b49300-7c23-11eb-8501-9104d8cc660b.png)

```ruby
for min_samples in [1, 3, 5, 7, 9]:
    print("\nmin_samples={}".format(min_samples))
    eps=3
    dbscan2 = DBSCAN(min_samples=min_samples, eps=eps)
    labels=dbscan2.fit_predict(standard_cluster)
    print("eps: {}".format(eps))
    print("Number of clusters: {}".format(len(np.unique(labels))))
    print("Cluster sizes: {}".format(np.bincount(labels + 1)))
```
Output:

![image](https://user-images.githubusercontent.com/43609221/109867093-5794c800-7c23-11eb-83a4-da93971b0dd6.png)

I seemed to get as balanced of clusters as I will get from eps=3 and min_samples=5, so I used these values for the final DBSCAN model.

```ruby
dbscan_cluster = DBSCAN(eps=3, min_samples=5).fit(standard_cluster)
```
Then I concatenated the resulting cluster labels back to the original dataset and the subset to create strip plots.

Full Strip Plot | Subset Strip Plot
--- | ---
![image](https://user-images.githubusercontent.com/43609221/109867579-e570b300-7c23-11eb-882b-9941f4ec8fb4.png) | ![image](https://user-images.githubusercontent.com/43609221/109867613-ef92b180-7c23-11eb-919d-948f9c24b6c1.png)

## Agglomerative Clustering
The final clustering method I used is agglomerative with the standardized dataset. I begin by plotting a dendrogram to see how many clusters I should expect.

```ruby
plt.figure(figsize=(20, 8))
plt.title('Hierarchy Dendrogram')
dend = sch.dendrogram((sch.linkage(standard_cluster, method='ward')))
```
![image](https://user-images.githubusercontent.com/43609221/109869170-c5da8a00-7c25-11eb-904e-93befb97f5a6.png)

Though this isn't pretty, it's clear there are 2 distinct clusters so I apply k=2 to the AgglomerativeClustering function on the data and, again, concatenate that back to the original dataset. Finally, I plot the final strip plots with the agglomerative cluster labels.

```ruby
agg = AgglomerativeClustering(n_clusters=2).fit(standard_cluster)
```

Full Strip Plot | Subset Strip Plot
--- | ---
![image](https://user-images.githubusercontent.com/43609221/109869501-2c5fa800-7c26-11eb-98c4-f4e75dad1f7b.png) | ![image](https://user-images.githubusercontent.com/43609221/109869513-32558900-7c26-11eb-89e2-ad359da304d5.png)

## Evaluate Models: Silhouette Scores
To evaluate my models' performances, I calculate the silhouette scores with the sklearn.metrics library. 

```ruby
print('kmeans: {}'.format(silhouette_score(listings_num, kmeans.labels_)))
print('Normalized kmeans: {}'.format(silhouette_score(normal_cluster, kmeans_normal.labels_)))
print('Standardized kmeans: {}'.format(silhouette_score(standard_cluster, kmeans_standard.labels_)))
print('DBSCAN: {}'.format(silhouette_score(standard_cluster, dbscan_cluster.labels_)))
print('Agglomerative: {}'.format(silhouette_score(standard_cluster, agg.labels_)))
```
Output:

![image](https://user-images.githubusercontent.com/43609221/109870530-72693b80-7c27-11eb-87db-6466321587de.png)

Of the 5 models I created, the normalized k-means model performed the best. The k-means model on the original dataset (not standardized or normalized) came in at a close second, while DBSCAN was the third best performing model, but with a low score.

# Discussion
A public review system is incredibly important for rental services, especially when it comes to staying in someone else's home. The sentimnent analysis revealed that reviewers tend to be positive and subjective when leaving comments for a listing. This is in line with a histogram of ratings for the listings dataset - the majority of listings are rated very highly (75/100 and above), while just a few (less than 100) are scored below a 30/100.   

Each clustering method varied in performance with none of the models achieving a score over 0.67. However, in looking at the strip plots from the normalized k-means cluster model, I can see some patterns in clusters emerge. For example, the cluster with the highest prices also contained the most bedrooms, bathrooms, widest range of maximum nights' stay, the highest minimum nights threshold, the lowest rating scores, and the widest range of polarity and subjectivity scores. This makes sense if these higher priced listings are large homes intended for long-term rentals. As guests stay in a home longer and pay higher prices, they may expect higher quality services from a listing. Failure to meet those higher expectations could lead to the lower ratings and polarity in reviews.

# Further Research
There are many opportunities for further research into this topic. Every city has its "good" parts of town and the "bad" ones, I would be interested in mapping this data onto the city of Denver to see how latitude and longitude affect the clusters or sentiments towards listings. 

# References
Airbnb. (2021). About Us. Retrieved from https://news.airbnb.com/about-us/
Cox, M. (2021). *InsideAirbnb.* Retrieved from http://insideairbnb.com/index.html
Else-If. (2019). Visualizing multidimensional clusters. *Kaggle.* Retrieved from https://www.kaggle.com/ellecf/visualizing-multidimensional-clusters
Maklin, C. (2019, June 30). DBSCAN Python example: The optimal value for epsilon (eps). *Towards Data Science.* Retrieved from https://towardsdatascience.com/machine-learning-clustering-dbscan-determine-the-optimal-value-for-epsilon-eps-python-example-3100091cfbc
