# Introduction
What is the relationship between the nutritional value of a recipe and the way that people feel about it? This is the question I aim to answer by exploring the Recipes and Ratings dataset from Food.com. This dataset is particularly interesting because through it, we can gain insights on the way that we think about nutrition in home-cooked meals.
This dataset is made up of two sets of data, one containing recipes, their nutritional information, the number of ingredients, the number of steps, and information on who posted each recipe. The recipes data set has 83782 rows. The other set contains the interactions with each recipe from users. It includes the id of the recipe, the id of the user who rated it, the rating they gave, the date of their review. It has 731927 rows.

# Data Cleaning and Exploratory Data Analysis
In order to be able to use the data, I needed to clean it and do some preprocessing. The first step was merging the DataFrames. I started by merging the `raw_recipes` data with the `raw_interactions` data with a left merge on the id column from the recipe DataFrame and the recipe id column from the interactions DataFrame. This ensured that only the interactions regarding the recipes in the data were kept. I followed this up by replacing the ratings that had values of 0 with null values. This was done to not skew the analysis, like computing the average of the ratings. Next, I calculated the average rating per recipe as a pandas Series and merged it with the merged data from before. Finally, to conclude the cleaning step, I dropped the redundant `recipe_id` column, I dropped the `review` column as it would not see any use, and I dropped a row in the data that did not have interactions data associated with it. I noticed that the `nutrition` column in the data was actually a Series of strings not lists containing the nutritional information, so I converted it to be a Series of lists. Then I created separate columns for each point of nutritional data. With this, I was able to get started with some exploratory data analysis.
Below, is the first few rows of the cleaned dataset stored in a DataFrame called `merged`. *Some of the less relevant columns have been omitted for space purposes*.
|     id |   n_steps |   n_ingredients |   recipe_average_rating |   calories |   protein_(PDV) |
|-------:|----------:|----------------:|------------------------:|-----------:|----------------:|
| 333281 |        10 |               9 |                       4 |      138.4 |               3 |
| 453467 |        12 |              11 |                       5 |      595.1 |              13 |
| 306168 |         6 |               9 |                       5 |      194.8 |              22 |
| 306168 |         6 |               9 |                       5 |      194.8 |              22 |
| 306168 |         6 |               9 |                       5 |      194.8 |              22 |
## Exploratory Data Analysis
### Univariate Analysis
To start the analysis, I drew up some plots to get a look at the distributions of the `protein_(PDV)` column and the `recipe_average_rating` column that I computed in the cleaning step. The plot below is a box plot of the `recipe_average_rating` column. It quickly shows the median average rating, and the quartiles of the ratings distribution, giving a feel for the average ratings data at a glance. From it, we can see that half of the data are above a rating of 4.86 and that the first quartile is at the 4.5 mark. This clearly indicated that the majority of the ratings in the dataset are quite high.
<iframe src="assets/Avg_Ratings_Box.html" width="800" height="600" frameborder="0"></iframe>

### Bivariate Analysis
Next, I made some plots looking at pairs of columns to try to identify possible associations and patterns in the data. The most important plot I created was a scatter plot of protein content versus average ratings. To improve the readability of the plot, I applied a log transformation to the protein because there was a large range of values. The plot shows the log transformed protein content of each recipe against the average rating of the recipes. The trend line in the plot shows that as the protein content increases, the ratings decrease slightly. Another interesting thing about the scatter plot is that there are groupings of data points along round numbers like 3 or 3.5. This may indicate that there are recipes that received few reviews resulting in an average rating that is a nice number like the ones the data grouped around.
<iframe src="assets/Protein-vs-Avg-Ratings-scatter.html" width="800" height="600" frameborder="0"></iframe>

### Interesting Aggregates
Aggregating the data revealed some interesting things about the data. Splitting the data by ranges of protein and taking the mean of the `recipe_average_rating` column revealed that the recipes with the highest protein content had the highest mean in ratings. This bit of information was somewhat misleading because as it turned out that category has the smallest number of reviews. This was revealed by grouping the data by the protein ranges and then counting the number of ratings. The DataFrame shows that the max protein range only had seven ratings so it doesn't actually mean that users preferred recipes with that high of a protein amount as much as the averages suggested.
| log_protein_range   |   recipe_average_rating |
|:--------------------|------------------------:|
| 0.0-0.6             |                   10984 |
| 0.6-1.2             |                   15679 |
| 1.2-1.8             |                   23898 |
| 1.8-2.4             |                   28269 |
| 2.4-3.0             |                   38455 |
| 3.0-3.6             |                   35606 |
| 3.6-4.2             |                   42714 |
| 4.2-4.8             |                   28334 |
| 4.8-5.4             |                    6860 |
| 5.4-6.0             |                     617 |
| 6.0-6.6             |                     170 |
| 6.6-7.2             |                      47 |
| 7.2-7.8             |                      11 |
| 7.8-8.4             |                       7 |
# Assessment of Missingness
### NMAR Analysis
The dataset contains missing values in few columns, but some of them are missing by design like the `rating` column and the `recipe_average_rating` column where null values were used to replace ratings of 0 as seen in the data cleaning section. Does the dataset contain columns that are NMAR? After engaging with the data, I think that there are not any NMAR columns in the dataset; however, I do think that there is a MAR column.
### Missingness Dependency
The `description` column in the dataset contains missing values, so I performed permutation tests to assess the missingness of the `description` column. To start I identified columns that may be related to `description`. I ran permutation tests with `minutes`, `n_ingredients`, and `n_steps` because I suspected that recipes with higher values in these categories may be more complicated and thus, warrant an in depth description of the recipe resulting in a non-null value. This would mean that lower values in these columns may point to a missing description value. I also ran a permutation test with the `calories` column because I suspected that that column would likely have little to do the missingness of the `description` values. Labeling the data as either missing or not missing a description value and then permuting these and calculating the difference in group means, I ran permutation tests for the aforementioned columns. The permutation test with the `minutes` column resulted in a p-value of about 0.44 meaning that the observed difference in means is not statistically significant, indicating that the `description` column is most likely not dependent on the `minutes` column. The permutation test with the `n_ingredients` column is a different story. This one produced a p-value, which is well below the standard significance level of 0.05. This suggests that the difference in means is unlikely to have occurred by chance and that it supports the idea that the `description` column is MAR dependent on the number of ingredients. The final test that was statistically significant was the one with the `calories` column. This test yielded a p-value of around 0.048, just below 0.05, meaning that the `description` values are possibly missing at random dependent on the number of calories. The plots below show the empirical distributions of the permuted differences for calories and number of steps respectively.
<iframe src="assets/Perm-Diff-Cals.html" width="800" height="600" frameboarder="0"></iframe>
<iframe src="assets/Perm-Diff-N-Ing.html" width="800" height="600" frameborder="0"></iframe>

