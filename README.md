Can an Art-Style Classifier Trained on WikiArt Recognize Museum Images?
Author: Abdullahi Jimale
Course: CIS 631, Minnesota State University, Mankato, Fall 2026
Instructor: Dr. Suboh Alkhushayni
Overview
Art-style classifiers are usually trained on WikiArt. Museums and art applications may want to use these models on their own collections, but images from different sources can vary in digitization, color, and framing. These differences may affect how well a model recognizes art styles. This project measures that performance gap and tests whether fine-tuning with a small number of museum images improves results at another museum.

The study focuses on collection digitizations from the Metropolitan Museum of Art (the Met) and the Rijksmuseum. It does not test photographs taken inside galleries.
Research Questions
1. How much does performance change when a WikiArt-trained model is tested on images from the Met and the Rijksmuseum?
2. Does fine-tuning on a small number of images from one museum improve performance at the other museum?
Data Sources
Dataset	Role	Initial collection
WikiArt	Training, validation, and held-out testing	103,250 images
The Met	Museum evaluation	2,214 paintings
Rijksmuseum	Museum evaluation	4,695 paintings


WikiArt data came from the Artificio/WikiArt_Full dataset. Met paintings came from the European Paintings and Lehman Collection. Rijksmuseum paintings came from the SK collection, accessed through its Search API and IIIF.

The project did not use a common 27-style WikiArt release because 58% of its images have an unknown artist, making an artist-disjoint split difficult.
Data Preparation
The WikiArt collection was filtered to remove 4,294 images that were not paintings, 2,661 black-and-white images, and 425 near-duplicates.

Museum style labels were derived from Wikidata movement labels, painter movements, and rules for regional schools and historical periods. Copies identified as “after” an artist were not assigned that artist’s movement. Works without a reliable movement label after 1780 were left unlabeled.

A random review of 100 museum labels, with 50 from each museum, found 94 labels correct. For artists appearing in both WikiArt and a museum collection, 20 of 23 reviewed labels agreed with WikiArt.

The project also removed likely overlaps between WikiArt and museum collections: 446 Met paintings and 282 Rijksmuseum paintings. A visual review of 60 removed pairs found that 46 showed the same painting or another version of it.

Six styles had enough examples across all three datasets to meet the project’s minimum counts:
Style	WikiArt	Met	Rijksmuseum
Northern Renaissance	1,769	192	106
Mannerism	1,238	62	73
Baroque	4,165	398	2,128
Rococo	2,501	166	472
Romanticism	7,820	53	38
Realism	8,651	113	291


The WikiArt split contains 18,501 training images, 3,786 validation images, and 3,857 test images, with no artists shared between splits. The museum test sets contain 984 Met paintings and 3,107 Rijksmuseum paintings.
Models and Experiments
The main baseline is a ResNet-50 initialized with ImageNet weights and fine-tuned on WikiArt. It was trained with three random seeds. Chance and majority-class baselines provide comparison floors.

The project also plans CLIP and DINOv2 linear probes, CORAL domain adaptation, and few-shot experiments using 5, 10, 25, or 50 museum images per style. The few-shot tests will run in both directions: Met to Rijksmuseum and Rijksmuseum to Met.

The ablations test whether image characteristics explain the performance gap. So far, they have changed color normalization, resolution, cropping, JPEG quality, and grayscale. These preliminary ablations use one trained model, seed 2. Results for seeds 1 and 3 are still to come.
Preliminary Results
Test set	Paintings	Balanced accuracy	Macro-F1
WikiArt test	3,857	61.1 ± 1.3%	59.7 ± 1.3%
The Met	984	59.8 ± 1.0%	57.5 ± 1.7%
Rijksmuseum	3,107	57.5 ± 0.2%	45.9 ± 1.5%


The results average three seeds and include standard deviations. Because Baroque makes up a large share of the Rijksmuseum set, plain accuracy can be misleading. Always predicting Baroque gives 68.5% plain accuracy there, but only 16.7% balanced accuracy.

For artists absent from WikiArt training, balanced accuracy was 61.1% on WikiArt, 56.3% at the Met, and 57.7% at the Rijksmuseum. The Met difference was statistically significant (p = 0.03). The Rijksmuseum difference was not (p = 0.08). At the Met, paintings by artists seen during training scored about 13 points higher, suggesting that the model partly recognizes painters.
Early Ablation Results
The ablation results show changes in balanced accuracy, in percentage points, compared with normal images:
Image change	WikiArt	Met	Rijksmuseum
Center crop	—	+0.6	+0.5
512-pixel images	—	0.0	+0.1
Color matched to WikiArt	−4.7	−3.5	−4.6
JPEG quality 50	−8.1	−4.5	−1.1
Grayscale	−8.3	−7.0	−4.2


Center cropping and higher resolution had little effect. Color matching lowered results across all three sets, and grayscale reduced performance on WikiArt as well as on the museum sets. These early findings suggest that color carries useful style information, while the tested image-quality changes do not explain much of the museum performance gap. The ablation results need confirmation across the remaining seeds.
Evaluation
The project reports balanced accuracy, macro-F1, per-style results, confusion matrices, and expected calibration error. Baseline results use three random seeds with bootstrap confidence intervals.
Limitations
- Museum images are professional digitizations, not gallery photographs.
- Style labels come from Wikidata and date rules. The review found six incorrect labels in a sample of 100.
- Romanticism and Mannerism have relatively few museum examples.
- The main evaluation includes two Western museums.
- The current ablation results come from one trained model. CLIP may also have encountered some paintings during pretraining.
Project Status
Completed
- Collected and cleaned the three datasets
- Removed WikiArt–museum overlaps and visually checked a sample
- Derived museum labels and reviewed 100 examples
- Created an artist-disjoint WikiArt split
- Ran the three-seed ResNet-50 baseline
- Ran preliminary image ablations on one seed
Next Steps
- Run the ablations on seeds 1 and 3, including grayscale training
- Complete the few-shot transfer curve in both directions
- Evaluate CORAL, CLIP, and DINOv2
- Analyze model errors and tighten artist-name matching
- Complete the related-work review
