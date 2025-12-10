# CustomBranchPredictor
CSE240A

The Code includes 
- base gshare branch predictor
- Alpha 21264 tournament branch predictor
- A Custom hybrid perceptron-based gshare predictor 

# Global history perceptron predictor with PC⊕GHR Indexing to prevent aliasing

I have chosen the Perceptron branch predictor as branch prediction is learning from past data and ML perceptron does a good job.··
It uses the 10bit lower index of the PC XORed with GHR to index into the table of 2^10 rows.··
Each row contains 57 weights + 1 bias weight.··
Y prediction is done by multiplying and accumulating the weights (wi) in the index with the ghr (xi) of that index.··
If Y predicted > 0 then its TAKEN else NOT TAKEN.··
Whenever i have a misprediction or the confidence is lower than the threshold i train the weights.··
Whenever we train the bias weight is incremented or decremented by 1 according to the outcome sign. If the outcome is not taken then the weight should decrease and vice versa.··
The weights is incremented or decremented by xi*outcome(sign).··
Perceptron threshold is taken as theta = 1.93 * H + 14 from the paper - Dynamic Branch Prediction with Perceptrons” Daniel A. Jiménez and Calvin Lin··

## Structure
The predictor maintains a table of 2^7 = 128 perceptrons, each containing:··
57 history weights (w₁…w₅₂)··
1 bias weight (w₀)··
Indexing··
index = (PC >> 2) ⊕ GHR··
Prediction··
y = w0 +  Σ ( wi * xi )   for i = 1..61··
Where ··
xi = +1 if the i-th most recent branch was TAKEN··
xi = −1 if NOT TAKEN	··
w0 is the bias weight··
wi is weight correlated with past branch i step ago,··
y >= 0 → predict TAKEN  ··
y <  0 → predict NOT TAKEN··
Training
The perceptron is trained only when:
The prediction is wrong 
or the confidence is low (|y| <= θ)
This prevents overtraining and ensures stability.
The training updation done
w0 ← w0 + t
wi ← wi + t * xi
Where-
w0 is bias weight
wi is the learned weight that represents how strongly the branch’s outcome from i steps ago (the i-th global history bit) influences the current branch's behavior.
xi is the ghr ith branch
Where t is 1 if outcome is Taken and t is -1 if outcome is not taken

## Design Choices
### Why Perceptron?
Perceptrons give long-range correlations and can accurately is very good at  learning from past data.
Why 57 history bits?
Tried increasing history length but there was no significant improvement. This provides the best accuracy/complexity balance and follows established results from perceptron research. H=57 captures complex branch relationships without overfitting.
Why PC⊕GHR indexing?
Using only PC bits caused aliasing and the accuracy was less.
 PC⊕GHR distributes branches across rows giving more accuracy.
Why the HPCA-2001 threshold formula?
The θ formula was derived analytically to prevent runaway confidence and allow stable learning.

### 2) You should include a table which shows the performance of the tournament predictor as well as your custom predictor for all the given traces. 


| Benchmark | Misprediction Rate (Custom Perceptron) | Misprediction Rate (Tournament) |
|-----------|----------------------------------------|----------------------------------|
| Blender   | 27.701                                 | 29.228                           |
| Leela     | 82.014                                 | 93.025                           |
| GCC       | 6.860                                  | 14.066                           |
| Cam4      | 6.528                                  | 6.547                            |




### 3) You should also include a table which shows your total hardware budget usage.

Custom Hardware Budget

The only hardware consuming structure is Perceptron weights table 

Perceptron Weights Table Size
2**7(number of rows) * 57(weights per pc index row) * 8bits(weight)= 57KBits



