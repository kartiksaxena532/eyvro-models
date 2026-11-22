# eyvro-models

Model files for [EYVRO](https://www.eyvro.in)'s in-browser tools, served by jsDelivr so they never load from the website's own hosting.

## handwriting/rec-v1

A small handwriting recogniser (about 0.8 million parameters, 1.6 MB). It reads one detected line or word at a time and outputs text. It runs in the visitor's browser with [onnxruntime-web](https://www.npmjs.com/package/onnxruntime-web). The site uses it to read prescription slips (SPH, CYL, AXIS, ADD, PD) and to turn handwriting into text. Photos are read on the device and never uploaded.

```
https://cdn.jsdelivr.net/gh/kartiksaxena532/eyvro-models@v1/handwriting/rec-v1.onnx
https://cdn.jsdelivr.net/gh/kartiksaxena532/eyvro-models@v1/handwriting/rec-v1.json
```

| | |
|---|---|
| Input | `image`: float32 `[batch, 1, 32, width]`, grayscale, 32 px high, contrast-stretched (2nd to 98th percentile), scaled to [-1, 1], ink dark |
| Output | `probs`: float32 `[batch, width/4, 85]`, per-step probabilities; greedy CTC decode, index 0 is the blank |
| Characters | listed in `rec-v1.json` (`chars`, index i+1 is `chars[i]`) |
| Architecture | fully convolutional (CNN + dilated 1-D convolutions), trained with CTC; weights stored as float16, computed in float32 |

Finding the text on the page is a separate model: PaddleOCR's PP-OCRv4 mobile detector (Apache-2.0), loaded from the `@gutenye/ocr-models` npm package.

### Training and scores

It was trained on synthetic images only: text rendered in 83 handwriting fonts and 34 print fonts, plus numbers built from real handwritten MNIST digits. The images were then degraded like phone photos (blur, JPEG, uneven light, table lines, neighbouring text). Exact-match rates on held-out sets, from `rec-v1.json`:

| Set | Exact match | Character error |
|---|---|---|
| Prescription values and labels | 89.7% | 1.9% |
| Numbers in real handwriting (MNIST test writers, never seen in training) | 94.2% | 1.4% |
| Names, dates, phone numbers | 81.0% | 3.6% |
| English words and short phrases | 64.2% | 5.8% |

These are synthetic test sets. Real doctors' handwriting will score lower, which is why the site asks people to check every value against their slip.

### Data credits

- Handwriting and print fonts from [Google Fonts](https://fonts.google.com) (SIL Open Font License / Apache 2.0), used only to render training images.
- [MNIST](https://yann.lecun.org/exdb/mnist/) handwritten digits (derived from NIST Special Databases 1 and 3), used to build numbers in real handwriting.
- English text from EYVRO's own journal copy.
