# Deep Learning Model Comparison: 
## CNN/CVN, LSTM, and Transformer

This document compares the technical architectures, advantages, and disadvantages of Convolutional Neural Networks (often referred to as CVN in specific contexts), Long Short-Term Memory (LSTM) networks, and Transformer models in the context of neutrino physics experiments like NOvA, and examines the recent shift toward using Transformer models.

### Technical Comparison of Model Architectures

| Feature | CNN/CVN | LSTM | Transformer |
| :--- | :--- | :--- | :--- |
| **Primary Input Format** | Pixel-mapped views (2D images) of detector hits (e.g., XZ and YZ views). | Reconstructed features of sequential data (prongs, slices), where the number of prongs is variable. | Reconstructed features of sequential data (prongs, slices).|

| **Core Mechanism** | Convolutional layers that extract spatial and feature information from images, followed by fully connected layers. | **Recurrent Structure** that processes sequential input $x_t$ based on the previous hidden state $h_{t-1}$.
Utilizes a **memory cell** and **gating mechanisms** (input, forget, output gates). | **Self-Attention Mechanism** (introduced by Vaswani et al.) that processes and captures relationships between all elements in a sequence simultaneously.
Typically uses a stack of encoder layers. |
| **Handling of Sequence** | Not designed for sequence data; operates primarily on spatial locality in image data. | Sequential processing, where the current hidden state $h_t$ is updated based on $x_t$ and $h_{t-1}$. | Parallel processing of all input elements; sequence length is fixed by zero-padding or truncation during batch training (e.g., $Pmax=10$ prongs in NOvA). |
| **Output** | Set of scores (probabilities) representing the event class (e.g., $νµ$ CC, NC, cosmic ray). | Predicts physical quantities, such as the energy of the incoming neutrino and the outgoing muon (for $νµ$ CC events). | Predicts various target quantities, including $νµ$ and muon energies. |
| **Order Dependency** | N/A (Spatial inputs). | Highly order-dependent due to sequential processing.
| **Order-invariant** because sum pooling is applied over the encoder outputs across the prong dimension, and positional encoding is not applied to prongs. |

### Technical Aspects

#### Convolutional Visual Network (CVN)

In NOvA, the CVN (Convolutional Visual Network, sometimes referred to as CNN) model is primarily used for **event classification**.

*   **Architecture Flow:** The model takes two pixel-mapped views (XZ and YZ projections) of detector hits as input.
Each view is processed independently through multiple convolutional layers, which are responsible for extracting localized features.
The outputs from these two convolutional stacks are then combined with other data, such as the reconstructed interaction vertex position, and passed through a series of fully connected layers to refine the classification decision.
*   **Application:** The final layer outputs scores, typically ranging from 0 to 1, representing the probability that the event belongs to different classes, such as $νµ$ charged-current (CC), neutral-current (NC), or cosmic-ray backgrounds.
A related model, ProngCVN, uses convolutional layers for classifying individual prongs (particles) within an event.

#### Long Short-Term Memory (LSTM)

LSTM networks are an extension of Recurrent Neural Networks (RNNs) specifically designed to model sequential data where long-range dependencies are important.

*   **RNN Context:** Standard RNNs process data sequentially, updating a hidden state $h_t$ based on the current input $x_t$ and the previous hidden state $h_{t-1}$.
However, they struggle with long-range dependencies because the backpropagation of gradients over many steps can lead to **vanishing or exploding gradients**.
*   **Gating Mechanism:** LSTMs address this by incorporating a **memory cell** and a **gating mechanism** to regulate information flow.
The gates—specifically the input gate, forget gate, and output gate—control three operations: how new information is added to the memory cell, how existing information is removed, and what information is extracted from the cell to form the output and next hidden state.
This allows the LSTM to maintain information over long sequences more effectively.
*   **NOvA Implementation:** The LSTM Energy Estimator processes prong features (3D and 2D prongs, sorted by length) and fixed-size slice information.
The sorted prong features are transformed using linear and Batch Normalization layers before being passed into the LSTM layer.
The LSTM output is then merged with slice features and passed through a final Multi-Layer Perceptron (MLP) to predict energies.

