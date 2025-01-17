#  Extensions Explained

## Performance of Default
The Default method was to always choose the sentence translated by the first Tucker in each task.

	Result: 0.5005 		


## Description of Baseline (baseline-solution)
For the baseline-solution system, we ran a 150-tree Random Forest (sklearn.ensemble.RandomForestClassifier) on data obtained from Turker features in `survey.tsv`. As a pre-processing step, we converted "YES/NO" features to binary-valued ones, and replaced "UNKNOWN" values with `float('NaN')`. To generate labels for supervised training, we found, for each source sentence, the index of the translation with the highest average BLEU score computed against each of the reference sentences. To generate the features, we simply mapped each Turker ID to the corresponding attributes in the pre-processed survey data. The random forest is trained on 20% of the full dataset, and generates predictions for the rest of the data. The baseline-solution system achieves a score of 0.530 on the full data set, significantly better than that achieved by the default. For comparison, the oracle system, which generates labels in the manner above for the entire data set (with no training necessary), achieved a score of 0.581.


## Description of Extension1
In this extension, I have used Machine Learning approach using AdaBoost Regression (sklearn.ensemble.AdaBoostRegressor). I have generated the following features: 
	
	- Worker-based Features (binary feature for each Turk worker, total 51, to take into the consideration of the quality of each Turker's translations. For instance, if Turker A's qualities were constantly high, then the algorithm would give higher weight for Turker A. However, if Turker B's qualities were constantly low, then the algorithm would give lower weight for Turker B.)
	- Worker-based Features (binary feature for native English, native Urdu, currently living in India, currently living in Pakistan)
	- Worker-based Features (numeric feature for the number of years speaking English and Urdu)

However, some data were missing (reported as 'UNKNOWN'), and some were absurd (e.g., the response for the number of years of speaking English and Urdu were 100). For such cases, I have used the following ways to replace the missing / absurd data with reasonable values.
	
	- If the native English / native Urdu values were missing: replaced 'UNKNOWN' with 0.5 
	- If the number of years of speaking English and Urdu were absurd: calculated the average of the number of years of speaking English and Urdu from the provided sample (using other Turkers' responses), and replaced such value with the sample mean.
	- If the number of years of speaking English and Urdu were missing: similarly, replaced 'UNKNOWN' value with the sample mean

Upon completion of Worker-based features, I have constructed two sentence-level features. 
	
	- Proportion of sentence length: length of candidate sentence (by Turker) divded by length of the source sentence (original Urdu sentence). My naive idea was that two sentences should have a similar length.
	- Proportion of number of unique tokens: number of candidate sentence's English token divided by the number of unique English tokens from all four candidate sentences.

Once all features are generated, I have used AdaBoost Regressor to train. The reason for choosing AdaBoost is because of its characteristics. AdaBoost Algorithm generates n classifier for each task. For instance, I have used 1000 AdaBoost model for each task, and the Machine Learning algorithm takes average of each task to come up with the finalized weight for each feature. This process is repeated for the entire number of training data. I have used this approach since generating sub-classifer n times would allow the classifer to converge to the 'true mean.' Based on this notion, I have implemented AdaBoost algorithm with 1000 iterations on each task, and I have employed the exponential loss function.

Since there was no explicit y labels for Machine Learning, I have manually engineered y-value for each candidate sentence using a metric that evaluates the quality of each candidate sentence. BLEU score is used to quantitiatively measure the quality of each document by comparing it against with the reference document. In this task, since the provided reference document and candidate document are in sentence format, I decided to use smoothed-BLEU measure to evaluate the quality of each candidate sentence against a group of reference sentences. In the training process, my goal was to correctly predict the smoothed-BLEU score of each candidate sentence using the worker-level features and sentence level features (total 59). Using these features along with y-hat (in this case, smoothed-BLEU score I have calculated using known four reference sentences (LDC1, LDC2, LDC3, LDC4) against each candidate sentence), I trained the AdaBoost Regression Algorithm to predict y-hat (smoothed-BLEU score).

After the training is done, AdaBoostRegressor model is used to predict y-hat values (smoothed-BLEU score) for 7168 cases (total 1792 source sentence, each source sentence has 4 candidate sentences). Once y-hat values are calculated for all 7168 cases, the four every 4 candidate sentences, the candidate sentence with highest y-hat value is chosen to be the most acceptable sentence generated by Turker in each task. In total, 1792 sentences are returned as solutions.


## Performance of Extension1
	
	Result (n = 300, exponential): 0.5398
	Result (n = 500, exponential): 0.5399 //Best Performance
	Result (n = 1000, exponential): 0.5395
	Result (n = 1500, exponential): 0.5390



## Description of Extension2
Extension2 adds additional features to improve the baseline model, which used worker-level features only. Using the language model provided in Homework 2, I added the log probability of each translated sentence as a feature. Following Zaidan and Callison-Burch, I also added the ratio of the length of each translated sentence to the average length of the reference sentences, and vice versa, as a pair of sentence-level features. (The intuition is that good translations are likely to have similar length to the reference sentences.) As before, the labels for supervised training were obtained by computing the index of the best translation sentence, i.e., the one with the highest average BLEU score against the set of reference sentences. Using only these additional features, the same Random Forest classifier (with 150 trees) performed significantly better than the baseline, achieving a score of 0.543.

