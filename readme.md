# Interpretable Deepfake Forensics: Pixel-Level Forgery Localization

## Project Overview
This project tackles the complex challenge of deepfake localization. While traditional deepfake detection simply outputs a binary "Real" or "Fake" label, this pipeline is designed to identify and highlight the exact pixel boundaries of a facial forgery. By generating pixel-precise heatmaps, the model provides interpretable forensic evidence of manipulation.

## Dataset Details
The model is trained and evaluated on the high-difficulty **HiDF (High-fidelity DeepFake) Dataset**, which contains over 42,000 manipulated images and videos. 

* **Structure:** The dataset consists of "Fake" images (face-swaps) and their corresponding "Real" source images. 
* **Ground Truth Generation:** Instead of relying on pre-drawn masks, my custom dataloader dynamically generates ground-truth forgery masks during training by calculating the absolute pixel difference (`cv2.absdiff`) between the Real and Fake pairs.
* **Demographic Metadata:** The dataset includes detailed metadata classifying subjects by **Gender**, **Age**, and **Race**, allowing for rigorous fairness and bias testing across different demographics.

## Model Architecture: Swin-UNet
To achieve high-resolution localization, this project implements a hybrid **Swin-UNet** architecture that combines the global context awareness of Vision Transformers with the spatial precision of Convolutional Neural Networks.

* **Encoder (Swin Transformer):** I utilize a pre-trained `swin_tiny_patch4_window7_224` model. Unlike standard CNNs that look at local pixel clusters, the Swin Transformer processes the image as a sequence of patches, allowing it to understand global structural relationships and blending artifacts across the entire face.
* **Decoder (CNN Up-sampler):** A cascading series of `ConvTranspose2d` layers rebuilds the spatial resolution from the encoded feature maps back up to the original 224x224 image size.
* **Additive Skip Connections:** To prevent the loss of fine spatial details during the encoding process, feature maps from the Swin Transformer are permuted to `[B, C, H, W]` format and added directly to the corresponding layers in the decoder.

## Training Details
* **Input Size:** 224x224 RGB Images
* **Loss Function:** Binary Cross-Entropy (BCE) Loss
* **Data Augmentation:** Horizontal flipping and ImageNet normalization via Albumentations.
* **Fault-Tolerant Dataloader:** Designed to handle noisy open-source datasets, the custom dataloader utilizes a `try...except` recursive loop to automatically skip and replace corrupted, missing, or mismatched image pairs without crashing the training loop.
* **Epochs:** 20 complete passes over the dataset.

## Testing Results & Metrics
After 20 epochs, the model was evaluated on a held-out subset of the data. The evaluation focused on standard segmentation metrics to measure mask overlap:

| Metric | Score | Interpretation |
| :--- | :--- | :--- |
| **Pixel-wise Accuracy** | 94.41% | Overall pixel correctness (skewed high due to large unmanipulated background areas). |
| **F1-Score (Dice)** | 0.4994 | Balances precision and recall, proving the model successfully isolates the facial region. |
| **Intersection over Union (IoU)** | **0.3425** | The primary metric. Indicates the exact overlap between the predicted heatmap and the true forgery boundaries. A strong baseline for highly blended, invisible deepfakes. |

## Inferences: AI Fairness & Bias Report
To ensure the model is ethical and unbiased, I cross-referenced the IoU predictions with the dataset's demographic metadata. The results yield several crucial inferences about the model's behavior:

* **Gender Neutrality:** The model performs almost identically on **Female (0.2932 IoU)** and **Male (0.2918 IoU)** faces. This indicates it relies on structural blending artifacts rather than overfitting to gender-specific features like makeup or facial hair.
* **Racial Consistency:** Despite a significant class imbalance in the training data (e.g., White faces heavily outnumbering Indian/Black faces), the IoU remained remarkably stable across all tested races (ranging tightly from 0.2840 to 0.3050).
* **The "Age" Bias:** A notable performance drop was identified when predicting deepfakes of **Children (0.2445 IoU)** compared to **Adults (0.2980 IoU)**. This highlights a structural bias in the underlying dataset and generation tools, as deepfake algorithms are predominantly trained on adult facial proportions.