# Final Project: Wine Analysis

## Authors: Jeffrey Hwang, Donovan Pilcher, Aaron Bajorunas, Tianle Qi
## Date: 12/9/2024

## Setup and Configuration

```{r}
# install.packages("ggplot2")
# install.packages("dplyr")
# install.packages("ggcorrplot")
# install.packages("tidyr")
# install.packages("car")
# install.packages('leaps')
#install.packages("randomForest")
#install.packages("gridExtra")
library(gridExtra)
library(randomForest)  
library(leaps)
library(car)
library(ggplot2)
library(dplyr)
library(tidyr)
library(ggcorrplot)
```

## Data Import and Cleaning

```{r}
redwine <- read.csv("winequality-red.csv")
whitewine <- read.csv("winequality-white.csv")

redwine$wine_type <- 1
whitewine$wine_type <- 0

wine_data <- rbind(redwine, whitewine)

head(wine_data)
```

## Exploratory Data Analysis (EDA)

```{r}
wine_data %>%
  pivot_longer(cols = -c(quality), names_to = "Variable", values_to = "Value") %>%
  ggplot(aes(x = Value)) +
  geom_histogram(bins = 30, fill = "steelblue", color = "black") +
  facet_wrap(~Variable, scales = "free_x") +
  theme_minimal() +
  labs(title = "Distributions of Independent Variables")
```

The code above generates histograms for each independent variable in wine_data, showing how each histogram is skewed from the outliers.

```{r}
remove_outliers <- function(df) {
  numeric_cols <- sapply(df, is.numeric)
  numeric_cols <- setdiff(names(df)[numeric_cols], c("quality", "wine_type"))  # exclude 'quality' and 'wine_type'
  
  df_clean <- df
  
  for (col in numeric_cols) {
    Q1 <- quantile(df[[col]], 0.25, na.rm = TRUE)
    Q3 <- quantile(df[[col]], 0.75, na.rm = TRUE)
    IQR <- Q3 - Q1
    lower_bound <- Q1 - 1.5 * IQR
    upper_bound <- Q3 + 1.5 * IQR
    
    df_clean <- df_clean[df_clean[[col]] >= lower_bound & df_clean[[col]] <= upper_bound, ]
  }
  
  return(df_clean)
}

wine_data_clean <- remove_outliers(wine_data)
```

Using IQR (1.5 times below and above Q1 and Q3), we cleaned the data from outliers while excluding quality and wine type variables.

```{r}
wine_data_clean %>%
  pivot_longer(cols = -c(quality, wine_type), names_to = "Variable", values_to = "Value") %>%
  ggplot(aes(x = Value)) +
  geom_histogram(bins = 30, fill = "steelblue", color = "black") +
  facet_wrap(~Variable, scales = "free_x") +
  theme_minimal() +
  labs(title = "Distributions of Independent Variables")
```

The plot above shows the new histograms for each independent variable with the cleaned data, demonstrating better symmetry after removing outliers.

```{r}
ggcorrplot(cor(wine_data_clean), type = "lower", lab = TRUE) +
  labs(title = "Correlation Matrix of Variables")
```

This correlation matrix heatmap displays the Pearson correlations between all variables, visually highlighting high, low, or no correlation.

## Final Model

```{r}
wine_data_clean$density_residual_sugar <- wine_data_clean$density * wine_data_clean$residual.sugar

wine_data_final <- wine_data_clean %>%
  select(-density, -residual.sugar, -fixed.acidity, -citric.acid)
```

We created a new feature 'density_residual_sugar' by multiplying density and residual.sugar, then removed redundant columns to prepare our final dataset.

```{r}
formula_significant3 <- quality ~
  volatile.acidity +
  chlorides +
  total.sulfur.dioxide +
  sulphates +
  alcohol +
  wine_type +
  density_residual_sugar +
  
  I(volatile.acidity^2) +
  I(free.sulfur.dioxide^2) +
  I(alcohol^2) +
  
  volatile.acidity:pH +
  volatile.acidity:alcohol +
  
  chlorides:pH +
  chlorides:wine_type +
  
  free.sulfur.dioxide:sulphates +
  free.sulfur.dioxide:alcohol +
  free.sulfur.dioxide:wine_type +
  
  total.sulfur.dioxide:sulphates +
  
  pH:sulphates +
  pH:wine_type +
  pH:density_residual_sugar +
  
  alcohol:wine_type

model_significant3 <- lm(formula_significant3, data = wine_data_final)
summary(model_significant3)
```

This final model represents our optimal combination of terms for predicting wine quality, achieving an adjusted R² value of 0.3372.

## Model Validation

