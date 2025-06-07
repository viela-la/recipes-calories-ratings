Author: Viela Lansangan

# Introduction

Those of us looking to lead healthier lives often start with improving our diets, usually by cooking homemade meals instead of ordering out. The most health-conscious may look to trying low-calorie recipes, since "low-calorie" is commonly equated to "healthy." The satisfaction of not only cooking for ourselves, but also cooking for the sake of a better diet, is something worth celebrating and even recording -- perhaps by leaving a favorable review on the recipe that helped us get there. Following that thought process, **I wanted to explore whether recipes with lower calories are rated better than those with higher calories.**

## The Basic Datasets

Two separate datasets were used for the purpose of this exploration. The first one, `recipes`, contains recipes posted on food.com since 2008 and consists of 83782 rows and 12 columns as outlined below:

| Column | Description |
| ------ | ----------- |
| `name` | Title of the recipe page |
| `id` | ID of recipe |
| `minutes` | Duration of cooking time |
| `contributor_id` | User ID who submitted this recipe |
| `submitted` | Date recipe was submitted |
| `tags` | Food.com tags for recipe |
| `nutrition` | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)] |
| `n_steps` | Number of steps in recipe |
| `steps` | Text for recipe steps, in order |
| `description` | User-provided description |
| `ingredients` | Text for recipe ingredients |
| `n_ingredients` | Number of recipe ingredients |

The second dataset, `interactions`, contains reviews and ratings submitted for the recipes in the `recipes` dataset. It consists of 731927 rows and 5 columns as outlined below:

| Column | Description |
| ------ | ----------- |
| `user_id` | User ID |
| `recipe_id` | Recipe ID |
| `date` | Date of interaction |
| `rating` | Rating given |
| `review` | Review text |

For now, the most relevant columns from both datasets are the `rating` and `nutrition` columns.

# Data Cleaning and Exploratory Data Analysis

## Data Cleaning

The first step I took in the cleaning process was merging the `recipes` and `interactions` datasets. I left-merged them respectively on the `id` and `recipe_id` columns and created the `recipes_ints` table. This ensures each interaction is matched with its respective recipe.

The second step I took was replacing all of the ratings of 0 with `np.nan`. This ensures that interactions simply missing reviews aren't actually providing ratings of 0 stars and possibly introducing bias and skewing recipes' ratings to be lower.

The third step I took was creating the `avg_rating` column, which contains the mean rating of all of the interactions of each recipe in the table. This gives a more generalized interpretation of each recipe's overall rating.

Since my question is directly connected to the nutritional value of each recipe, I separated the listed values in the `nutrition` column into their own respective columns as `float64`s, with PDV standing for "Percentage of Daily Value", or how much that nutrient contributes to a total daily diet. The following columns were created and added back to `recipes_ints`:

| Column | Description |
| ------ | ----------- |
| `calories` | Number of calories |
|`total_fat_pdv` | Total fat (PDV) |
| `sugar_pdv` | Sugar (PDV) |
| `sodium_pdv` | Sodium (PDV) |
| `protein_pdv` | Protein (PDV) |
| `saturated_fat_pdv` | Saturated fat (PDV) |
| `carbohydrates` | Carbohydrates (PDV) |

The last step I took in my data cleaning process was converting the dates in the `date` column from strings into `Datetime` objects, in case I ever wanted to use chronological analysis.

Below is the head of the cleaned `recipes_ints` DataFrame, which consists of 234429 rows and 24 columns. Displayed are only the most relevant columns:

|    | name                                 |   recipe_id |   avg_rating |   calories |   total_fat_pdv |   sugar_pdv |   saturated_fat_pdv |   carbohydrates_pdv |
|---:|:-------------------------------------|------------:|-------------:|-----------:|----------------:|------------:|--------------------:|--------------------:|
|  0 | 1 brownies in the world    best ever |      333281 |            4 |      138.4 |              10 |          50 |                  19 |                   6 |
|  1 | 1 in canada chocolate chip cookies   |      453467 |            5 |      595.1 |              46 |         211 |                  51 |                  26 |
|  2 | 412 broccoli casserole               |      306168 |            5 |      194.8 |              20 |           6 |                  36 |                   3 |
|  3 | 412 broccoli casserole               |      306168 |            5 |      194.8 |              20 |           6 |                  36 |                   3 |
|  4 | 412 broccoli casserole               |      306168 |            5 |      194.8 |              20 |           6 |                  36 |                   3 |


To further simplify my variate analyses, I created a subset of `recipes_ints`. First, I grouped the table by `recipe_id` and aggregated by the `mean`, which simply resulted in a single row for each recipe. Then I queried only for the columns displayed above, minus the `name` column. Finally, I cut and sorted each recipe into a `recipe_bin`, which essentially smoothed out the decimal average ratings into five distinct ordinal categories. This allows me to analyze per star rating.

