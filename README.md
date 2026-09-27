# 🌊 Underwater Image Enhancement — Zero-DCE + FUnIE-GAN

This project compares two ways to clean up underwater photos, combining **Zero-DCE** (a lighting-fixing model) and **FUnIE-GAN** (a color/contrast-fixing model). Both were trained on the **Sea-thru** and **UIEB** datasets.

## The Problem

Photos taken underwater look bad because of two main issues:

- **Color Loss** — Water absorbs warm colors (red, yellow, orange) first, so photos end up looking blue or green.
- **Haze and Low Contrast** — Tiny particles floating in the water scatter light, making photos look foggy and hiding fine details.

## The Goal

Turn a bad, distorted underwater photo into a clear, natural-looking one — like it was taken on dry land. In short:

- Fix the colors so they look natural again.
- Remove the underwater haze/fog.
- Boost the contrast so details are visible.

## Method

Two versions of FUnIE-GAN were trained and compared:

| | **Version A (Baseline)** | **Version B (Proposed)** |
|---|---|---|
| What goes into the model | Raw Sea-thru photos | Sea-thru photos brightened by Zero-DCE first |
| What the model compares against | UIEB (clear reference photos) | Brightened Sea-thru + UIEB combined |
| Loss function | Adversarial + Perceptual (MobileNetV2) | Adversarial + Perceptual (MobileNetV2) |

**The idea being tested:** brightening the dark Sea-thru photos with Zero-DCE first, then training on a bigger mixed dataset, should give better and more natural results than just using the raw photos.

## Dataset & Steps

1. **Data used**: 1,063 Sea-thru photos (raw underwater images) + 890 UIEB photos (clean reference images).
2. **Resize** every photo to 256×256 pixels.
3. **Data check (EDA)**: found that 95.2% of the Sea-thru photos are very dark, and colors are strongly shifted toward green (almost no red left) — this confirms the color-loss problem described above. Also found 114 near-duplicate photos.
4. **Data cleaning**: removed those 114 duplicates plus 2 blurry photos, leaving 947 usable photos.
5. **Data augmentation**: flipped, rotated, and slightly adjusted brightness/contrast of the training photos to give the model more variety.
6. **Models used**:
   - **Zero-DCE** — a small, lightweight model that brightens dark photos without needing "before/after" example pairs.
   - **FUnIE-GAN** — the main model that fixes color and contrast; made of a Generator (the "fixer") and a Discriminator (the "judge").
   - **Perceptual loss** using MobileNetV2, a lightweight way to check the fixed photo still looks like the original scene.
7. **Training**: each version trained for 5 rounds (epochs) through the data.
8. **How results were measured**: mainly using UCIQE (higher = better color/contrast) and NIQE (lower = looks more natural). PSNR/SSIM were also measured, but here they just show how much the photo changed, not how good it looks.

## Final Results

Average scores across all test photos:

| Model | UCIQE ↑ (higher=better) | NIQE ↓ (lower=better) | PSNR | SSIM |
|---|---|---|---|---|
| Original photo (no fixing) | 0.3868 | 15.1677 | — | — |
| Zero-DCE only | 0.4048 | 18.5853 | 12.694 | 0.3249 |
| **Version A (baseline)** | **0.4440** | **15.3463** | 7.135 | 0.1254 |
| Version B (proposed) | 0.4284 | 26.5207 | 6.148 | 0.1400 |

**Best color/contrast (UCIQE):**
1. 🥇 Version A — 0.4440
2. 🥈 Version B — 0.4284
3. 🥉 Zero-DCE only — 0.4048
4. Original — 0.3868

**Most natural-looking (NIQE, lower is better):**
1. 🥇 Original photo — 15.1677
2. 🥈 Version A — 15.3463
3. 🥉 Zero-DCE only — 18.5853
4. Version B — 26.5207

## Conclusion

The results show that **Version A (the simple baseline)** actually did better than **Version B (the proposed, "smarter" version)** on both main scores. The idea that pre-brightening with Zero-DCE and using a bigger mixed dataset would help did **not** hold up here — Version B ended up looking the *least* natural of all four options (even worse than doing nothing), even though its color/contrast score was still decent.

Some important context for these results:

- Training only ran for **5 epochs** per version — that's quite short, so the models likely hadn't fully learned yet. These results are early/preliminary, not the final word on which method is better.
- PSNR/SSIM only show how much a photo changed, not whether it looks better, since there were no exact "before/after" matched pairs to compare against.
- The dataset really was very dark and color-shifted, which is why trying a brightening step (Zero-DCE) made sense to test — it just didn't pay off in this particular run.

**What to try next**: train for more epochs, adjust the balance of the perceptual loss for Version B, and test on more photos to see if these results stay the same.

## Output Files

```
/kaggle/working/
├── dataset_512/                 # Resized photos (256×256)
├── eda_output/                  # Data-check charts (size, brightness, duplicates, colors, noise)
├── seathru_dce_corrected/       # Photos after Zero-DCE brightening
├── checkpoints/                 # Saved model weights (Version A & B)
└── model_output/
    ├── comparison_results.png   # Side-by-side comparison images
    └── summary_chart.png        # Chart summarizing all scores
```