```{r}
# Forward selection
null_model <- lm(quality ~ 1, data = wine_data_final)
forward_model <- step(null_model, 
                     direction = "forward", 
                     scope = list(lower = null_model, upper = model_significant3), 
                     trace = 0)
summary(forward_model)
```

```{r}
# Backward elimination
backward_model <- step(model_significant3, direction = "backward", trace = 0)
summary(backward_model)
```

Both forward selection and backward elimination resulted in the same variables and adjusted R², confirming our variable selection.

## Conclusions

The final linear regression model demonstrated strong performance with an adjusted R² of 0.3372, explaining 33.7% of the variance in wine quality. Key findings include:

1. The model shows good fit with relatively low prediction error
2. Key predictors include volatile acidity, chlorides, total sulfur dioxide, sulphates, alcohol, wine type, and density-residual sugar interaction
3. All key predictors showed highly significant p-values
4. Quadratic terms and interaction effects significantly improved prediction accuracy

### Future Work

Potential areas for future research include:
- Further model generalization while maintaining similar adjusted R²
- Identification of additional external variables affecting wine quality
- Investigation of non-linear relationships between variables

## Appendix: Statistical Analysis

### Initial Model Exploration

```{r}
# Fit initial model with all base variables
model0 <- lm(quality ~ ., data = wine_data_clean)
summary(model0)
```

Our initial model using all base variables showed:
- Non-significant variables: citric.acid, residual.sugar, and density_residual_sugar
- Model explains 28.4% of wine quality variability
- Overall statistical significance (F-statistic: 154.1, p-value < 2.2e-16)

```{r}
# Check for multicollinearity
options(scipen = 999, digits = 7)
vif0 <- vif(model0)
print(vif0)
```

Variance Inflation Factor (VIF) analysis revealed:
- High multicollinearity in residual.sugar, density, and density_residual_sugar
- Most other variables showed low to moderate VIF values
- Decision made to remove density and residual.sugar due to high VIF values

### Model Refinement - Stage 1

```{r}
# Create interaction term and remove high VIF variables
wine_data_clean$density_residual_sugar <- wine_data_clean$density * wine_data_clean$residual.sugar
model1 <- lm(quality ~ . - residual.sugar - density, data = wine_data_clean)

summary(model1)
print(vif(model1))
```

First refinement results:
- Slightly lower adjusted R² after removing residual.sugar and density
- All remaining variables significant except fixed.acidity and citric.acid
- All VIF values now below 5, resolving multicollinearity issues

### Model Refinement - Stage 2

```{r}
# Remove non-significant variables
model2 <- lm(quality ~ . - residual.sugar - density - fixed.acidity - citric.acid, 
             data = wine_data_clean)

summary(model2)
print(vif(model2))
```

Second refinement showed:
- Performance nearly identical to model1 (similar Adjusted R²)
- Significant F-statistic (213.3, p-value < 0.0001)
- Further improved model parsimony

### Final Base Model

```{r}
# Create final dataset and fit base model
wine_data_final <- wine_data_clean %>%
  select(-density, -residual.sugar, -fixed.acidity, -citric.acid)

model3 <- lm(quality ~ ., data = wine_data_final)

summary(model3)
print(vif(model3))
```

Final base model characteristics:
- All predictors significant
- Low VIF values throughout
- Clean, focused dataset with necessary predictors only

### Advanced Model Development

```{r}
# Full model with quadratic and interaction terms
formula_full <- quality ~
  volatile.acidity +
  chlorides +
  free.sulfur.dioxide +
  total.sulfur.dioxide +
  pH +
  sulphates +
  alcohol +
  wine_type +
  density_residual_sugar +
  
  I(volatile.acidity^2) +
  I(chlorides^2) +
  I(free.sulfur.dioxide^2) +
  I(total.sulfur.dioxide^2) +
  I(pH^2) +
  I(sulphates^2) +
  I(alcohol^2) +
  I(density_residual_sugar^2) +
  
  volatile.acidity:chlorides +
  volatile.acidity:free.sulfur.dioxide +
  volatile.acidity:total.sulfur.dioxide +
  volatile.acidity:pH +
  volatile.acidity:sulphates +
  volatile.acidity:alcohol +
  volatile.acidity:wine_type +
  volatile.acidity:density_residual_sugar +
  
  chlorides:free.sulfur.dioxide +
  chlorides:total.sulfur.dioxide +
  chlorides:pH +
  chlorides:sulphates +
  chlorides:alcohol +
  chlorides:wine_type +
  chlorides:density_residual_sugar +
  
  free.sulfur.dioxide:total.sulfur.dioxide +
  free.sulfur.dioxide:pH +
  free.sulfur.dioxide:sulphates +
  free.sulfur.dioxide:alcohol +
  free.sulfur.dioxide:wine_type +
  free.sulfur.dioxide:density_residual_sugar +
  
  total.sulfur.dioxide:pH +
  total.sulfur.dioxide:sulphates +
  total.sulfur.dioxide:alcohol +
  total.sulfur.dioxide:wine_type +
  total.sulfur.dioxide:density_residual_sugar +
  
  pH:sulphates +
  pH:alcohol +
  pH:wine_type +
  pH:density_residual_sugar +
  
  sulphates:alcohol +
  sulphates:wine_type +
  sulphates:density_residual_sugar +
  
  alcohol:wine_type +
  alcohol:density_residual_sugar +
  
  wine_type:density_residual_sugar

full_model <- lm(formula_full, data = wine_data_final)
summary(full_model)
```

