[Lire ce README en français](README.fr.md)
# Introduction to neural networks with NumPy

This repository gathers Python projects implemented only with NumPy. No deep learning library (PyTorch, Keras, scikit-learn, etc.) does the work for us: forward propagation, backpropagation, gradient descent, activation functions and loss functions are all coded entirely by hand.

The goal here is not performance but understanding what is going on.

---

## Table of contents

1. [Handwritten digit recognition (MNIST)](#1--handwritten-digit-recognition)
2. [Network that learns to play Connect 4](#2--network-that-learns-to-play-connect-4)
   
---

## 1 · Handwritten digit recognition

First project: a fully connected neural network that recognizes handwritten digits from 0 to 9.

### Data
The network is trained on the **MNIST** image dataset. The TensorFlow library is used only to load the dataset (never to build or train the network). Each image is $28 \times 28$ pixels in grayscale.

### Architecture

| Layer | Size | Role |
| :--- | :--- | :--- |
| **Input** | 784 neurons ($28 \times 28$ flattened) | One neuron per pixel |
| **Hidden** | 64 neurons | Pattern extraction |
| **Output** | 10 neurons | One neuron per digit (0 to 9) |

`784 pixels` ──► `[ 64 neurons ]` ──► `[ 10 neurons ]` ──► `predicted digit`

### How it works
1. **Forward propagation**: the image goes through the layers (matrix product + bias). The activation function used is ReLU.
2. **Error computation**: mean squared error between the prediction and the true value. 
   * *Example prediction*: `[0.08, 0.07, 0.12, 0.14, 0.12, 0.09, 0.13, 0.13, 0.06, 0.06]` (maximum prediction at index 3, i.e. the digit 3).
   * *Actual target*: `[0, 0, 0, 0, 0, 0, 0, 0, 1, 0]` (the true digit is an 8).
1. **Backpropagation (**: the gradient is computed layer by layer, entirely by hand.
2. **Update**: adjustment of the weights and biases by gradient descent.

---

## 2 · Network that learns to play Connect 4

Second project, much more ambitious: make a neural network learn to play Connect 4, without any strategy rule injected at the start.

### Modeling
* The Connect 4 board is modeled as an array of 42 cells (7 columns and 6 rows).
* The rules and the workings of the game are implemented through Python functions.
* The network outputs the column to play for the next move.

### Compared architectures
Two network architectures are tested and compared:

| Layers        | Network A                  | Network B                  |
| :------------ | :------------------------- | :------------------------- |
| **Input**     | 42 neurons (1 per cell)    | 42 neurons (1 per cell)    |
| **Inner 1**   | 64 neurons                 | 64 neurons                 |
| **Inner 2**   | -                          | 32 neurons                 |
| **Output**    | 7 neurons (1 per column)   | 7 neurons (1 per column)   |

* **Activation functions**: `ReLU` for the hidden layers, `Softmax` for the output layer.

### Game modes for training
* **Network vs random**: the opponent plays random moves.
* **Network vs itself**: the network plays against itself and learns from its own games.

### Learning methods
Three approaches, from the simplest to the most refined, were tested:

1. **a) Simple learning (Win / Loss)**
   * At the end of the game, if the network wins, all its moves are reinforced (the probability of the move played is pushed towards 1). If it loses, all its moves are penalized (towards -1).
   * Limitation: The network learns no fine tactics. All the moves of a won game are rewarded, even the bad ones. A simplistic technique such as stacking 4 pieces in the same column is over-reinforced. Result: no defense, no global strategy.
1. **b) Reward-based learning**
   * Each move receives a score computed from tactical criteria (aligning pieces, blocking the opponent). This score weights the impact of the gradient. Each decision is evaluated individually.
3. **c) Tactical learning (Supervised)**
   * For each move: the best theoretical move is computed according to the scoring system, the error between the move actually played and this best move is measured, then backpropagated. The network directly learns the "right move" to play.

### Problem detected:
 When training against itself, the network ends up adopting the trivial strategy of stacking 4 pieces in the same column without countering the opponent.
 
* **Possible cause**: Since it plays against itself (and is bad at the beginning), stacking is the fastest way to find a winning pattern in 4 moves because its opponent (itself ) has the same strategy and therefore does not prevent it from stacking . It therefore judges this strategy to be optimal.

* **Possible solutions**: Inject more random exploration to explore new strategies, and also because a random move can counter the stacking and thus force the network to favor another strategy.

###  Improvements made

* **Random exploration**: during training, the network does not always blindly choose the best estimated move, it allows itself some randomness to discover new approaches.

* **Style customization**: by modifying the score weightings, the model can be steered towards a more offensive or more defensive style.
