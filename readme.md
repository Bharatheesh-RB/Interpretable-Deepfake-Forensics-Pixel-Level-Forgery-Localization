# Interpretable Deepfake Forensics: Pixel-Level Forgery Localization

## Project Intention
* As generative AI becomes increasingly advanced, deepfakes are reaching a point where they are completely indistinguishable to the human eye. 
* Most current detection models operate as "black boxes"—they might flag an image as fake, but they offer no visual evidence or explanation as to *why*. 
* In forensic, legal, and real-world applications, this lack of transparency is a major flaw. 

* The intention of this project is to solve the "black box dilemma." 
* I aim to shift the focus from simple binary detection (just guessing "Real" or "Fake") to **interpretable localization**, ensuring the AI provides clear visual evidence of its decision.

## The Dataset (HiDF)
* To rigorously test this model, I bypassed older, easily detectable datasets and benchmarked my architecture against the **HiDF (Hierarchical Deepfake Dataset)**. 
* Released in 2025, it is a state-of-the-art dataset designed specifically to produce "human-indistinguishable" deepfakes.

**Key Characteristics:**
* **Scale:** Contains 60,500 high-quality images, split perfectly evenly between 30,250 Real images and 30,250 Fake images.
* **Diversity:** Features over 6,127 unique subjects, ensuring the model learns to generalize across a massive variety of facial structures rather than memorizing a few faces.
* **Metadata:** Includes detailed annotations for race, gender, and age, allowing me to evaluate the model for demographic fairness and bias.

Unlike earlier deepfakes that contain obvious visual glitches, the images in HiDF are highly realistic and often trick standard AI models. By forcing my Vision Transformer ( Swin Transformer ) to analyze and segment these high-fidelity fakes, I ensure my localization heatmaps are robust and capable of handling the most advanced face-swapping techniques available today.

## What I Did
To achieve this, I developed a deep learning segmentation pipeline designed to hunt down pixel-level anomalies in high-fidelity deepfake datasets. 

### The Approach:  
* Instead of using standard Convolutional Neural Networks (CNNs), which often miss global facial inconsistencies, I utilized a hierarchical Vision Transformer architecture known as the **Swin Transformer**. 
* This allows the model to analyze the relationship between every patch of the image simultaneously, detecting subtle semantic mismatches (ex: lighting or geometric errors across different parts of the face).

### The Output: 
* Rather than outputting a single label, the model processes a suspected image and generates a pixel-precise binary mask. 

### The Result: 
* The final output is an interpretable heatmap overlaid on the original image, explicitly highlighting the exact manipulated regions (such as a swapped mouth or altered eyes). 
* This provides the clear, geographical evidence needed for trustworthy deepfake forensics.