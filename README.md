- Github: https://github.com/wcq19941215/SceneTextRemoval

# Metric
## PSNR (Peak Signal-to-Noise Ratio)
- `psnr = 10 * math.log10(r ** 2 / mse)`
```python
def mean_psnr(a, b, max_val=255.0):
    psnr_value = tf.image.psnr(a, b, max_val)
    return psnr_value, tf.reduce_mean(psnr_value)
```
## SSIM (Structural Similarity Index Map)
```python
def mean_ssim(a, b, max_val=255.0):
    ssim_value = tf.image.ssim(a, b, max_val)
    return ssim_value, tf.reduce_mean(ssim_value)
```

# Paper Review
- Paper: [Scene text removal via cascaded text stroke detection and erasing](https://arxiv.org/pdf/2011.09768.pdf)
- It is obvious that if we can extract the exact text stroke, which means that we can preserve original contents of input image as much as possible, and then we could achieve better result.
- However, such precise areas are difficult to obtain, to best of our knowledge, there is no related research to focus on distinguishing text strokes from non-stroke area in the pixel-wise level.
- In this paper, we propose a novel “end-to-end” framework based on generative adversarial network (GAN) to address this problem. The key idea of our approach is first to extract text strokes as accurately as possible, and then improve the text removal process.
- We design a text stroke detection network (TSDNet), which can effectively distinguish text strokes from non-text area.
- Specifically, we decouple the text removal problem into text stroke detection and stroke removal. We design a text stroke detection network and a text removal generation network to solve these two sub-problems separately.
  - The learning-based methods can be roughly classified into two main categories, i.e., text removal without/with using mask. The former simply takes the given image as input and removes all the texts from the whole input image. This kind of methods often left noticeable remnants of text or distort non-text area incorrectly, and cannot remove text locally. The latter usually uses a region mask, i.e., a rectangle or polygon mask roughly indicating the text region [e.g., Fig. 1(b)], as additional input to facilitate the text removal.
  - In other words, the mask used by MTRNet covers some unnecessary/redundant regions (i.e., non-stroke areas), especially when text strokes are scattered sparsely.
  - It is obvious that if we can extract the exact text stroke, which means that we can preserve original contents of input image as much as possible, and then we could achieve better result. However, such precise areas are difficult to obtain, to best of our knowledge, there is no related research to focus on distinguishing text strokes from non-stroke area in the pixel-wise level.
  - In this paper, we propose a novel “end-to-end” framework based on generative adversarial network (GAN) to address this
problem. The key idea of our approach is first to extract text strokes as accurately as possible, and then improve the text
removal process. These two processes can be further enhanced via a simple cascade.
  - In addition, current public datasets for
scene text removal are all synthetic, which to some extent affect the generalization ability of trained models. To facilitate this research and be close to real-world setting, we construct a new dataset with high quality.
  - The main contributions of our work include:
      - We design a text stroke detection network (TSDNet), which can effectively distinguish text strokes from non-text area.
      - We propose a text removal generation network and combine it with TSDNet to construct a processing unit, which is cascaded to obtain our final network. Our method demonstrates the superior performance.
      - We propose a weighted-patch-based discriminator to pay more attention to the text area of given images, making it easier for the generator to generate more realistic images.
      - We construct a high-quality real-world dataset for the scene text removal task, and this dataset can be used to benchmark related text removal methods. It can also be used in other related tasks.
      - CRAFT [17] effectively detect arbitrary text area by exploring each character and affinity between characters. In this work, we adopt this method as the tool to measure the performance of scene text removal (more details in Section IV-B).
      - Existing approaches of the scene text removal can be classified into two major categories: traditional non-learning methods and deep-learning-based methods. Traditional approaches typically use color-histogram-based or threshold-based methods to extract text areas, and then
propagate information from non-text regions to text regions depending on pixel/patch similarity [1]–[3].

## Metric
- We evaluate the performance from two different aspects: (1) can the method remove text from an image completely; (2) can text area be replaced with appropriate content.
- An accurate text detector is often used for the former evaluation metric. As for a text-erased image, the cleaner text is erased, the fewer text will be detected. In this work, we use the state-of-the-art text detector CRAFT and use DetEval protocol for evaluation (i.e., recall, precision, and f-measure).
- For the second evaluation metric, we adopt general image inpainting metrics, and mainly use the following three indicators: 1) mean absolute error (MAE); 2) peak signal-to-noise ratio (PSNR); and 3) the structural similarity index (SSIM).
- FOR PSNR AND SSIM (in %), HIGHER IS BETTER; FOR MAE, R (RECALL), P (PRECISION), AND F (F-MEASURE), LOWER IS BETTER.

# Comparison with Other Methods
- We quantitatively and qualitatively compare our method with state-of-the-art text removal methods: ST Eraser, EnsNet, and MTRNet, as well as recent image inpainting method: GatedConv.
