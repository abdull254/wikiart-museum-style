# Can an Art-Style Classifier Trained on WikiArt Recognize Museum Photos?

**Author:** Abdullahi Jimale
**Course:** CIS 631, Minnesota State University, Mankato (Fall 2026)
**Instructor:** Dr. Suboh Alkhushayni

## Overview

Art-style classifiers are usually trained and tested on WikiArt, where they score well. But museums, art apps, and search tools want to use these models on their own collections, and museum images are different: different cameras, lighting, color, and cropping. A model can score high in testing and still fail when the data source changes. This is called a **domain gap**.

This project measures how big that gap is for art-style classification, and whether a small amount of museum data can close it.

## Research Questions

1. **How much does performance drop** when a model trained on WikiArt is tested on images from the Metropolitan Museum of Art (the Met) and the Rijksmuseum?
2. **Does a museum fix transfer?** If the model is fine-tuned with a few images from one museum, does the improvement carry over to the other museum?

## Data Sources

| | WikiArt | The Met | Rijksmuseum |
|---|---|---|---|
| Role | Training | Test | Test |
| What it stores | Image + style label | Title, artist, date, object type | Title, artist, date, object type |
| Style label? | Yes | No, must be derived | No, must be derived |

The Met images come from the Met Open Access collection. The Rijksmuseum images come from its public collection data.

## Data Preparation

### WikiArt

- Started with about **42,500 images in 13 style folders**.
- Checked class balance, image sizes, and color modes. Classes are unbalanced: Romanticism has about 6,800 images and Western Medieval about 1,150.
- Removed **115 exact duplicate** images, and found **one label conflict** (the same image under Art Nouveau and Expressionism; both styles are dropped from the final set).
- Converted all images to RGB and resized them to one input size.
- Reduced the styles from 13 to **10 museum-friendly styles**, chosen before any model training:
  Early Renaissance, High Renaissance, Northern Renaissance, Mannerism, Baroque, Rococo, Romanticism, Realism, Impressionism, Post-Impressionism.

The cleaning checks were first tested on a balanced sample of **3,000 images (300 per style)**. That sample had no broken files, no duplicates, and no conflicting labels. It also showed two hidden problems:

- **Black-and-white images:** 209 of the 3,000 are black-and-white, and 91 of those are Northern Renaissance (likely prints and engravings). A model could learn the shortcut "black-and-white means Northern Renaissance." This is tested with a grayscale ablation.
- **Unknown artists:** about 53% of sampled images are labeled "Unknown Artist." This affects the artist-disjoint split (see below).

### Museum Style Labels

The museums do not label paintings by style, so labels are derived:

1. **Paintings only.** Prints, drawings, sculpture, and other objects are removed, so failures come from style and not object type.
2. **Wikidata movements.** Each painting's art movement from Wikidata is mapped to the 10 styles.
3. **Date rule.** General labels like "Italian Renaissance" are split by date: before 1490 is Early Renaissance, 1490 to 1530 is High Renaissance. Later paintings are dropped.
4. **Hand-check.** A sample of 50 derived labels is checked by eye.

| Museum | Paintings found | With a reliable style label |
|---|---|---|
| The Met | 13,051 | 2,009 |
| Rijksmuseum | 5,827 | 1,610 |

Paintings without a reliable label are left out rather than guessed.

Only **four styles have at least 50 paintings in both museums**: Realism, Baroque, Romanticism, and Rococo. These four form the main cross-museum test. The Met is also tested on its eight usable styles as an extra result.

At the Rijksmuseum, Baroque is about 55% of labeled paintings, so a model that always guesses Baroque would score about 55%. A majority-class baseline is always reported next to results.

### Leakage Checks

- **Artist-disjoint split:** each artist appears in only one of training, validation, or test. The final split uses known-artist images only.
- **Overlap removal:** paintings that appear in both WikiArt and a museum collection are removed, so the model is never tested on a painting it saw in training.

## Models

| Level | Model | Purpose |
|---|---|---|
| 1 | Chance and majority class | Floors any real model must beat |
| 2 | Color-feature baseline | Simple non-deep-learning reference |
| 3 | ResNet-50 (transfer learning from ImageNet) | Main model, fine-tuned on WikiArt |
| 4 | CLIP and DINOv2 | Newer models trained on larger, more varied data |

## Preliminary Result: Color Baseline

On the 10-style, 3,000-image sample:

| Setting | Accuracy |
|---|---|
| Chance | 10.0% |
| Artist-disjoint split | 20.9% |
| Random split | 25.8% |

Color alone beats chance, so color carries some style information. But accuracy drops when artists are kept apart, so part of the random-split score came from recognizing artists, not styles. For this reason, all experiments use an artist-disjoint split.

## Experiments

**Experiment 1 (RQ1):** Train and test on WikiArt to get the best-case score. Then test the same model on the Met and the Rijksmuseum with no changes. The drop is the domain gap.

**Experiment 2 (RQ2):** Fine-tune with 5 to 50 museum images per style and test on the other museum, in both directions (Met to Rijksmuseum, Rijksmuseum to Met). This produces a few-shot curve.

**Why the gap happens:** Ablations change one thing at a time, such as training with grayscale images. AdaBN adjusts the model to a new image source without new labels.

## Evaluation

- Top-1 and top-3 accuracy
- Macro F1 (treats every style equally)
- Per-style results and confusion matrices
- Three random seeds per experiment, with confidence intervals
- Calibration (does the model's confidence match its accuracy?)

## Limitations

- Style labels are imperfect; experts sometimes disagree on a painting's style.
- Most museum paintings have no reliable style label, so museum test sets are small.
- The main cross-museum test covers only four shared styles.

## Status

**Done**
- WikiArt cleaning and duplicate checks
- Style reduction from 13 to 10
- Museum style mapping and per-style counts
- Color baseline

**Next**
- Finish the 50-label hand-check
- Run cleaning checks on the full dataset
- Build the final artist-disjoint split and remove overlapping paintings
- Train ResNet-50, then run the museum tests and the few-shot curve

## Setup

The project uses Python in a virtual environment.

```bash
git clone https://github.com/abdull254/wikiart-museum-style.git
cd wikiart-museum-style
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

The WikiArt dataset is not included in this repository because of its size. Download it separately and place it in the data folder used by the scripts.

## Contact

Abdullahi Jimale
[LinkedIn](https://www.linkedin.com/in/jimale1) · [GitHub](https://github.com/abdull254)
