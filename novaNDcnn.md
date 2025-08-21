# NOvA electron id using CNN

NovA’s current approach to identifying electrons in its near detector relies on modern deep‐learning techniques that “see” the raw detector data as images.
In essence, the experiment converts the two‐dimensional (and combined 3D) hit patterns recorded in its segmented liquid–scintillator cells into pixel maps that capture the spatial (and sometimes timing) structure of each neutrino interaction.
These images are then fed into convolutional neural networks (CNNs) that have been trained to distinguish between different interaction types.

Below is an outline of how the machine–learning–based electron identification works and some ideas for improving its performance below 0.5 GeV.

---

### Electron Identification in NOvA’s Near Detector

**Input Data Formation:**  
Both the near and far detectors produce detailed “pixel maps” from the light collected in their PVC cells filled with liquid scintillator.
For each event, two views (often referred to as the XZ and YZ projections) are generated.
These views capture the spatial distribution of energy deposits from the charged particles produced when a neutrino interacts.

**Convolutional Neural Network (CNN) Classifiers:**  
NOvA developed the Convolutional Visual Network (CVN) to classify neutrino interactions.
In the context of electron identification, the network is trained on a large library of simulated events (as well as data) to recognize the characteristic shower–like pattern produced by electrons (or electromagnetic showers from photons) versus the track–like pattern from muons or the more diffuse deposits from neutral–current events.
 

For more granular particle identification, a dedicated “prong–CNN” is applied after the event is segmented into clusters (or “prongs”) representing individual final–state particles.
This network learns to assign particle–type probabilities to each prong, helping to decide if a given shower is consistent with an electron.

**Role in the Near Detector:**  
The near detector’s data is crucial for understanding the unoscillated neutrino beam composition and for constraining systematic uncertainties in neutrino interaction cross sections.
An accurate electron identification algorithm ensures that backgrounds (such as misidentified neutral–current events or photon–induced showers) are well controlled, which in turn improves the extrapolation of these measurements to the far detector where oscillation effects are measured.
However, this algorithm hasn't been trained on events that have a large portion of events below 0.5 GeV.

The combination of the CVN (for overall event classification) and prong–CNN (for particle–level discrimination) provides a robust, end–to–end identification chain that leverages the full granularity of the detector.

---

### Training of the Event Classifier (CVN)

• **Loss Function & Optimization:**  
  The training is performed using supervised learning with a softmax cross–entropy loss function.
An adaptive gradient method (typically Adam) is used to update the network weights.
The loss is computed over mini–batches, and standard regularization techniques (such as dropout or L2 weight decay) are applied judiciously.
 
• **Hyperparameter Tuning:**  
  Hyperparameters (including learning rate, batch size, dropout probability, and the number of convolutional filters) are tuned by monitoring the validation loss.
The training typically continues until the loss on a reserved validation set no longer improves.

• **Training, Validation, and Testing:**  
  The full simulated data set is split—often with roughly 70–80% used for training and the remainder for validation and testing.
  
  • **Validation Metrics:**  
  During training, performance is monitored using overall classification accuracy as well as receiver–operating characteristic (ROC) curves for each event category.
The CVN’s ability to discriminate among event types is quantified by metrics such as purity and efficiency in each class.
 
• **Data–Driven Cross–Checks:**  
  Since the network is trained entirely on simulation, extensive validation is performed using real detector data.
Techniques like “muon–removed” studies (where a well–identified muon is subtracted and replaced by a simulated electron) are used to compare the selection efficiency in data versus simulation, ensuring that the CVN’s performance is robust when applied to actual events

---

### Training of the Particle (Prong) Classifier (prong–CNN)

• **Input Preparation:**  
  Prongs are extracted from reconstructed events.
For each prong, a cropped image (typically covering both detector views) is generated.
These images capture the local energy deposition patterns that characterize different particle types (electrons typically produce electromagnetic showers, while muons generate long, thin tracks).

• **Loss Function & Training Method:**  
  Again, a softmax cross–entropy loss function is typically used.
Because some particle types  might be rarer than others, the training set is sometimes balanced by reweighting or data augmentation.
The optimization is carried out using Adam or a similar method, with hyperparameters tuned on an independent validation subset.

• **Metrics Evaluated:**  
  Validation focuses on per–class performance, looking at metrics such as classification accuracy, purity (the fraction of selected prongs that are correctly identified), and efficiency (the fraction of true prongs correctly classified).