# Hypothesis Testing
The topic that interested me about this dataset was the relationship between nutritional content and the way that people feel about recipes based on their nutritional values. More specifically, I wanted to explore the relationship between the protein content of recipes and their average ratings. Before starting doing any of the testing, I suspected that recipes that are high in protein would receive ratings that are lower than those that are low in protein because healthy recipes are often not as tasty as unhealthy ones. To be specific, I defined 'high in protein' to be recipes that are in the top 50% of the dataset in protein content and define 'low in protein' to be recipes in the bottom half of the dataset by protein content. I chose these to be the definitions for a few reasons. First, the median protein content of the data is 18 which represents 18% of the recommended daily value. This amount is not too high, and it neatly splits the data into two groups. They are also clear definitions that allow for testing to be done nicely. This led me to the pair of hypotheses that follow.
### Null Hypothesis
The recipes that are high in protein receive ratings that are similar to the ratings that recipes that are low in protein receive.
### Alternative Hypothesis
The recipes that are high in protein receive ratings that are lower than the ratings that recipes that are low in protein receive. 
### Testing
To test whether high protein recipes get higher ratings, I performed a permutation test and used difference in means in the recipe_average_rating column. To start, I assigned a new column is_high_protein containing boolean values denoting whether each recipe is high in protein and then used it to perform a permutation test.
### Results
I found that the observed difference in means was just -0.0186 which is not very big, but according to the permutation test, the observed difference is much larger than the permuted differences, indicating that this difference was unlikely to have occurred by chance under the null hypothesis. The plot also shows just how large of a difference there is between the permuted difference and the observed difference. With our calculated p-value, we can safely say that we reject the null hypothesis that high protein and low protein recipes receive the same ratings, but we cannot claim that these results prove the alternative hypothesis! Below is the empirical distribution of the difference of means showing the gap between observed difference and the permuted differences.
<iframe src="assets/Hypoth-Dist.html" width="800" height="600" frameborder="0"></iframe>

# Framing a Prediction Problem
I now want to frame a prediction problem. The focus has been on the nutritional content of recipes and I will continue this theme by choosing to predict the number of calories that a recipe has. The dataset offers a few good options for predictor variables, which I extracted from the `nutrition` column in the data cleaning step. These columns are total fat, protein, carbohydrates, and saturated fat. To measure the performance of the prediction models, I used root mean square error (RMSE). I chose RMSE because it is easy to interpret and it is standard practice to do so.
# Baseline Model
The baseline model I started with used only the carbohydrates and saturated fat columns to predict the number of calories in a recipe. I used a standard scaler to scale the features and used linear regression to predict the values. I split the data into training and testing sets, performed 5 fold cross validation and generated RMSE scores for the training and validation sets. Finally, I fit the model with the whole training set and predicted the calories of the recipes. The results were not great. The baseline model resulted in an RMSE of around 200 for every fold for both training and validation. The test RMSE on the unseen data was about the same as the test and validation. The baseline model needed improvement and that is what the Final model did.
# Final Model
To improve the model several changes were made. The initial model only had two features, so I increased that started to work with saturated fat, carbohydrates, total fat, and protein values instead of just saturated fat and carbohydrates. Next, I engineered a couple of features. By summing the fat and carbohydrates, I created a new feature. Next, I created another feature which was the product of the protein and the carbohydrates. I then transformed them using a quantile transformer to get the values under control by handling the outliers. This was especially important because values in the dataset had very large ranges which could skew the performance of the model. I also added a hyperparameter: polynomial features. This increased the complexity of the model which allowed it to more closely fit the data improving its performance. I used `GridSearchCv` to find the best hyperparameters. The final model resulted in a significant improvement in performance. The test RMSE for the final model dropped down to about 42 from about 200. 
# Fairness Analysis
Once the final model was created and tested, I had to perform a fairness analysis on the model to determine if the model was performing better for certain groups over others. In order to assess the fairness, I had to perform a permutation test on two groups in the data. I decided it would be interesting to go back to the groups defined in the hypothesis testing: recipes high in protein and recipes low in protein. I used the difference in RMSE to determine the fairness, and I chose the following hypotheses to test.
### Null Hypothesis
The model is fair, and its RMSE is roughly the same for recipes that are high in protein and recipes that are low in protein. 
### Alternative Hypothesis
The model is not fair and will have lower RMSE for recipes high in protein.
### Result
There was an observed difference in RMSE of about -2.48 indicating that the high protein group performed slightly better than the low protein group. However, the permutation test revealed that this difference was not statistically significant with a p-value of about 0.4. This means that we fail to reject the null hypothesis, supporting the idea that the model is fair when it comes to performance between high and low protein recipes.