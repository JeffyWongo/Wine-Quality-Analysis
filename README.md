### Wine Quality Analysis  

Creating interaction and second-order models for Wine Quality Analysis  

[Project Slides](https://docs.google.com/presentation/d/1erbW4fPmB2GrKnDtiREE0j_FVKpiZNxoFrAHTbloMBo/edit?usp=sharing)  

---

## My Dataset  
I worked with two datasets, *wine-quality-white.csv* and *winequality-red.csv*, which contain 4,898 and 1,599 samples, respectively. These datasets include physicochemical properties and sensory quality ratings for Portuguese “Vinho Verde” wines. Each dataset has 11 physicochemical variables—fixed acidity, volatile acidity, citric acid, residual sugar, chlorides, free sulfur dioxide, total sulfur dioxide, density, pH, sulfates, and alcohol level—alongside a quality score rated on a scale from 0 to 10.  

The quality score serves as the response variable, reflecting the combined influence of the physicochemical attributes on the wine's sensory profile. To ensure a comprehensive analysis, the data were collected using a dual-method approach: laboratory measurements provided objective data, while blind sensory evaluations by professional tasters added subjective assessments. This combination allowed me to explore the relationship between chemical composition and perceived wine quality, providing insights for scientific and practical applications.  

---

## Research Question  
My goal was to investigate the connection between a wine’s quality and its chemical attributes, such as fixed acidity, volatile acidity, pH, chlorides, and residual sugar. I aimed to determine whether wine quality could be predicted from these properties and to identify which chemical factors most significantly influence the quality rating.  

I hypothesized that higher quality ratings would correlate with moderate acidity, lower residual sugar levels, and appropriate alcohol content. I also expected balanced levels of sulfur dioxide, pH, and chlorides to positively affect wine quality.  

---

## Interesting Findings  
- **Volatile Acidity**: I found that higher volatile acidity strongly decreases wine quality due to its association with spoilage and vinegar-like aromas.  
- **Density and Residual Sugar**: These variables are strongly correlated—during fermentation, as sugar converts to alcohol, the wine's density decreases.  
- **Red vs. White Wines**: Red wines generally received higher quality ratings than white wines in this dataset.  
- **Residual Sugar and Alcohol**: These are negatively correlated; wines with higher alcohol content (>12.5% ABV) tend to have no residual sweetness (dry wines).  

---

## Conclusion  
- Wines with higher residual sugar often have better quality, indicating perfectly ripened grapes.  
- Alcohol content has a strong positive correlation with quality—more sugar during fermentation results in higher alcohol content.  
- High volatile acidity generally lowers wine quality.  
- Small amounts of salt content (chlorides) can enhance a wine's flavor.  

However, I noted that:  
- There may be additional variables influencing wine quality.  
- The dataset contains significantly more white wine samples than red, which may skew results.  
- Quality ratings are inherently subjective.  
- Future work could involve exploring external variables or refining the model by focusing on fewer key factors.  
