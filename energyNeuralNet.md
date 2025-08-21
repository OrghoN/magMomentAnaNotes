# NOvA Neutrino Energy Estimation with LSTM and Transformer Networks

## Introduction

This document summarises the development and analysis of Transformer-based neural networks for event classification and particle identification within the NOvA.
The focus is on how these networks improve upon previous approaches, particularly Convolutional Neural Networks (CNNs) and Long Short-Term Memory (LSTM) networks, and their potential for interpretability and further application.
The NOvA experiment aims to study neutrino oscillations by observing the disappearance of muon neutrinos and the appearance of electron neutrinos at a far detector, about 800km from the source.

## What are CNN, LSTM and Transformers

### CNN (Convolutional Neural Network):
A Convolutional Neural Network (CNN) is a type of deep learning model primarily used for processing grid-like data, such as images.
CNNs are designed to automatically and adaptively learn spatial hierarchies of features through layers of convolutional operations.
These operations apply filters (also known as kernels) to input data, allowing the model to detect patterns such as edges, textures, and more complex features as the data progresses through the layers.
CNNs are particularly effective for tasks like image classification, object detection, and segmentation due to their ability to capture spatial relationships within data.
The architecture typically consists of convolutional layers, pooling layers (for down-sampling), and fully connected layers, which enable the network to make predictions based on the learned features.

### LSTM (Long Short-Term Memory):
Long Short-Term Memory (LSTM) is a type of recurrent neural network (RNN) architecture designed to handle the vanishing gradient problem and better capture long-term dependencies in sequential data.
LSTMs use special units called memory cells, which maintain information over time through a set of gates (input, forget, and output gates).
These gates regulate the flow of information, allowing the model to decide which information to keep and which to discard as new data arrives.
This structure makes LSTMs highly effective for tasks like time-series prediction, natural language processing, and speech recognition, where the relationship between earlier and later inputs is crucial.
Unlike traditional RNNs, LSTMs can retain information across many time steps, which helps them model sequences with long-range dependencies.

### Transformer Network:
The Transformer network is a deep learning architecture that revolutionized natural language processing (NLP) tasks by focusing on self-attention mechanisms, which allow the model to weigh the importance of different words in a sequence, regardless of their position.
Unlike RNNs and LSTMs, which process sequences step by step, the Transformer processes all input data simultaneously (in parallel), making it more efficient.
The architecture consists of an encoder-decoder structure, with both parts utilizing self-attention layers to capture complex relationships in the data.
Transformers excel in tasks like machine translation, text generation, and language understanding due to their ability to model global dependencies and process large datasets quickly.
Popular models like BERT and GPT are based on Transformer architecture, showcasing its impact on NLP and beyond.

## Transition from CNNs to Transformers

Problem with CNNs: The primary challenge with CNNs is their difficulty in relating inputs (pixel images) to predicted classifications.
Motivation for Transformers: Researchers sought a more diagnosable network that can work with both reconstructed quantities and event display images.
This led to the use of attention-based Transformer networks.
The intention was to move beyond event classification to predicting event interaction mode as well as all prong particle types in an event.
Advantages of Transformers: Transformers offer permutation invariance, crucial for particle physics where the order of particles within an event is arbitrary.
Additionally, they are more flexible than LSTMs: "This model architecture is more flexible, easily handling many loss variables at once.

## NOvA Experiment and Data

- Experiment Setup: NOvA employs two detectors spaced 809 km apart to observe neutrino interactions.
The detectors are composed of cells filled with liquid scintillator.
- Neutrino Interactions: Neutrinos can interact via elastic scattering.
- Dataset: The transformer network is trained on 6.7 million simulated Monte Carlo events from the far detector.
These events include muon neutrino  CC, electron neutrino CC, NC, and cosmic ray muons.
- Event Reconstruction: The event images are split into "prongs," which are individual particle tracks or showers within an event.
NOvA’s prong reconstruction splits event image into several individual prong images.

There's also a 'slice' image representing the pixels not assigned to any prong.
Prong selection has cuts based on pt/p < 0.95 and a cosmic veto, to match previous CNN training.

## Transformer Network Architecture

- Prong Processing: The network uses a "Prong Architecture" comprised of four MobileNet layers followed by a Fully Connected Encoder, and a Prong embedding, for each prong in the event.
All Prong Classifiers share their weights.
- Transformer Encoder: The prong embeddings are input to the transformer encoder.
This attention mechanism calculates "queries," "keys," and "values," subsequently computing similarity scores and a weight matrix.
The output is the product of the weight matrix and values. Transformer block encodes input vector into “queries,” “keys,” and “values”.
- Summarizer: Outputs from the Transformer encoder are fed into a summarizer, which is also attention based.
Output of transformer encoders fed to summarizer.
- Classification: The transformer network predicts both the event interaction mode and the types of particles ("prongs") within each event.
- Loss Variables: The transformer can accommodate several loss variables, such as reconstructed neutrino and lepton energies, through the use of a Mean Absolute Percentage Error (MAPE) loss.

## Network Evaluation and Performance:

High Performance: The network shows high Area Under the Curve (AUC) values in the Receiver Operating Characteristic (ROC) curves for both event and particle classification.
Particle Classification: Similarly, particle classification ROC curves also show strong performance, with electron and muon classifications showing "AUC: 0.9684" and "0.9834" respectively.
Comparison to LSTMs: The Transformer model has demonstrated competitive performance in energy estimation when compared to LSTMs, but has the key advantage of easily accommodating multiple output variables and being more flexible in architecture.

## Interpretability:

- Importance of Interpretability: Researchers aim to "quantify relative impacts of reconstructed quantities vs event display images and systematically perturb prong image pixels to study how features of prong impact classification".
- Attention Scores: Attention scores are used to determine the importance of different parts of the input to event classification, demonstrating which prongs are important for different event types.
For example, the "leptonic prong is importance for charged current events".
- Pixel Gradients (Salience): Salience maps are computed to show how pixel values change the classification output, indicating which regions of the pixelmap positively or negatively contribute to the prediction of a particular class.
"Red, or positive, would mean more likely to predict the given class if more hits in the location".
- Integrated Importance Maps: Integrating along the width of the detector, importance maps are plotted as a function of distance from the vertex, providing insights on the shape of the track and shower profiles associated with different particles.

## Fine Tuning and Future Applications:
Feature Extraction: A new module has been created that extracts feature vectors (embeddings) from the transformer.
These can be used for other classification/regression applications without retraining entire network.
Reusing Embeddings: The raw outputs of the transformer can be used as abstract features for events and prongs, enabling exploration of new variables.

## Variable Definitions:


- Event-level:calE: Total energy of event.
- Prong-level:calE: Energy of a prong, from hits assigned to that prong.
- weightedCalE: Prong energy, with downweighting for hits shared with other prongs.
- dir: Direction vector of prong.
- nhit: Total number of hits in the prong.
- nhitx: Number of hits in the prong in the XZ view.
- nhity: Number of hits in the prong in the YZ view.
- len: Length of the prong
- nplane: Number of hit columns in the event display
- start: Starting position of prong
- gap: Distance between vertex and start

## Conclusion:

The development of Transformer networks for NOvA represents a significant advancement in event classification and particle identification.
These networks overcome limitations of previous methods, offering greater flexibility, interpretability, and performance.
The ability to extract embeddings and fine-tune the network for new variables opens possibilities for further applications within and beyond the NOvA experiment.
The ongoing research is focused on refining models and making detailed studies, and also comparing the new techniques with the more established LSTM based methods.