Full model improvements:
- Adjusted R² increased to 0.3387
- Significant improvement over base model
- Includes comprehensive interaction and quadratic terms

### Model Selection

```{r}
# Stepwise regression in both directions
stepwise_model <- step(full_model, direction = "both", trace = 0)
summary(stepwise_model)
```

Stepwise selection results:
- Improved adjusted R² with fewer predictors
- More significant F-statistic than initial model
- Balanced complexity and prediction accuracy

### First Significant Model Iteration

```{r}
formula_significant <- quality ~
  volatile.acidity +
  chlorides +
  total.sulfur.dioxide +
  sulphates +
  alcohol +
  wine_type +
  density_residual_sugar +
  
  I(volatile.acidity^2) +
  I(free.sulfur.dioxide^2) +
  I(total.sulfur.dioxide^2) +
  I(alcohol^2) +
  I(density_residual_sugar^2) +
  
  volatile.acidity:total.sulfur.dioxide +
  volatile.acidity:pH +
  volatile.acidity:alcohol +
  volatile.acidity:density_residual_sugar +
  
  chlorides:pH +
  chlorides:wine_type +
  
  free.sulfur.dioxide:sulphates +
  free.sulfur.dioxide:alcohol +
  free.sulfur.dioxide:wine_type +
  
  total.sulfur.dioxide:sulphates +
  total.sulfur.dioxide:wine_type +
  
  pH:sulphates +
  pH:wine_type +
  pH:density_residual_sugar +
  
  alcohol:wine_type +
  wine_type:density_residual_sugar

model_significant <- lm(formula_significant, data = wine_data_final)
summary(model_significant)
```

First iteration achievements:
- Reduced to 28 terms while maintaining adjusted R²
- Improved F-statistic compared to stepwise model
- Enhanced model interpretability

### Final Model Refinement

```{r}
formula_significant2 <- quality ~
  volatile.acidity +
  chlorides +
  total.sulfur.dioxide +
  sulphates +
  alcohol +
  wine_type +
  density_residual_sugar +
  
  I(volatile.acidity^2) +
  I(free.sulfur.dioxide^2) +
  I(alcohol^2) +
  
  volatile.acidity:pH +
  volatile.acidity:alcohol +
  
  chlorides:pH +
  chlorides:wine_type +
  
  free.sulfur.dioxide:sulphates +
  free.sulfur.dioxide:alcohol +
  free.sulfur.dioxide:wine_type +
  
  total.sulfur.dioxide:sulphates +
  
  pH:sulphates +
  pH:wine_type +
  pH:density_residual_sugar +
  
  alcohol:wine_type

model_significant2 <- lm(formula_significant2, data = wine_data_final)
summary(model_significant2)
```

Final refinement results:
- Further reduced to 25 predictors
- Maintained similar adjusted R²
- Improved F-statistic
- Enhanced model clarity and interpretability


## Additional Exploratory Analysis

### Quality Distribution Analysis

```{r}
ggplot(wine_data, aes(x = quality)) +
  geom_bar(fill = "steelblue") +
  labs(title = "Distribution of Wine Quality",
       x = "Quality Score",
       y = "Number of Wines",
       caption = "Source: Wine Quality Dataset") +
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5))
```

The quality distribution shows a relatively symmetric pattern, indicating a balanced representation across different quality levels in our dataset.

### Feature Importance Analysis

```{r}
# Random Forest for feature importance
library(randomForest)

rf_model <- randomForest(quality ~ ., 
                        data = wine_data, 
                        importance = TRUE,
                        ntree = 500)

importance(rf_model)

varImpPlot(rf_model,
           main = "Feature Importance for Wine Quality",
           cex = 0.8)
```

