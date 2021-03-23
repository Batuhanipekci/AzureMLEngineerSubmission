# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Summary
**In 1-2 sentences, explain the problem statement: e.g "This dataset contains data about... we seek to predict..."**
The sample dataset "bankmarketing_train.csv" for the proof-of-concept of Hyper and AutoML is taken from https://archive.ics.uci.edu/ml/datasets/Bank+Marketing. The data contains the personal client attributes (age, job, marital status), loan details and duration, as well as macroeconomic indicators. The target column y is whether the client has subscribed a term deposit.

**In 1-2 sentences, explain the solution: e.g. "The best performing model was a ..."**
The best performing model was a LightBGM with an accuracy of 0.9170. The best model is a VotingEnsemble selected by the Azure AutoML, which yielded a better accuracy metric than the simple Logistic Regression and the optimized Logistic Regression using HyperDrive. As a sidenote, accuracy -although easily interpretable- is not a reliable measure in binary classification, especially high imbalanced dataset (The optimal models tend to have a bias against the class with more samples). Having said that, the AutoML solution VotingEnsemble has also very high weighted accuracy (0.9595) and weighted AUC (0.9492).

## Scikit-learn Pipeline
**Explain the pipeline architecture, including data, hyperparameter tuning, and classification algorithm.**

The pipeline architecture for HyperDrive optimization was a data preparation script, train.py which reads the data and splits the data into train and validation tests. The model was Logistic Regression, and the parameters are C (the inverse regularization parameter), and max_iter (maximum iteration of the regression optimization algorithm). Lower C, as well as, high max_iter might cause overfitting, so those variables should be carefully tuned. The trained model is then tested on a separate validation dataset.

**What are the benefits of the parameter sampler you chose?**

HyperDrive makes it easy to implement parameter sampling with an optimal early stopping policy. C is sampled from a uniform distribution (0,1), while max_iter was sampled from a list of 3 choices (10, 100, 500). The parameter sampler allowed us to do random search and identify the regions in the parameter space that would yield the best results.

**What are the benefits of the early stopping policy you chose?**

The early stopping policy (Bandit Policy) checks the optimization job every 2 iterations and prevented the tuning process going astray with the values that are not very contributing to the metric improvement. If the metric falls outside of the slack value range (top %10), the job is terminated, so that we can spend more time on the other hyperparameters that might matter.

## AutoML
**In 1-2 sentences, describe the model and hyperparameters generated by AutoML.**

AutoML has chosen the VotingEnsemble of  LightBGM, XGBoostClassifier, RandomForest, LogisticRegression, ExtremeRandomTrees with different hyperparameter configurations. The VotingEmsemble has taken the outputs of the algorithms weighted by their performances on the validation data. This combination yielded the best result. The validation has been done via 4-folds cross validation, which is much reliable than the single-shot validation set used in HyperDrive Logistic Regression.

## Pipeline comparison
**Compare the two models and their performance. What are the differences in accuracy? In architecture? If there was a difference, why do you think there was one?**

The HyperDrive Logistic Regression model was beaten by AutoML in terms of accuracy (0.9171, 0.9110). Also let's emphasize again that the measures of AutoML were obtained from cross validation, as opposed to a single validation set as in HyperDrive. This makes AutoML results more reliable. It is quite difficult to beat AutoML, as it has the power to combine all high performing models. Although, some can argue that Logistic Regression is a statistically based model, which might perform better in the long-run with different test sets. Other things aside Logistic Regression will always be a good benchmark to check AutoML model accuracy.

## Future work
**What are some areas of improvement for future experiments? Why might these improvements help the model?**

Further work is required for overcoming the class imbalance in the dataset, which was also detected by the AutoML pipeline. Subsampling or chosing a weighted primary metric to optimize should be considered. Also a deeper search on the hyperparameter space of the algorithms that were implemented by AutoML is advisable. Perhaps, some of the consistently underperforming models can be removed from the AutoML config and the hyperparameter optimization for the better performing ones can be prioritised.

