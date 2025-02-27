# Contrastive Limited Adaptive Histogram Equalisation (CLAHE)
Even for human experts, it can be difficult to identify subtle pathological characteristics in poor-contrast regions, especially signals from weak regurgitant jets. To tackle this problem, **CLAHE** [1,2] is applied as a comparative pre-processing step, commonly used for medical image enhancement. Unlike **global histogram equalisation** [3], which calculates the histogram of pixel intensities and performs equalisation globally, **CLAHE applies equalisation on a local scale**.

The input image is divided into **8 × 8 × 8 (H × W × D) kernels**. In each kernel, the intensity histogram is equalised. For high-contrast regions, a **clip limit** is applied to prevent over-amplification.

---

# MIRA Training Configuration

## Augmentations
For the **random transformations**, the probability is set to **0.3**:
- Z-score normalisation [4]
- Random bias field
- Random adjustment contrast
- Random flipping (x-/y-axis)
- Random 90-degree rotation
- Random Gaussian Noise

---

## First Stage of Pre-Training
- **Epoch**: 500
- **Batch size**: 32
- **Gradient accumulation**: 8
- **Optimizer**: LARS [5]
- **Learning rate scheduler**: Cosine Annealing [6]
- **Hyperparameters**:
  - Learning rate: `0.3`
  - Momentum: `0.9`
  - Weight decay: `1e-5`
  - Trust Coefficient: `1e-3`
- **Loss function**: Normalised temperature-scaled cross-entropy (**NT-Xent**) [7]
- **Temperature**: `0.1`
- **Projection head**: Default **SimCLR 2-layer MLP** [8]

---

## Second Stage of Pre-Training
- **Epoch**: 2500
- **Batch size**: 2
- **Gradient accumulation**: 64
- **Optimizer**: LARS [5]
- **Learning rate scheduler**: Cosine Annealing [6]
- **Hyperparameters**:
  - Learning rate: `1e-5`
  - Momentum: `0.9`
  - Weight decay: `1e-5`
  - Trust Coefficient: `5e-4`
- **Loss function**: Normalised temperature-scaled cross-entropy (**NT-Xent**) [7]
- **Temperature**: `0.05`
- **Projection head**: Default **SimCLR 2-layer MLP** [8]

---

## Classification
- **Epoch**: 500
- **Batch size**: 16
- **Gradient accumulation**: 1
- **Optimizer**: Adam [9]
- **Hyperparameters**:
  - Learning rate: `1e-4`
  - Weight decay: `1e-5`
  - AMSGrad: `True`
- **Loss function**: **Focal loss** [10]
- **Gamma**: `2`

---

# Valve Landmarks Extraction Error
For the **valve landmarks extraction**, the model is evaluated on **100 labelled UKBB cases**. The **mean pixel error** for the predicted landmarks is **14.6 ± 9.90mm**. The individual landmark errors are shown in the table below. The landmarks are used for the cropping of the region of interest, thus the errors lie in an acceptable range of **50 pixels from the image center**. There are **only 2 cases** that exceeded the range and used the **image center as an alternative**.

### **Landmark Extraction Error Table**
| Landmark  | Error (mm) | Landmark  | Error (mm) | Landmark  | Error (mm) |
|-----------|------------|-----------|------------|-----------|------------|
| 1 (Mitral Anterior) | **15.7 ± 10.3** | 2 (Mitral Posterior) | **15.0 ± 8.83** | 3 (Mitral Septal) | **9.88 ± 5.94** |
| 4 (Mitral Freewall) | **14.5 ± 11.4** | 5 (Mitral Septal) | **11.6 ± 6.62** | 6 (Mitral Freewall) | **21.3 ± 10.4** |

---

# Classification Results
The table below presents the **evaluation metrics** for individual **LAX views** of the **MIRA-3 model**.

### **Classification Results of MIRA-3**
| View   | Accuracy | Specificity | Sensitivity | Precision | F1-score | AUC  |
|--------|---------|-------------|-------------|------------|-----------|------|
| 2-CH   | **0.68 ± 0.01** | **0.77 ± 0.08** | **0.46 ± 0.15** | **0.68 ± 0.02** | **0.67 ± 0.01** | **0.67 ± 0.03** |
| 3-CH   | **0.70 ± 0.02** | **0.89 ± 0.05** | **0.24 ± 0.16** | **0.65 ± 0.05** | **0.65 ± 0.05** | **0.70 ± 0.03** |
| 4-CH   | **0.71 ± 0.03** | **0.74 ± 0.07** | **0.66 ± 0.10** | **0.74 ± 0.02** | **0.72 ± 0.02** | **0.77 ± 0.01** |
| **LAX** | **0.71 ± 0.04** | **0.71 ± 0.07** | **0.71 ± 0.05** | **0.75 ± 0.02** | **0.72 ± 0.04** | **0.78 ± 0.01** |

---

# References
1. Reza, A. M. (2004). Realization of contrast limited adaptive histogram equalization (CLAHE) for real-time image enhancement.
2. Stimper, T. et al. (2019). Multidimensional histogram equalization.
3. Abdullah-Al-Wadud, M. et al. (2007). A dynamic histogram equalization for image contrast enhancement.
4. Patro, S. et al. (2015). Normalization: A preprocessing stage.
5. You, Y. et al. (2017). Large batch training of deep networks with LARS optimizer.
6. Loshchilov, I. et al. (2017). SGDR: Stochastic gradient descent with warm restarts.
7. Sohn, K. (2016). Improved deep metric learning with multi-class N-pair loss.
8. Chen, T. et al. (2020). A simple framework for contrastive learning of visual representations (SimCLR).
9. Kingma, D. P. et al. (2015). Adam: A method for stochastic optimization.
10. Lin, T. Y. et al. (2020). Focal loss for dense object detection.

---