*Note*: This smoothing resulted in a rounding-up of each average rating. For example, recipes with an average rating of 1.2 were sorted into a `rating_bin` of 2 rather than 1. Inadvertently, many recipes fell into the higher-rating bins.

Below is the resulting `cals_avg_ratings` DataFrame head:

|   recipe_id |   avg_rating |   calories |   total_fat_pdv |   sugar_pdv |   sodium_pdv |   protein_pdv |   saturated_fat_pdv |   carbohydrates_pdv |   rating_bin |
|------------:|-------------:|-----------:|----------------:|------------:|-------------:|--------------:|--------------------:|--------------------:|-------------:|
|      469990 |            1 |      110.5 |              12 |          22 |            2 |             2 |                  17 |                   3 |            1 |
|      423015 |            1 |      124.7 |               6 |          14 |            1 |             6 |                  11 |                   6 |            1 |
|      416845 |            1 |      117.4 |              10 |           0 |            6 |             2 |                   4 |                   4 |            1 |
|      468835 |            1 |      260.3 |              24 |          12 |           13 |            11 |                  16 |                   8 |            1 |
|      289197 |            1 |      351.6 |              37 |           7 |           20 |            51 |                  33 |                   2 |            1 |

## Univariate Analysis

For this first analysis, I examined the distribution of recipes per rating group. As discussed above, there is a higher number of recipes in bins with more favorable ratings (4, 5) than those less favorable (1, 2, 3). However, there may also be an implicit reason for this bias. Generally, users don't take the time to leave reviews or ratings for things they felt indifferently about, or even disliked. This can also be the reason for such a left-skewed distribution.

<iframe
src = 'assets/rating-dist.html'
width='800'
height='600'
frameborder='0'
></iframe>

For this next analysis, I examined the distribution of total calories.

First, I plotted the distribution of all calories. It is very heavily skewed to the right, meaning there is at least one recipe with an exceptionally high calorie count (a quick query reveals that the most severe outlier is a recipe for powdered hot cocoa mix).

<iframe
src = 'assets/calorie-dist.html'
width='800'
height='600'
frameborder='0'
></iframe>

Then, I identified and removed outliers using the IQR method. The resulting plot is almost normal, with a slight right skew in the form of a decreasing trend. This suggests that as calories increase, there are less of those recipes on the website.

<iframe
src = 'assets/calorie-dist-iqr.html'
width='800'
height='600'
frameborder='0'
></iframe>

## Bivariate Analysis

I performed a bivariate analysis comparing the rating bins and the average calories per bin. The average number of calories per rating bin seems to decrease as the rating increases. This implies that higher-rated recipes have less calories.

<iframe
src = 'assets/cals-per-group.html'
width='800'
height='600'
frameborder='0'
></iframe>

## Interesting Aggregates

Here, I wanted to compare the average nutritional values per rating bin. I grouped the `cals_avg_ratings` DataFrame by `rating_bin`, aggregated by the mean, and examined the resulting table.

|   rating_bin |   avg_rating |   calories |   total_fat_pdv |   sugar_pdv |   sodium_pdv |   protein_pdv |   saturated_fat_pdv |   carbohydrates_pdv |
|-------------:|-------------:|-----------:|----------------:|------------:|-------------:|--------------:|--------------------:|--------------------:|
|            1 |      1       |    445.615 |         34.6621 |     76.2733 |      45.4329 |       29.4856 |             44.3056 |             14.8591 |
|            2 |      1.98443 |    462.408 |         35.3991 |     76.5227 |      32.8623 |       31.687  |             48.0188 |             15.5869 |
|            3 |      2.96783 |    436.94  |         31.6625 |     81.5238 |      37.3587 |       33.4546 |             38.9364 |             14.9175 |
|            4 |      3.95373 |    426.951 |         31.5778 |     62.6154 |      28.1859 |       35.281  |             38.778  |             13.6748 |
|            5 |      4.89969 |    426.306 |         32.556  |     68.1843 |      28.5697 |       32.6561 |             40.1286 |             13.5941 |

Bin 2 seems to have the highest average calories, total fat (PDV), saturated fat (PDV), and carbohydrates (PDV). These same columns seem to generally decrease onward with higher ratings.

For sake of comparison, I created another DataFrame with the caloric outlier(s) removed and performed a similar aggregation.

