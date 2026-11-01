**1.Pick any one app you use daily (like Zomato, Instagram, or Flipkart) and write a short problem definition for a data science project that could improve the app's user experience. Clearly state the business goal and what data you would need.**

**SOLUTION**

**App chosen: Instagram**

**Problem Definition:** Instagram wants to improve user experience by showing each user more relevant posts and reels based on their interests. Many users may lose interest if they see content that does not match their preferences, leading to lower engagement and reduced time spent on the app. A data science project can analyze user behavior and recommend content that matches their interests more accurately.

**Business Goal:** The main business goal is to increase user engagement, retention, and time spent on the app by personalizing the content feed. This can help Instagram retain users and improve overall satisfaction.

**Data Needed:**
- User profile data: age, location, interests, language, device type
- User interaction data: likes, comments, shares, saves, follows, scroll depth
- Content data: post type, hashtags, category, creator, engagement rate
- Session data: time spent on app, active hours, frequency of visits
- Historical behavior: content viewed, skipped, clicked, or ignored

By using this data, Instagram can build a recommendation system that delivers personalized content and improves the user experience.

**2.Use the requests library in Python to fetch the latest 5 posts from the JSONPlaceholder API (https://jsonplaceholder.typicode.com/posts) and print the title of each post.<br><br><em><strong>Hint:</strong> Use requests.get() and response.json().</em>**

**SOLUTION**

```python
import requests

response = requests.get('https://jsonplaceholder.typicode.com/posts')
posts = response.json()

for post in posts[:5]:
    print(post['title'])
```

**3.Given a CSV file with columns: song_name, artist, streams, and genre (simulate a mini Spotify dataset with at least 10 rows), write Python code to load the data using pandas and display the number of songs in each genre.**

**SOLUTION**

```python
import pandas as pd

# Load the CSV file
songs = pd.read_csv('spotify_songs.csv')

# Count the number of songs in each genre
genre_counts = songs['genre'].value_counts()

# Display the result
print(genre_counts)
```

**4.Create a simple bar chart using matplotlib to visualize the number of movies per genre from a dataset (you can make up a small dataset with at least 3 genres and 10 movies, like BookMyShow's movie listings).**

**SOLUTION**

```python
import matplotlib.pyplot as plt

# Sample movie dataset
movie_genres = ['Action', 'Comedy', 'Drama', 'Thriller']
movie_counts = [4, 3, 5, 2]

# Create bar chart
plt.bar(movie_genres, movie_counts, color=['blue', 'green', 'orange', 'red'])
plt.title('Number of Movies per Genre')
plt.xlabel('Genre')
plt.ylabel('Number of Movies')
plt.show()
```

**5.Use ChatGPT to suggest 3 new features that could be engineered from a dataset of online food orders (like Zomato orders: order time, restaurant, cuisine, delivery time, rating). List the features and explain how each could help a data science model.<br><br><em><strong>Hint:</strong> Paste the column names into ChatGPT and ask for feature engineering ideas.</em>**

**SOLUTION**

Here are 3 useful engineered features for a food-order dataset:

1. **Peak Order Time Flag**  
   A new feature such as "is_peak_hour" can be created from the order time column. For example, lunch and dinner time slots can be marked as peak hours. This helps a model understand whether orders placed during busy times are more likely to be delayed or more likely to lead to high customer ratings.

2. **Delivery Speed Score**  
   This can be calculated as a ratio of delivery time to distance or by comparing actual delivery time against expected time. It helps a model predict customer satisfaction, estimate delivery performance, and identify restaurants with slow service.

3. **Cuisine Popularity Index**  
   A feature can be created by counting how often a cuisine is ordered or how highly it is rated across the dataset. This helps a model understand whether certain cuisines are more preferred by users and can improve recommendations or demand forecasting.

These engineered features make the data more informative for machine learning models by capturing patterns such as time-based demand, service quality, and customer preference.