The random forest analysis reveals:
- Alcohol content is the strongest predictor of wine quality
- Wine type shows the least significance among all features
- Variable importance aligns with our linear model findings

### Key Feature Relationships

```{r}
# Create detailed boxplots for alcohol and volatile acidity
p1 <- ggplot(wine_data, aes(x = factor(quality), y = alcohol)) +
  geom_boxplot(fill = 'lightblue') +
  labs(title = 'Alcohol Content by Wine Quality',
       x = 'Quality Score',
       y = 'Alcohol (% by volume)') +
  theme_minimal()

p2 <- ggplot(wine_data, aes(x = factor(quality), y = volatile.acidity)) +
  geom_boxplot(fill = 'lightgreen') +
  labs(title = 'Volatile Acidity by Wine Quality',
       x = 'Quality Score',
       y = 'Volatile Acidity (g/L)') +
  theme_minimal()

# Create interaction plot
p3 <- ggplot(wine_data, 
             aes(x = volatile.acidity, y = alcohol, color = factor(quality))) +
  geom_point(alpha = 0.6) +
  geom_smooth(method = 'lm', se = FALSE, color = 'red') +
  labs(title = 'Interaction: Alcohol and Volatile Acidity',
       x = 'Volatile Acidity (g/L)',
       y = 'Alcohol (% by volume)',
       color = 'Quality Score') +
  theme_minimal()

grid.arrange(
  arrangeGrob(p1, p2, ncol = 2),
  p3,
  nrow = 2
)
```

Key observations from the plots:
1. Higher quality wines tend to have higher alcohol content
2. Lower volatile acidity correlates with higher quality
3. There's a slight negative correlation between volatile acidity and alcohol content

### Alcohol Content Distribution

```{r}
# Create density plot for alcohol content by quality
ggplot(wine_data, aes(x = alcohol, fill = as.factor(quality))) +
  geom_density(alpha = 0.5) +
  labs(title = "Distribution of Alcohol Content by Quality",
       x = "Alcohol (% by volume)",
       y = "Density",
       fill = "Quality Score")
```

The density plot reveals:
- Higher quality wines cluster around higher alcohol levels
- Significant overlap exists between quality groups
- Lower quality wines tend toward lower alcohol concentrations

### Model Diagnostics

```{r}
# Generate diagnostic plots for final model
par(mfrow = c(2, 2))
plot(model_significant3)
```

### Detailed Red Wine Analysis

```{r}
winequality_red <- read.csv("winequality-red.csv", header = TRUE)

features <- c("alcohol", "volatile.acidity", "fixed.acidity", "citric.acid", 
              "residual.sugar", "chlorides", "free.sulfur.dioxide", 
              "total.sulfur.dioxide", "density", "pH", "sulphates")

labels <- c("Alcohol", "Volatile Acidity", "Fixed Acidity", "Citric Acid", 
            "Residual Sugar", "Chlorides", "Free Sulfur Dioxide", 
            "Total Sulfur Dioxide", "Density", "pH", "Sulphates")

# Create boxplot function
create_boxplot <- function(feature, label, color) {
  ggplot(winequality_red, aes(x = factor(quality), y = .data[[feature]])) +
    geom_boxplot(fill = color) +
    labs(title = paste("Red Wine", label, "by Quality"),
         x = "Quality Score",
         y = label) +
    theme_minimal(base_size = 12)
}

plots <- mapply(create_boxplot, 
                features, 
                labels, 
                scales::hue_pal()(length(features)), 
                SIMPLIFY = FALSE)

# Create density plot for red wine
density_plot <- ggplot(winequality_red, 
                      aes(x = alcohol, fill = factor(quality))) +
  geom_density(alpha = 0.5) +
  labs(title = "Red Wine Quality Distribution by Alcohol Content",
       x = "Alcohol (% by volume)",
       y = "Density",
       fill = "Quality Score") +
  theme_minimal(base_size = 14)

# Display plots in groups
for (i in seq(1, length(plots), by = 4)) {
  grid.arrange(
    grobs = plots[i:min(i+3, length(plots))], 
    ncol = 2, nrow = 2, 
    top = paste("Red Wine Quality Attributes (Group", ceiling(i/4), ")")
  )
}

print(density_plot)
```

Key findings from red wine analysis:
1. Higher quality red wines consistently show:
   - Higher alcohol content
   - Lower volatile acidity
   - Higher sulphate levels
2. Chemical attributes show clear trends across quality levels
3. Alcohol content distribution varies significantly by quality rating

This comprehensive analysis strengthens our understanding of wine quality determinants and validates our model's focus on key chemical attributes.