#### Transformer

The Transformer architecture bypasses the sequential nature of RNNs entirely, relying solely on the self-attention mechanism to process relationships across the entire input sequence simultaneously.

*   **Self-Attention:** This mechanism allows each element in a sequence (e.g., a prong) to "attend" to every other element, calculating the relationship weight regardless of their relative positions.
This enables the model to effectively capture global relationships and dependencies, especially over long ranges.
*   **Architecture Flow:** In NOvA's Transformer Energy Estimator (TransformerEE), 3D prong features are processed through a stack of 12 transformer encoder layers.
Slice information is handled separately by an MLP.
*   **Sequence Handling:** Since the architecture is designed for parallel processing, for batch training, events with fewer than the maximum prong count ($P_{max}$) are **padded with zeros**, and those exceeding $P_{max}$ are truncated.
Crucially, the outputs from the transformer encoders are combined using **sum pooling** over the sequence (prong) dimension.
This application of sum pooling, combined with the lack of applied positional encoding for prongs, ensures that the final output is **invariant to the order of the prongs**.

### Advantages/Disadvantages

| Model | Advantages | Disadvantages |
| :--- | :--- | :--- |
| **CNN/CVN** | Excellent for visual/spatial feature extraction; provides high-accuracy event classification.| Less effective for representing variable-length sequential data (like particle prongs). |
| **LSTM** | Capable of learning both short-range and long-range dependencies better than standard RNNs due to gating mechanisms. Was the widely adopted architecture for sequential tasks. | Struggles with extremely long-range dependencies compared to Transformers. Prone to issues like the "Energy Crisis" (two-peak patterns observed in $νµ$ CC energy prediction). |
| **Transformer** | Highly effective at capturing **global relationships** and processing long sequences due to parallel self-attention.
Performance tends to improve significantly with larger sequence lengths (many prongs).
**Order-invariant** when used with sum pooling on prongs. | Requires input sequence length constraint (padding/truncation) for batch training. |

The idea that Transformers would replace LSTMs stems from the inherent advantages of the self-attention mechanism in capturing dependencies across sequential data and processing information in parallel, especially as datasets grow large.

In the context of the NOvA experiment, the timeline for replacing the LSTM Energy Estimator (LSTM EE) with the Transformer Energy Estimator (TransformerEE) for $νµ$ CC interactions was accelerated due to a specific performance failure:

1.  **Prior State:** The LSTM Energy Estimator was the currently implemented and widely adopted model for sequential data tasks, like energy estimation based on reconstructed prongs.
2.  **The "Energy Crisis" (Early 2024):** In February 2024, NOvA collaborators discovered the "Energy Crisis" within the LSTM EE model.
This crisis was characterized by **strange two-peak patterns** in the $νµ$ CC predicted energy spectra when analyzed in different hadronic quantiles, particularly noticeable in the first hadronic quantile.
Although mitigation efforts (like increasing the batch size) were tried, the patterns persisted.
3.  **Replacement Decision:** Due to this persistent issue, the collaborators decided to replace the LSTM EE with the Transformer Energy Estimator (TransformerEE).
The TransformerEE was tested and was found **not to exhibit these two-peak patterns** in $νµ$ CC spectra.
4.  **Future Outlook:** The Transformer model has demonstrated promising performance on NOvA muon neutrino event data, suggesting future dedicated Transformer models could be trained on NC events to improve results further.

While CNN/CVN remains essential for image-based classification tasks (like event PID), the shift from LSTMs to Transformers is largely driven by the latter's superior handling of feature-based sequence data, addressing the inherent technical limitations and specific performance failures observed in the recurrent architecture.
