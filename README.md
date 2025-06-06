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
The columns of `recipes_ints` with significant missingness are `rating`, `review`, and `description`. 

# Hypothesis Testing

# Framing a Prediction Problem

# Baseline Model

# Final Model

# Fairness Analysis