Detailed studies of the prong CNN’s performance as a function of the prong’s energy, size, and location within the event are performed to ensure the network is robust over the full range of event topologies.
 
• **Cross–Checks with Data:**  
  In addition to standard validation on simulation, the prong–CNN is also compared to independent reconstruction outputs (for instance, comparing prong–CNN–based particle identification with more traditional identification methods) as well as data–driven “control” samples.
Such studies help verify that the network’s performance does not suffer from simulation mismodeling

---

### Key Input Variables  
• **For CVN:**  
  – Pixel intensities corresponding to calibrated energy deposits in each detector cell  
  – Spatial coordinates implicitly given by the arrangement of pixels (typically mapped by plane and cell index)  
  – (Optionally) Reconstructed vertex positions that help anchor the event geometry  

• **For prong–CNN:**  
  – Cropped pixel images of prongs (which capture energy deposition patterns)  
  – Implicit spatial features such as shape and orientation of the prong  
  – (Optionally) Additional reconstructed variables such as prong energy, direction, and distance from the event vertex  

---

### Optimizing for Energy Values Below 0.5 GeV

At very low energies (below approximately 0.5 GeV), several challenges arise.
The electromagnetic showers produced by low–energy electrons are less extended and deposit less light than those at higher energies.
Noise (both intrinsic detector noise and from low–energy background processes) can become comparable to the signal, making it harder for the network to distinguish the subtle features of genuine electron events.
Several strategies may be pursued to improve the performance in this regime:

**Training Sample Enrichment:**  
• **Flat Flux Training:** One can generate or reweight Monte Carlo training samples to provide a more uniform (or enhanced) sample of events in the sub–0.5 GeV range.
This counters the typical bias where most events come from the higher–energy part of the beam spectrum.
 
• **Data Augmentation:** Augmenting the training dataset by simulating variations (e.g., applying noise models, small shifts, or rotations) may help the network learn robust features even when the signal is weak.

**Specialized Network Architectures and Loss Functions:**  
• **Fine–Resolution Convolutional Layers:** Adjusting the network architecture to include more layers or convolutional filters with a smaller receptive field may allow the network to pick up on the finer details of the low–energy electromagnetic shower shapes.
 
• **Tailored Loss Functions:** Loss functions that are less sensitive to outliers—such as the logcosh loss or mean absolute percentage error—might be better suited when the signal is small and uncertainties are relatively large.
These functions can help in fine–tuning the network’s predictions in the low–energy regime.

**Enhanced Preprocessing and Feature Extraction:**  
• **Noise Reduction Techniques:** Preprocessing steps that suppress detector noise (for example, thresholding or applying filtering techniques that emphasize contiguous low–energy deposits) can improve the signal-to-noise ratio.
 
• **Additional Detector Information:** Incorporating timing information or other ancillary data (such as the longitudinal profile of the energy deposit) may help the network discriminate between genuine low–energy electron showers and background fluctuations.

**Transfer Learning and Multi–Task Learning:**  
• **Transfer Learning:** If a network is already well–trained on a broader energy range, fine–tuning it on a dedicated low–energy subset can sometimes yield improvements.
 
• **Multi–Task Networks:** A network that simultaneously learns both classification (is this an electron event?) and regression (what is the energy?) might share useful features that improve performance at low energies.

---

### Summary

NOvA’s electron identification in the near detector uses CNNs (CVN and prong–CNN) to classify neutrino interactions based on their pixel–map images.
This method has significantly improved the experiment’s ability to identify electron neutrino events compared to non machine learning methods.
However, the performance at energies below 0.5 GeV can be limited by the reduced size and distinctiveness of low–energy showers and by noise effects.
To optimize performance in this regime, one can enhance training datasets to include more low–energy examples, modify the network architecture and loss functions to be more sensitive to subtle features, apply targeted noise–reduction preprocessing, and incorporate additional information from the detector.
These improvements could lead to a more precise measurement of electron neutrino energy at low energies and, consequently, to improved magnetic moment analysis.

I am still unsure whether to pursue the path of improving the CNN's that already exist for reconstruction in the Near Detector or whether to try switching things over to the transformers that are being used in the far detector.
Long run, I think that the transformer has more potential, but at the same time, NOvA is an experiment that is already in a mature state and I assume will be superceded by DUNE when that comes online and starts collecting data.
Of the techniques proposed for improving the CNN's below 0.5 GeV, I'm not yet sure which ones will be easier to implement or provide the most improvement.