|   rating_bin |   avg_rating |   calories |   total_fat_pdv |   sugar_pdv |   sodium_pdv |   protein_pdv |   saturated_fat_pdv |   carbohydrates_pdv |
|-------------:|-------------:|-----------:|----------------:|------------:|-------------:|--------------:|--------------------:|--------------------:|
|            1 |      1       |    304.122 |         21.5167 |     55.8792 |      42.6059 |       22.5818 |             26.7584 |             10.5372 |
|            2 |      1.98516 |    336.227 |         24.2551 |     52.0845 |      27.1267 |       26.7855 |             31.272  |             11.201  |
|            3 |      2.96726 |    334.141 |         23.7032 |     49.7767 |      23.6167 |       29.452  |             29.2052 |             10.7433 |
|            4 |      3.95441 |    336.917 |         23.9987 |     44.3904 |      23.9751 |       30.4784 |             29.7278 |             10.6373 |
|            5 |      4.89906 |    328.02  |         24.2504 |     47.731  |      24.1633 |       27.8554 |             30.048  |             10.1777 |

Interestingly, all of the averages all around generally decreased. Of course, higher caloric recipes in turn have higher nutritional values. The removal of those nutritional and caloric outliers removed the skew. Bin 2 still leads in total fat (PDV), saturated fat (PDV), and carbohydrates (PDV). Highest average calories now just barely goes to Bin 4. Bin 1 having the lowest average calories now contradicts my initial bivariate analysis where Bin 5 had the lowest average calories. 

# Assessment of Missingness

## NMAR Analysis
The columns of `recipes_ints` with significant missingness are `rating`, `review`, and `description`. I suspect `rating` and `review` are NMAR because users don't always take the time to leave reviews on recipes, especially if they were indifferent about the outcome or otherwise didn't feel very strongly about it. Users who do leave reviews often do it out if they hold intense feelings towards the recipe.

## Missingness Dependency
### Dependent
Here, I'll be testing the dependency of the `rating` column missingness upon the `calories` column using a permutation test.

**Null Hypothesis**: The missingness of ratings does not depend on the number of calories in the recipe.

**Alternate Hypothesis**: The missingness of ratings does depend on the number of calories in the recipe.

**Test Statistic**: Absolute mean differences of calories between the group missing ratings and the group not missing ratings.

**Significance Level**: 0.1

The two distributions of missingness *look* similar.
<iframe
src = 'assets/cals-missing.html'
width='800'
height='600'
frameborder='0'
></iframe>

I shuffled the missingness of the `rating` column in `recipes_ints` 500 times and calculated the absolute mean difference for each simulation. The **observed test statistic** is 69.01, shown on the empirical distribution graph below. I found a p-value of 0.0, which is < 0.05. I **reject the null hypothesis** and conclude that the the missingness of `rating` depends on `calories`. 
<iframe
src = 'assets/cals-missing-emp.html'
width='800'
height='600'
frameborder='0'
></iframe>

### Independent
Here, I'll be testing the independency of the `rating` column upon the `minutes` column using a permutation test, similar to before.

**Null Hypothesis**: The missingness of ratings does not depend on the number of minutes in the recipe.

**Alternate Hypothesis**: The missingness of ratings does depend on the number of minutes in the recipe.

**Test Statistic**: Absolute mean differences of cooking time between the group missing ratings and the group not missing ratings.

**Significance Level**: 0.1

The two distributions of missingness are somewhat similar.
<iframe
src = 'assets/min-missing.html'
width='800'
height='600'
frameborder='0'
></iframe>

Like in the dependent section, I shuffled the missingness of the `rating` colum 500 times and calculated the asbolute mean difference for each simulation. The **observed test statistic** is 51.45, shown on the graph below. I found a p-value of 0.118, which is > 0.05. I **fail to reject the null hypothesis** and conclude that the missingness of `rating` is independent on `minutes`, or the cooking time of the recipe.
<iframe
src = 'assets/min-missing-emp.html'
width='800'
height='600'
frameborder='0'
></iframe>

# Hypothesis Testing
I return to my original question of whether recipes with lower calories are rated better than those with higher calories. To reveal the relationship between calories and ratings, I used the `cals_avg_ratings` DataFrame for a permutation test.

**Null Hypothesis**: All recipes are rated regardless of total calories.

**Alternate Hypothesis**: Recipes with less calories are rated higher than recipes with more calories.

**Test Statistic**: Mean difference between ratings of low-calorie recipes and high-calorie recipes.

**Significance Level**: 0.1

For my test statistic, I chose the mean difference instead of absolute mean difference because I'm measuring a directional difference between two distinct categories.

First, I divided the DataFrame into two: recipes whose calorie count is higher than the overall average and recipes whose calorie count is lower. This simplifies classification of "high calorie" and "low calorie." I used the `calories` and `avg_rating` columns for the 1000 simulations, each time shuffling the `calories` column. I also calculated and stored the test statistic to create the below distribution:
<iframe
src = 'assets/hypoth-emp.html'
width='800'
height='600'
frameborder='0'
></iframe>

