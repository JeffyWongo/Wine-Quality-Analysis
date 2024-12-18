# Wine-Quality-Analysis
Creating interaction and second-order models for Wine Quality Analysis

[Project Slides](https://docs.google.com/presentation/d/1erbW4fPmB2GrKnDtiREE0j_FVKpiZNxoFrAHTbloMBo/edit?usp=sharing)

## Our Dataset: 
The dataset we are using is one of the datasets given to us: wine-quality-white.csv and winequality-red.csv 4898 observations with 11 variables. The dataset consists of physicochemical properties and sensory quality ratings for Portuguese “Vinho Verde” wines, including a red wine dataset with 1,599 samples and a white wine dataset with 4,898 samples. Each dataset includes 11 physicochemical factors—fixed acidity, volatile acidity, citric acid, residual sugar, chlorides, free sulfur dioxide, total sulfur dioxide, density, pH, sulfates, and alcohol level—all of which contribute to sensory quality ratings on a discrete scale from 0 to 10. While all physicochemical factors are measured as continuous variables, the final quality score is treated as a discrete variable, serving as the dependent variable or response variable that determines the overall quality of the wine. This score reflects the combined influence of the physicochemical attributes on the wine's sensory profile.
The dataset is collected using a dual-method approach to ensure a comprehensive analysis of wine quality. Laboratory measurements are performed on the eleven physicochemical variables, offering precise and consistent data. Sensory evaluations, on the other hand, are conducted through blind taste tests performed by professional tasters, who assess the wines without considering external factors such as brand or price. This thorough combination of objective laboratory data and subjective sensory analysis creates a reliable dataset for exploring the relationship between a wine’s chemical composition and its perceived quality, providing valuable insights for both mathematical study, scientific study, and practical applications.
Research Question:

In this dataset, we are looking for the connection between a wine’s quality vs its other attributes, such as fixed acidity, volatile acidity, pH, chlorides, residual sugar, etc. We want to discover if we can use the wine’s contents to predict its quality. Specifically, which chemical properties most significantly influence the quality rating of the wine? In this research we also hope to discover how important acidity plays a role in the quality of wine, if there is any correlation at all. Our hypothesis is higher quality ratings are expected to correlate with moderate levels of acidity, lower levels of residual sugar, and appropriate alcohol content. Additionally, balanced levels of sulfur dioxide, pH, and chlorides may positively affect the quality as well.

## Interesting Findings
- Higher volatile acidity strongly decreases wine quality
    - Association with spoilage and vinegar-like aromas
- Density and residual sugar are strongly correlated
    - As sugar is converted into alcohol during fermentation, the density of the wine decreases.
- Red wines tend to have higher quality ratings compared to white wines in this dataset
- Residual sugar and alcohol levels are strongly negatively correlated, due to higher ABV ranges (>12.5%) having no sweetness (dry wine)

## Conclusion
- Wines with more residual sugar have higher quality	
    - Perfectly ripened grapes
- Alcohol content has strong correlation with quality
    - More sugar = more alcohol during fermentation
- Wines with high volatile acidity generally have lower quality
- Small amounts of salt content (chlorides) in wine enhance flavor

- There could be other variables that explain variance in the wine
- Significantly more white wine samples than red wine
- Quality rating is subjective
- Explore other variables outside of the dataset
- Experiment with less variables in the final model

