# Stock Price Movement Prediction Using FFNN CNN and RNN in Tensorflow

This project aims to predict the movement of future trading price of Netflix (NFLX) stock using transaction data on January 3, 2017 from the Limit Order Book (LOB). Stationary features were created to overcome autocorrelation and reduce noises of the time series data. For this project, a random forest model was built as baseline and three types of neural network models were develpted and compared: Feed Forward Neural Networks (FFNN), Convolutional Neural Networks (CNN), and Recurrent Neural Networks (RNN).  The models were compared based on not only the accuracy, but also other metrics such as f1-score and Cohen’s Kappa.

## Data, Feature Engineering and Labelling

**Data**

The raw data for this project is the high frequency stock market data, which contains information on all the sell and buy transactions for stocks traded on NASDAQ on January 3, 2017. Example fields included in the data are timestamp, order id, book event type, price, quantity and side. For this project, the data for NFLX are used. There are 477,239 observations, including Add, Modify, Trade and Cancel event types. To predict future movement of trading price, only traded events are included which has 19,599 observations. This data is splitted into train and test sets at a ratio of 7:3 from the time horizon due to the time-series nature of the data. Table 1 shows the details about the structures for each set and the labeling process will be explained later. 

<img width="665" alt="table1_train_test_sets_structure" src="https://user-images.githubusercontent.com/42804316/57708270-0cb55500-7637-11e9-86d5-0ca02547a17c.png">

**Feature Engineering**

To mitigate autocorrelation and reduce noise in time series data, several features were created from existing features. For instance, a mixed moving average was calculated using moving averages in the windows of 3, 6, 12, and 24. Other features created include the count of rising prices among the past 10 traded transactions, average quantity for the past transactions and price difference between every other 5 transactions.  See Table 2 for details about each feature used in the models. 

<img width="797" alt="table2_feature_engineering" src="https://user-images.githubusercontent.com/42804316/57708579-949b5f00-7637-11e9-8129-046e222a7efe.png">

Then, the numerical features were standardized with a mean of 0 and a standard deviation of 1 to ensure they are treated the same in the models regardless of their units.. Lastly, to ensure the input data is consistent for different types of models, the train and test sets were reshaped such that one instance (or one row in the data frame) corresponded to all 50 transactions in one prediction window. For instance, we would have 400 features (8 times 50) at transaction T49 for predicting the price for transaction T50.

**Labelling**

Ground truth labels were generated to indicate the movement of future trading prices. A window size of 50 was defined for the current transaction. Specifically, at transaction T49, we are interested in predicting the price for transaction T50, with T denoting transaction. The average price for the transactions within the prediction window (avg_PT0 to T49)  is compared with the price for the transaction right after (PT50). To reduce noise, we determined a threshold above which a price change would be labeled as an upward or downward change. A threshold of 0.03% was chosen to ensure the classes are balanced in the train set. See Table 3 for more details about the labeling process. 

<img width="771" alt="table3_labelling" src="https://user-images.githubusercontent.com/42804316/57709292-ba753380-7638-11e9-8c69-c8d09bd5a65c.png">

## Architectures of Neural Network Models

**Baseline Model: Random Forest**

Entropy (information gain) was used as the criterion to assess the quality of each split for each tree in the random forest model. The performance of the model was evaluated using average precision, average recall, f1-score and Cohen’s kappa. The hyper parameters were tuned using a grid search, and the optimal model had 300 trees and 30 minimum samples at leaf nodes. 

**Feed Forward Neural Networks (FFNN)**

The best FFNN has two hidden layers with ReLU as the activation functions. The first hidden layer has 300 neurons and the second layer has 100 neurons. The output layer outputs the logits and then goes through the softmax activation function. The cross entropy is utilized as the loss function and adam optimizer is adopted to optimized the parameters (weights and bias) of the model.. The network takes 400 features as the number of inputs for each instance, and outputs 3 probabilities for each class. The predicted class is determined by the highest probability among the three. During training session, the entire train set was splitted into train and validation sets sequentially at a ratio of 8:2. Specifically, the first 10,935 instances were in the train set while the remaining 2,734 instances were in the validation set. 

<img width="60%" alt="tensorboard_graph" src="https://user-images.githubusercontent.com/42804316/57709888-d927fa00-7639-11e9-967e-7ba0e0046907.png"><img width="40%" alt="simplified_flowchart" src="https://user-images.githubusercontent.com/42804316/57709910-e1803500-7639-11e9-9d7b-6936507a54b2.png"><br /> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fig 1. Tensorboard Computational Graph (left) and Simplified Flowchart (right)

