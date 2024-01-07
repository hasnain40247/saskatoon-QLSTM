# Saskatoon QLSTM - A Quantum Neural Network utilized on a 30-year old time-series dataset.
## Dataset
QLSTM. The crux of the problem statement is to accom-
plish time-series weather-forecasting using a quantum lstm.
The first step is to gather the dataset from weatherstat.
Which offers real meteorological data of weather station at
Saskatoon John G. Diefenbaker Intl. Airport latitude 52.14,
longitude -106.69 collected from weatherstats website. The
Climate Daily/Forecast/Sun Saskatoon weather dataset from
weatherstats contains historical weather data for the city
of Saskatoon, Saskatchewan, Canada. The dataset includes
daily observations of various meteorological parameters,
such as temperature, dew point, windchill, relative humidity,
station pressure, sea pressure and wind speed. These data
points as mentioned before may not all be important as the
amount of correlation is varied. 

## Methdology

The weather dataset was first feature reduced since there were a myriad of features that served little to no benefit into the prediction of the target variable and additionally lead to overhead when passed to the QNN. The initial approach was to use PyGAD to facilitate genetic algorithm by encoding the features into genomes and using the base LSTM and the QLSTM to retrieve a fitness score over some generations. While this worked well in conjunction with the LSTM, it did not fare well with the QNN computationally. Another alternative to this was to use a popular library - SelectKBest to narrow down the feature list.

The proposed methodology utilizes the SelectKBest method
from scikit-learn library to select the most important features
from our dataset. We first drop the target variable from the
feature set and store the remaining features in a new variable
X. It fills any missing values in the feature set with the last
valid observation along each column. The method specifies
the value of k as 12, indicating that the top 12 features will
be selected by the SelectKBest method. The SelectKBest
method is initialized with the f regression score function
Figure 6. Graph of target variable
which computes the correlation between each feature and
the target variable, and returns the corresponding F-value
and p-value for each feature. The SelectKBest method se-
lects the top k features based on the highest F-values. The
fit transform method is used to fit the SelectKBest object
to the feature set X and target variable y, and returns a new
feature set that contains only the top k features selected by
the SelectKBest method.

Once the dataset's dimensions were reduced, the next step was to construct the LSTM and QLSTM and supply it a small subset of dataset to select a suitable optimizer, learning rate and the number of hidden units both for LSTM and QLSTM. Upon trial and error, the finalized optimizer used was Adagrad, the learning rates were 0.01 and 0.05 repectively with 16 hidden units each. This combination ensured the best performance on the subset of data.

The next step was to construct and explain the working of the QLSTM itself. 

## Flow-through of the QLSTM

The dataset is setup with a lookback - i.e. during the forward pass, the QLSTM, takes in a sequence of #n past data features and the resulting target variable of the future. 

The forward pass -
At each time step t, the input data x is concatenated with the previous hidden state (h_t).
The concatenated vector v_t is passed through a linear layer (self.clayer_in) to get y_t, which represents the features fed into the quantum variational circuits (VQC).
These serve as a feature representation to the quantum variational circuits. y_t, now that it is ready to enter the QNN, must first undergo processing. This is in essense done by passing it through a Hadamard gate which the VQC Nodes handle. The Hadamard gate applies an amplitude encoding operation such that the classical vetor is converted into the amplitudes or the phases of th Bloch sphere which represents the qubit. Any further operations would be rotational operations on this newly recieved quantum state. Parameters obtained from y_t are thus used to determine rotation angles in quantum gates - qml.RX, qml.RY, qml.RZ. Once the classical information has been quantum encoded, the qubits are then operated upon by randomly generated ansatz. In this case, we simulate a particular type of ansatz which consists of a CNOT gate and a 3-axis rotational gate, meant to operate on the individual qubits. This process is carried out for each of the gates of the LSTM.

The output from each quantum circuit - (computed quantum properties or expectation values) is then processed further through classical neural layer (self.clayer_out).
This essentially helps transform the quantum outputs into its classical format, suitable for the remaining pass of the LSTM.
