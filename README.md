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
## Dataset
- To train the deep model for text removal task, paired text image and text-free image are required. However, it is difficult to obtain such paired data for real-world scene images, this is why synthetic datasets are used for constructing text removal dataset in previous approaches.
### Synthetic Datasets
- Currently, there are only two synthetic datasets, i.e., the Oxford synthetic scene text detection dataset and the SCUT synthetic text removal dataset. These two datasets adopted the same synthetic technology proposed by, and shared the same drawback, i.e., given a text-free image as background, a few or more images with synthesized text are obtained. For instance, the Oxford dataset synthesized 800,000 images using only 8,000 text-free images, which means that each 100 text images are synthesized using the same background image, leading to insufficiency of the diversity of background. Such kind of repetition would cause negative affect to the generalization ability of models.
- In addition, synthetic data is only an approximation of real-world data. Current text synthesis technologies cannot generate realistic enough text images, which would restrain the text removal ability of models trained on synthetic data. When existing text removal methods are trained on synthetic dataset, and then tested on real-world data, we found that there often exists obvious text remnants and unsatisfactory artifacts, which is quantitatively analyzed in Section IV-E.
### Read-world Dataset
- To this end, we propose to construct a real-world dataset for the text removal scenario. To construct such dataset, we first collect 5,070 images with text from ICDAR2017 MLT dataset, and 1,970 images captured from supermarkets and streets, etc. Then, post-processing are applied to obtain corresponding text-free images, region masks, and text stroke masks. We manually remove the text from these collected images and obtain text-free images as groundtruth using the inpainting tools in Photoshop©. The region masks are annotated by using VGG Image Annotator tool.
- ***For the ground-truth stroke mask, we first compute the difference between paired text images and text-free images, and then turn it into a binary image.*** To enrich the diversity of our dataset, we also use synthesis method and then manually select 4,000 images with high realism. In total, we obtain 11,040 images as training set (Train rw). Several samples of our dataset are shown in Fig. 4.
- To construct testing set (Test rw), we additionally collect 1,080 real-world images and apply the same above post-processing to obtain the text-free images, region masks and text stroke masks. In this work, we mainly conduct the experiments and comparisons on our real-world dataset.
- Oxford dataset: In the meanwhile, we also conduct experiments on a public synthetic dataset, i.e., the Oxford dataset, which is much larger than the SCUT dataset. For the Oxford dataset, we randomly select 75% of the whole dataset as training set (Train ox), and then randomly select 2,000 images from remaining set as testing set (Test ox).
## Metric
- We evaluate the performance from two different aspects: (1) can the method remove text from an image completely; (2) can text area be replaced with appropriate content.
- An accurate text detector is often used for the former evaluation metric. As for a text-erased image, the cleaner text is erased, the fewer text will be detected. In this work, we use the state-of-the-art text detector CRAFT and use DetEval protocol for evaluation (i.e., recall, precision, and f-measure).
- For the second evaluation metric, we adopt general image inpainting metrics, and mainly use the following three indicators: 1) mean absolute error (MAE); 2) peak signal-to-noise ratio (PSNR); and 3) the structural similarity index (SSIM).
- FOR PSNR AND SSIM (in %), HIGHER IS BETTER; FOR MAE, R (RECALL), P (PRECISION), AND F (F-MEASURE), LOWER IS BETTER.

# Comparison with Other Methods
- We quantitatively and qualitatively compare our method with state-of-the-art text removal methods: ST Eraser, EnsNet, and MTRNet, as well as recent image inpainting method: GatedConv.