The **observed test statistic** is 0.006 and the p-value is 0.09 -- which is < 0.1. I **reject the null hypothesis**. Recipes tend to be rated higher if they have less calories. User ratings may reflect the positivity associated not only with homemade meals but also health consciousness.


# Framing a Prediction Problem

Now, I plan to predict the **rating of a recipe** in the form of a **classification problem**. Since I'll be predicting the rating based on a discrete ordinal scale of 1-5 "stars", I'll be building a multiclass classifier model. 

I chose `rating_bin` as the response variable because it classifies all ratings into distinct categories. There's significant correlation between rating and calorie count, so I may be able to use calories to make my predictions. 

The metric I'll be using is the F1-score. As shown in the univariate analysis, there's a much higher number of recipes rated 3 or higher than lower -- causing imbalanced classes. If I were to use, say, accuracy as a metric, this imbalance would not be properly handled. 

At the time of prediction, the information we have are all of the columns in the original `recipes` dataset, not yet the information in the `interactions` dataset. The `recipes` information is readily available at the time of viewing the recipe, regardless of any reviews or ratings left on them. 

# Baseline Model

For the baseline model, I used a Random Forest Classifier. These are the features I used:

- `calories` (quantitative): As previously stated, I'll be utilizing the discovered correlation between low calorie count and high ratings in order to predict recipe ratings. I transformed this column using a RobustScaler so that any outliers are appropriately handled and they won't introduce bias.
- `carbohydrates_pdv` (quantitative): A bivariate analysis revealed that low average carbohydrate PDVs are also correlated with higher ratings. Additionally, carbohydrate counts are commonly associated with calorie counts, since in the public eye low carbs = low calories = healthy = high rating. I figured this association could help my prediction model.

The F1 score on the training data was **0.6829**. It's not especially low, but there's plenty of room for improvement.

# Final Model

For my final model, I started with choosing the following features:

- `n_steps` (quantitative): A bivariate graph showed that recipes with less steps in the cooking process were, on average, rated higher than those with more steps. This may be because users prefer quicker cooking processes or less verbose instructions. This correlation led me to believe it would help improve my model. I used a RobustScaler to transform this column as I found a few extreme outliers that I wanted handled.

- `calories` (quantitative): Like previously stated many times, there is a notable relationship between calories and ratings. I kept this column transformed with RobustScaler.

- `total_fat_pdv`, `sugar_pdv`, `saturated_fat_pdv`, `carbohydrates_pdv` (quantitative): A bivariate graph for each of these columns' rating bins' averages revealed that recipes with low values for these nutritional categories are rated higher. Total fat, sugar, saturated fat, and carbohydrates  are usually equated with calories from a health perspective, as in a high count in one means a high count in another. This relationship also led me to believe I could use these columns to further improve my model. I transformed these columns with a RobustScaler to effectively handle outliers.

I used a RandomForestClassifier again, but finetuned the `max_depth` and `n_estimators` hyperparameters using a GridSearchCV. The optimal values for these parameters were 2 and 12, respectively.

The new model resulted in an F1 score of **0.6867**, which is a **0.0038** improvement from the baseline model. Again, not especially high. The adjustments I had made weren't too effective

# Fairness Analysis

To evaluate the fairness of my model, I split the recipes into two groups: low carbs and high carbs. The split was done according to the median of `carbohydrates_pdv` (9.0), with the PDVs >= 9.0 being classified as `high_carb` and those with PDVs < 9.0 being classified as `low_carb`. I chose to split according to the median because the mean is prone to outliers, which could lead to a very uneven split. My evaluation metric of choice was **precision parity**. I think it's important to identify true positives when it comes to nutrition and ratings. People with dietary restrictions, such as consuming a certain amount of carbs, should be able to find good recipes that fit their needs.

**Null Hypothesis**: My model is fair, and its precision for predicting the ratings of low-carb recipes is the same as its precision for high-carb recipes.

**Alternative Hypothesis**: My model is unfair, and its precision for predicting the ratings of low-carb recipes is higher than its precision for high-carb recipes.

**Test Statistic**: Difference in precision (low carb - high carb)

**Significance Level**: 0.1

My observed test statistic was **0.014**. For my permutation test, I created a new binary column `low_carb`, which contained True or False for each recipe according to the criteria I described above. I ran 500 simulations where I permutated that column, then calculated the test statistic each time. I found a p-level of 0.002, which is < 0.1, so I **reject the null hypothesis** that my model is fair. My model's precision for predicting the rating of recipes with low carbs is better than its precision for rating recipes with high carbs.

<iframe
src = 'assets/fairness.html'
width='800'
height='600'
frameborder='0'
></iframe>