**Convolutional Neural Networks (CNN)**

It consists of one input layer, two convolutional layers, one pooling layers, one fully connected layer accompanied by one dropout layer,  and one output layer. The number of inputs is 400 which is the product of the height (50) and width (8) of one grid. Then, the input is further reshaped to accommodate the 2-D convolution which has a height of 50, a width of 8 and channels of 1. For two convolutional layers, 36 filters and 72 filters are utilized, respectively. For both layers, the kernel size is two, stride is one, padding is SAME, Relu is  the activation function. In the max pooling, the dimensionality of the feature maps are reduced to the half both for height and width using a kernel of two by two and a stride of two by two. The outputs from the pooling layer are reshaped back to 1-d vector for the fully connected layer, after which the dropout layer (dropout rate: 0.5) is applied to decrease overfitting. Finally, the cross entropy and the Adam optimizer are used to calculate the loss and train the model. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img width="270" alt="tensorboard_graph" src="https://user-images.githubusercontent.com/42804316/57710492-0a54fa00-763b-11e9-972b-8c6598f4b95c.png">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img width="280" alt="simplified_graph" src="https://user-images.githubusercontent.com/42804316/57710502-117c0800-763b-11e9-8b6f-39b4b38eaee7.png"><br /> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fig 2. Tensorboard Computational Graph (left) and Simplified Flowchart (right)

**Recurrent Neural Networks (RNN)**

The best RNN adopted GRU (gated recurrent units) cell. The number of units for each cell is 150, the time steps are 50, and the size of the input is 8. Then, the returned value from the last time step is fed into a dense layer to get the logits, which utilizes Tanh activation function. The obtained logits further go through the softmax activation function to get the 3 probability for each class. The highest probability corresponds to the predicted class. The cross entropy is used to compute the loss and the adam optimizer is used to optimize the parameters of the model.

<img width="35%" alt="tensorboard graph" src="https://user-images.githubusercontent.com/42804316/57712135-1abaa400-763e-11e9-9152-83c5e689e537.png"><img width="65%" alt="simplified_flowchart" src="https://user-images.githubusercontent.com/42804316/57712157-25753900-763e-11e9-89fe-102befe6daa7.png"><br /> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fig 3. Tensorboard Computational Graph (left) and Simplified Flowchart (right)

## Results

Figure 4 shows the details of the results for the metrics of random forests. They are based on the weighted averages of the metrics, which takes into consideration of the weights of each class. Figure 5 shows the detailed results for the FFNN trained using a learning rate of 0.0003, a batch size of 64 and 200 epochs with early stopping. 

<img width="380" alt="Fig4_RF" src="https://user-images.githubusercontent.com/42804316/57713902-5f940a00-7641-11e9-8314-e50c97e26243.png">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img width="352" alt="Fig5_FFNN" src="https://user-images.githubusercontent.com/42804316/57713910-63279100-7641-11e9-8816-68e9cca650ec.png">

 Figure 6 shows the detailed results for the CNN trained using a learning rate 0.001, a batch size of 50 and 10 epochs. Figure 7 shows the detailed results for the RNN trained using a learning rate of 0.001, a batch size of 50 and 10 epochs. 

<img width="351" alt="Fig6_CNN" src="https://user-images.githubusercontent.com/42804316/57713941-6c186280-7641-11e9-9b9e-dae789553bcf.png">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img width="354" alt="Fig7_RNN" src="https://user-images.githubusercontent.com/42804316/57713945-6f135300-7641-11e9-97f0-0e25f77edbea.png">

The prediction results for the four models are summarized in Table 4 below. The test data is imbalanced with the 0 class having a higher proportion, so the Cohen’s Kappa instead of accuracy is focused to evaluate model performance. It’s shown that all neural network models, along with the baseline random forest model, have similar Cohen’s Kappa values. The Cohen’s Kappa values indicate that our models have predictive power, rather than assigning labels randomly. In addition, improvements were also seen for other metrics. 

<img width="796" alt="table4_results_summary" src="https://user-images.githubusercontent.com/42804316/57714277-29a35580-7642-11e9-95d4-a835cda2f08c.png">

In conclusion, a baseline model and three neural network models: an FFNN, a CNN and a RNN were developed to predict the movement of the trading price of NFLX using data from one day. All models are working as they are able to predict the classes with above 70% accuracy. RNN is the best model with a Cohen’s Kappa of 0.53.
