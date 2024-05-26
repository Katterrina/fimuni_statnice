# S5. Umělá inteligence ve zpracování obrazu

> Formování obrazu (PSF, OTF, vzorkování). Klasifikace obrazu (VGGNet, GoogLeNet, ResNet, SENet). Detekce objektů (R-CNN, Fast R-CNN, Faster R-CNN, YOLO). Segmentace obrazu (FCN, UNet, Mask R-CNN). Podmíněné a nepodmíněné generativní modely (autoregresivní modely, VAEs, GANs). Modely založené na konvolučních sítích a transformerech (attention, CNN vs. ViT). (PA228)

# PA228 Machine Learning in Image Processing (P. Matula)

## IMAGES

- image definition
    - multivariable function
        - $m$ number of dimensions, $c$ number of channels
        - continuous $\mathbb{R}^m \rightarrow \mathbb{R}^c$
        - discrete $\mathbb{Z}^m \rightarrow \mathbb{R}^c$
        - digital $\mathbb{Z}^m \rightarrow \mathbb{Z}^c$
- fundamental steps in digital image processing
    - output: image
        - image aquisition
            - ilumination source - affects how the imaged scene looks
            - scene elements - static vs dynamic, a priory knowledge
                - scene adaptation may be cheaper than image processing!
            - imaging system - e.g. camera
            - (internal) image plane
            - digital image - output
        - image enhancement
        - image restoration
        - color image processing
        - wavelets & multiresolution processing
        - compression
        - morphological processing
    - output: image attributes
        - morphological processing
        - segmantation
        - representation & description
        - object recognition

### Imaging system is not perfect 

- point spread function
    - **PSF** is a response of the imaging system to an infinitesimal point source
    - space variant and depends on the illumination source
    - color shift, astigmatism, ...
- curvature of field, image distortions
- camera calibration
    - necessary in many applications
    - world coordinates &rarr; (extrinsic parameters - rigid 3D to 3D) &rarr; camera coordinates &rarr; (intrinsic parameters - projection 3D to 2D) &rarr; pixel coordinates &rarr; (distortion parameters - non-linear 2D to 2D) &rarr; refined pixel coordinates

### Linear Sift-Invariant System

- most imaging systems can be modeled (well approximated) as a linear shift-invariant system
- a system L is LSI iff
    - $L\{g_1+g_2\} =L\{g_1\}+L\{g_2\}$ for any functions $g_1$, $g_2$
    - $L\{s \cdot g\} =s \cdot L\{g\}$ for any function $g$ and scalar $s$
    - $L\{Tg\} =TL\{g\}$ where $T$ is a translation operator
- any LSI system can be described by convolution with the PSF of the system

### Convolition

- continuous 2D: $g\otimes h = \int \int g(x-s,y-t)\cdot h(s,t)\;ds\;dt$
- discrete 2D: $g\otimes h = \sum \sum g(x-s,y-t)\cdot h(s,t)$
- cross-correlation 2D: $g\circledast h = \sum \sum g(x+s,y+t)\cdot h(s,t)$
- convolutio/cross-correlation is often used in template matching
    - for convolution flip the kernel upside down and left to right
    - detect maxima in the response field
- **normalization**
    - mean to 0
    - foreground positive, background negative (not 0)
    - we want the foreground to match with the pattern, but also the background to match the background
- what is it used for
    - edges
        - derivative of Gaussian <https://campar.in.tum.de/Chair/HaukeHeibelGaussianDerivatives>
    - blobs
        - Laplacian of Gaussians (LoG)
        - regional maxima higher than a certain threshold
    - corners
    - ridges
    - textures
- filter banks
    - several filters combination
        - maximum, sum, average


![Gaussian](https://s3.hedgedoc.org/demo/uploads/e1fd6dd4-166b-4291-aad4-fd4f2155f17f.jpeg)


:::success
**You should know**
- *What is an image, mathematically*
Digital image is a function $\mathbb{N}^d\to \mathbb{N}^c$ where $d$ is a number of dimensions and $c$ is a number of channels. 
- *Basic understanding of image formation process*
    - *What are linear shift-invariant imaging systems*
    Linear shift invariant imaging systems folow these conditions: 
        - $L\{g_1+g_2\} =L\{g_1\}+L\{g_2\}$ for any image functions $g_1$, $g_2$
        - $L\{s \cdot g\} =s \cdot L\{g\}$ for any image function $g$ and scalar $s$
        - $L\{Tg\} =TL\{g\}$ where $T$ is a translation operator
    - *What is point spread function*
    It is not possible to see/display infinitely small point. Point spread function defines how the light from point source is spread. PSF is a response of the imaging system to an infinitesimal point source.
    - *Give examples of common image deformations*
    Color shift, astigmatism, coordinates distortion.
    - *What is image calibration*
    Correcting geometrical deformations caused by the imaging system, perspective etc. 
- *Understand relation between convolution and cross-correlation*
    Convolution is identical to correlation if we flip both axes of the kernel. Discrete convolution is defined as $g\otimes h=\sum\sum g(x-s,y-t)h(s,t)$.
    - *Be able to demonstrate importance of normalization for template matching*
    Normalization is important because we want to match foreground to foreground AND background to background. If background is 0, mismatch does not contribute to the convolution/cross-corelation result, so we want a range [-1,1] instead of [0,1]. Normalization is also necessary condition to get the output in the same range.
:::

### Pixel

- point sample of a continuous function
- not little square!
- 2D: triple (x,y,g(x,y))

### Fourier transform

- linear operator over non-linear basis functions
- any periodic function can be rewritten as a sum of sines and cosines
- we want to analyze signals and images in the frequency domain
    - FT is a tool for converting signals/images from time or spatial domain to the frequency domain
- in frequency domain
    - low pass filter - low freqencies - main areas, no texture
    - high pass filter - high frequencies - mainly textures, flat areas grey
    - background correction, e.g. uneven illumination correction
        - low-pass filter detects "background" changes - we can remove the result from the original image (and add LPF mean)
        - same system - large opening (mathematical morphology)
- FT can be used for speeding up calculation of convolutions with large kernels
- FT useful for data normalization (e.g. uneven illumination correction)
- decomposing image to high and low frequency components (details and slow image changes)

### Sampling theory

- sampling
    - converting continuous signal into a discrete signal
    - sampling in space or time
    - $\mathbb{R}^n\rightarrow\mathbb{R}^c$ &rarr; sampling &rarr; $\mathbb{N}^n\rightarrow\mathbb{R}^c$
- quantization
    - reduncing number of values, typically converts discrete image into a digital discrete image
    - $\mathbb{N}^n\rightarrow\mathbb{R}^c$ &rarr; sampling &rarr; $\mathbb{N}^n\rightarrow\mathbb{N}^c$
- aliasing
    - distortions caused by improper sampling 
    - anti-aliasing filters remove high frequency content before sampling
    - can negatively influence ML systems
- optical transfer function (OTF)
    - characterizes spatial frequencies transmitted through the optical system
    - formally FT of the PSF
    - in real systems finite (=band-limited)
    - largest transmitted frequency = cut-off frequency
        - spatial cut-off frequency = "the smallest object resolvable by an optical system"
- **convolution theorem** - relation of PSF and OTF
    - convolution in spatial domain = multiplication in Fourier domain
- when is it possible to reconstruct continuous signal (image) out of its discrete samples?
- Shannon-Nyquist theorem
    - signal should be sampled at least twice the cut-off frequency
    - *To resolve all frequencies in a function, it must be sampled at least twice the highest frequency present.*
    - sampling twice the highest frequency prevents aliasing <https://brianmcfee.net/dstbook-site/content/ch02-sampling/Nyquist.html>
- imaging systems & sampling
    - scanner has more green detecting fields (higer sampling frequency for green wave-lengths), because people are more sensitive to green
    - sequential read - problem with moving objects
- **noise**
    - different types form various sources in the imaging system
        - Gaussian, poisson, salt&pepper, uniform
    - possible to remove with filtering - usually high frequency

## IMAGE ENHANCEMENT

- highlight or remove/attenuate certain features of interest in the image
- often subjective - "better looking" image
- used in **data augemntation**
    - enlarge training dataset

### Point transforms

- **normalization** in image processing
    - in ML usually mapping of image values to [0,1] range
- centering - setting median/mean/mode/average of min and max to 0
- scaling - value multiplication
- standardization - mean to 0, standard deviation to 
- activation functions
    - can be seen as point transforms
    - sigmoid, tanh, ReLU, leaky ReLU, ...
- intensity stretching
    - stretch the minimum and maximum intensity values present to the possible minimum and maximum intensity values
    - <https://samirkhanal35.medium.com/contrast-stretching-f25e7c4e8e33>
- histogram equalization
    - distribute values to achieve histogram in a shape of linear function
    - <https://towardsdatascience.com/histogram-equalization-5d1013626e64>
    - <https://en.wikipedia.org/wiki/Histogram_equalization>
- histogram specification
    - same histogram as other image
- adaptive histogram equalization
    - CLAHE - contrast limited adaptive histogram equalization
    - histogram equalization in local windows
    - applied only from certain contrast to avoid noise amplification
    - popular in deep learning

### Geometrical transforms

- shift, stretch, rotation, perspective distortion, spiral deformation
- common for augmentation and camera calibration
    - interpolation and resampling
    - can lead to aliasing if not used correctly

### Local transforms (filters)

- pixel values changed based on local neighborhood (e.g. blur)
- shift invariant linear filtering 
    - done by convolution
    - how to call convolution weights: filter, mask, kernel, template, window
    - if only positive weights: smoothing, positive and negative weights: difference
    - realtion to frequency domain filtering
        - low-pass filters - smoothing
        - high-pass filters - details
        - band-pass filters - difference filters

### Gabor transform

- motivation - FT tells us what frequencies are present, but we do not know when they are active
- localized frequencies
- GT = FT (sinus function) . (Gauss function)
- Gabor filters similar to human visual system
- sinusoidal signal of particular frequency and orientation, modulated by a Gaussian wave
- usualy kearned by the first layer of image processing ML system
    - can be visualised
    - can replace learnable parameters

### Color

- pixel values are vectors 
- we should be aware of color models
    - RGB, HSV, CMYK, ...

:::success
**You should know**

- *What is high-frequency and low-frequency information*
High frequency - frequently changing in a spatial domain, for example detailed patterns, noise. Low frequency - slowly changing in a spatial domain, for example large objects. It is possible to remove noise using low-pass filter, because it keeps only the low frequencies.

- *Fourier transform is a linear operator over non-linear basis functions*
Fourier’s idea: any periodic function can be rewritten as a sum of sines and cosines of different frequencies. Weighted sum is a linear operatin, sines and cosines are non-linear basis functions.
- *Why sampling theory is important*
Sampling theory tells us conditions under which we can reconstruct continuous function out of its discrete samples. We should sample at least twice the cut-off (highest) frequency.
    - *What is aliasing and why it can appear*
    It is caused by improper sampling, it cerates patterns which were not present in the original image. For example, if we have black pixel/white pixel pattern and we sample every second pixel, the result is only black/white.
- *How the images can be normalized and augmented*
    - Normalization:
        - scaling - value multiplication
        - centering - setting median/mean/mode/average of min and max to 0
        - standardization - mean 0, standard deviation 1
        - normalization - mapping values to [0,1] (or [-1,1])
        - histogram equalization
    - Augmentation:
        - roation, scaling, flipping, changes of brightness, contrast, noise addition
- *Gabor filters are related to human vision and often learned by artificial neural networks*
Capture what frequencies are present and where. It is similar to human vision, because we also focus on "major changes" in the image and where are the changes. They are also often learned by NN because of their descriptive power.
- *Relation of Fourier and Gabor transforms (at least intuitively)*
Fourier transform captures what frequencies are present, Gabor filters captures also location and orientation. Fourier transform uses sines and cosines, Gabor sines and cosines multiplied by Gaussian.
:::

## IMAGE RESTORATION

- inverse process to image degradation
- goal: reconstruct an original image from its degraded version
- objective process (in contrast to image enhancement)
- restoration approaches
    - deep learnig
    - analytical methods
- typical problems
    - deconvolution (not up-convolution!)
        - removing motion blur, astronomy, microscopy, ...
    - denoising
    - dehazing
        - remove "blur" or "fog"
    - inpainting
        - related to photo manipulation
    - super-resolution
        - up-scaling methods: bicubic, linear, sparce coding (searching for fitting patch in a database) ...
        - deep learning example: patch extraction and representation, non-linear mapping, reconstruction
            - network learns one-to-one mapping between low-resolution and high-resolution images
            - if we visualize the filters learned by the network on ImageNet, we see common filters for edge and blob detections, textures etc.

### Analytical methods

- long history of research and development
- forward (degradation) model explicitely described
    - e. g. what type of noise
- both deteministic or stochastic methods
- prior knowledge incorporated in regularization, incorporation is usualy easy
    - all information we have about the degradation model and imaged object
- examples: inverse filtering, expectation maximization & maximum likelihood estimation
- typically iterative and computationally demanding
- preferred if the degradation system is known (and invertible)

### Deep learning methods

- model $R_{\theta}$ trained from data, typically to minimize $||f-R_{\theta}(g)||^2$, where $\theta$ are model parameters
- most common architectures
    - convolutional encoder-decoder
    - U-Net
    - conditional GANs
- training slow, inference fast
- complicated incorporation of prior or domain knowledge
- research still active
- limits and pitfalls
    - hallucination problem
        - incorect result due to inappropriate trining data
        - extremly serious problem for using DL in discovery
        - network "remembers" patterns from training dataset and tries to fit them to test data
    - generalization problem
        - large memorization capacity of networks
        - prone to overfitting and overlearning
    - adversarial fragility problem
        - the neural networks can be tricked into producing completely different outputs after the application of imperceptible perturbations to their inputs

## WAVELETES & MULTIRESOLUTION PROCESSING

### Scale space theory

- add "scale" dimension
- formal theory for handling image structures at different scales
- motivation: objects exist at different scales, which are typically a priori unknown
    - What operators to use?
    - Where to apply them?
    - How large they should be?
- image represented as one-parameter family of smoothed images
    - Gaussian filter with larger and larger width
        - Why Gaussian?
            - scale-space axioms: non-enhancement of local extrema, linearity, shift/rotational/scale-invariance, semigroup structure (=closed and associative)
- feature extraction in scale-space
    - detect edges, blobs, ridges, corners, etc.
    - N-jet: set of scale-space derivatives up to order N
    - features expresses as polynomial combinations of normalized Gaussian derivatives

### Multiresolution pyramids

- structure for representing images at more than one resolution
- designed for image compression
- used in computer graphics and image analysis
- save images at different scales
- Laplacian pyramid
    - save just changes between layers (blur and down-sample image, up-sample and blur and save the difference of the result and the original image)

### Wavelet transform

- similar to FT - composition of basis functions
    - scaling and wavelet finctions
- possible to use for compression 
- wavelet = wave-like oscillation with an amplitude that begins at zero, increases or decreases, and then returns to zero one or more times
    - varying frequency and limited duration
- father wavelet - low-pass filters - scaling
- mother wavelets - high-pass filters - wavelet functions
- **integral transforms**
    - function decomposition as a linear combination of basis functions with coefficients
    - FT, Gabor, wavelets
- wavelet scattering
    - wavelet scattering network
        - like CNN, but wavelet instead of general convolution, modulus instead of ReLU and averaging instead of pooling
        - invariant to translation, rotation, scaling and deformation
        - no training, wavelets chosen (fixed kernels)
        - choose wavelets
        - extracted features very good
- main application field: **compression**
    - reduction of the space needed for storing/transferring images/videos
    - lossless (Huffman, LZW) and lossy (JPEG) compression models
    - machine learning not used much
        - main obstacles: speed and memory

## MATHEMATICAL MORPHOLOGY

- non-linear theory for analysis of spatial structures
- main structure: complete lattice $(L,\leq)$, **ordering**
    - basic operators: $\land: L\times L \rightarrow L$ infimum, $\lor: L\times L \rightarrow L$ supermum
    - rules: ordering, $\land$, $\lor$
    - operators preserving the rules: erosion, dilation
- many useful opertors:
    - dilation, erosion, hit-or-miss
    - opening, closing, granulometry
    - reconstruction, connected filtering
    - extrema extraction
    - watershed
- connected operators
    - used for image simplification
    - connected component trees
        - useful image representation for connected filtering (attribute filters)
- regional maxima detection
    - convolution &rarr; response - how to find maximum of the response?
        - reginal maxima = a connected set of pixels at level t, for wich outer boundary pixels have all values strictly lower than t
        - thresholding 
            - problem: what threshold?
                - if the background is not flat, the result might be corrupted
                    - mathematical morphology: white top hat (original image minus large opening)
                    - background correction: low-pass filter, large opening
                regional maxima &rarr; combine to get the result
        - extrema filtering
            - openings are helpful to filter maxima, e.g. by shape
- granulometry
    - morphological way to estimate image properties
    - the idea is to apply morphological filters with different sizes and to measure responses

:::success
**You should know**

- *Where machine learning is difficult to apply and why*
    - *Image restoration*
    Reverse problem, we want to obtain an "original" image from a degraded one. Analytical methods are usually iterative and computationally demanding, but is is easy to incorporation of prior or domain knowledge and it is preferred if the degradation system is known and invertible. Deep learning methods need long training and it is hard to incorporate previous knowledge. Limits are the hallucination problem (restores something which was not there), generalization problem (networks have large memoization capacity), and adversarial fragility problem (NNs can be tricked to produce completely different output even if the changes to input were imperceptible).
    - *Image compression*
    Main issues are speed and memory.
- *What techniques can be used for multi-scale processing*
Image is represented as a set of images at different scales.
    - *Gaussian scale-space (intuitively understand)*
    Different scales created using Gaussian filter with larger and larger width. Images are "more and more smooth", so the biggest structures are still visible but the details disappear. We can focus on a different level of details at different scales.
    - *Wavelet decomposition (intuitively understand)*
    Based on wavelets, wavelet = wave-like oscillation, varying frequency and limited duration. Function is represented as a composition of wavelets with coefficients. It is possible to ommit "smallest" wavelets - compression, dicarding details.
- *How to detect extrema*
Extrema is a connected set of pixels for which outer boundary pixels have all values strictly lower. How to detect? Correct background, find regional maxima as defined before, return where local maxima and above threshold.
- *Examples of problems mathematical morphology is often used for*
Mathematical morphology works with erosion and dilation operators. It could be used to image simplification, background correction, extrema filtering, or removal of border particles.
:::


## SEGMENTATION

- segmentation is a partitioning of the image into constituent regions or objects
- it is often defined as a pixel-wise classification problem and closely related to object recognition
- nowadays approached mostly by deep learning
- binary segmentation
    - background and foreground
    - often followed by region labeling
    - simplest segmantation method: thresholding
        - point-wise binarization
        - the threshold t can be selected manually or automatically 

## REPRESENTATION & DESCRIPTION

- object is a set of pixels
    - object pixels can be given by a binary image called mask
    - we can extract object pixel values from the original image
- boundary representation 
    - chain-codes
    - polygonal approximations
    - signatures
    - boundary segments
- boundary description
    - length, curvature, ...
    - shape numbers
    - Fourier descriptors
    - statistical moments
- region representation
    - sets of pixels
    - skeletons
- description
    - area, compactness, ...
    - topology
    - texture
    - statistical moments

## OBJECT RECOGNITION

- objects correspond to individual image regions
    - sometimes also called patterns
- recognition is often defined as a classification and localization task of single or multiple objects
- object recognition tasks
    - single object
        - image classification - single class per image (object = image)
        - sematic segmentation - class per pixel (object = pixel)
        - classification + localization - single object bounding box (object = subimage)
    - multiple objects
        - object classification - all labels of objects in the image
        - object detection - bounding boxes for all objects
        - instance segmentation - mask per object
        - panoptic segmentation - class per pixel + instance
        - semantic segmentation - class per pixel
- related problems
    - pose estimation - estimation of the object pose in images or videos
    - image retrieval - ideally content-based (query image and response images)
    - image captioning - generation of a natural language description of the image
    - image registration - connect two images (e. g. panorama)
- research still active
- crucial to understand the fundamental problems and principles how to combine them
    - often based on CNNs and attention networks

:::success
**You Should Know**
- *Object Recognition and Image Segmentation*
    - *What it is*
    Segmentation - classification of pixels into classes corresponding to some objects in the image, partitioning of the image into constituent regions. Object recognition - which objects are in the image and where.
    - *Examples of typical problems*
        - object classification - which objects are in the image
        - object detection - which and where
        - instance segmentation - mark pixels of each object
        - panoptic segmantation - as instance, but including background
        - semantic segmantation - mark pixels of each class
- *Basics of Convolutional Neural Networks*
    - *Convolutional and activation layers*
    Convolution - kernel applied to each position of the image at each channel, result activated. Number of parameters: $(C\times W\times H +1)\times F$ where $W$ and $H$ are kernel width and height, $C$ is the number of channels of the original image and $F$ is the number of filters (output channels), +1 is bias.
    - *Pooling layers*
    Downsampling, so the representation is smaller and more managable, max pooling takes maximum of defined region. Other types of pooling, for example blur pooling against aliasing. No parameters.
    - *Fully connected layers*
    Flatten the image into a vector, each entry is connected with each output. Number of parameters: $C\times W\times H\times O +1$ where $O$ is output size.
    - *Be able to calculate the number of parameters*
:::

## IMAGE CLASSIFICATION

- one category for one image
- input: image, output: class (domain specific)
- why is it difficult?
    - small inter-class variations
    - large intra-class variations
    - all pixels change when camera moves
    - occlusions, ambiguity, and background clutter
        - dog hidden under bed, dog and cat in one image, ...
- feature extraction
    - we want to get image characteristics
    - the general idea is to transform image content into local feature coordinates invariant to translation, rotation, scale, and other imaging parameters
    - feature descriptors
        - **scale invariant feature transform (SIFT)**
            - <https://www.youtube.com/watch?v=4AvTMVD9ig0>
            - extracts key-points (KP) and their descriptions (D)
            - KP: position, orientation, scale, strength; D: 128-element vector
            - invariant to image scale and roattion
            - robust to affine distortion, change in 3D viewpoint, addition of noise, change in ilumination
            - steps
                - scale space peak selection - potential locations for finding features
                    - Laplacian-of-Gaussian "blob" detector, different sizes
                - key point localization - accurately locating the feature key points
                    - precise peak localization by fitting a 3D quadratic function
                    - location, scale, and principle curvatures
                    - outlier rejection in case of small contrast or large ratio or principle curvatures
                - orientation assignment - assigning orientation to the key points
                    - local orientation is calculated for each key point based on the orientation histogram in a surrounding region from local image gradient directions
                    - in ambiguous cases with multiple peaks multiple key points are created
                    - the location, scale, and orientation define a local coordinate system for each feature
                - key point descriptor - describing the key point as a high dimensional vector
                    - feature vector with 4 x 4 x 8 = 128 elements
                    - gradient magnitude and orientation at each sample point
                    - samples are weighted by a Gaussian window and accumulated into orientation histograms summarizing content over 4 x 4 subregions in 8 directions
        - **speeded up robust features (SURF)**

### ImageNet

- image classification dataset
- database of annotated images
- used in ImageNet Large Scale Visual Recognition Challenge since 2010


## NON-CNN-BASED IMAGE CLASSIFICATION

### Bag of visual words

- idea: extract a set of local descriptors and assign each descriptor the closest entry in a visual vocabulary
- pipeline
    - feature extraction &rarr; n-dimanional vector
    - clustering (K-means)
    - visual vocabulary construction
    - histogram of visual words &rarr; assign class according to SVM

### Fisher Vectors + SIFT

- generalization of bag of words
- describe image by what makes it different from other images
- probabilistic visual dictionary modeled by a Gaussian mixture model (GMM) (instead of K-means)

## CONVOLUTIONAL NEURAL NETWORKS (CNNs)

- stacking of convolutional, pooling and fully connected layers
- convolutional layers
    - preserves spatial structure
    - apply convolutional filter - "slide over the image spatially computing dot products"
    - keep the full depth of the input
    - result after convolution at all positions: activation map
    - stride - "jumps" of the filter
        - stride > 1 - subsampling
    - padding - we want to avoid volume skrinking in border areas
        - zero padding - if we normalize the image to [-1,1] range, 0 neutral values, they do not contribute to template matching  
    - output size: floor((image size - filter size + 2 padding) / stride + 1)
    - activation layer
        - after each convolution layer (assumed that an activation immediately follows a convolution, so it is sometimes omitted in a network architecture diagram)
        - activation function ReLU, leaky ReLU, sigmoid, tanh, maxout, ...
- pooling layers
    - subsampling, make image smaller
    - operates over each activation map independently
    - example: max-pooling (not shift-invariant), blur pooling
- fully connected layers
    - every input connected to every output
    - stretch image to vector ($32 \times 32 \times 3$ stretch to $3072 \times 1$)
    - number of weights = input vector length * output vector length

## INFLUENTIAL CNN-BASED ARCHITECTURES

### LeNet-5

- handwritten character recognition
- 2[convolution-subsamplig]+3[fully connected]
- main idea why it works
    - motive detection - convolution
        - How to detect mutual relations? Build more levels.
    - translation equivariance
        - How to detect a feature is present? Global pooling.

### AlexNet

- two different parts of processing (originally two processing units)
- input RGB image, central patch 256x256, random patches, no preprocessing
- 60M parameters
- first use of ReLU
- used NORM layers (not common anymore)
- heavy data augmentation
- model averaging

### ZFNet

- like AlexNet, but conv 7x7 stride 2 instead of 11x11 stride 4 and other number of filters changes in conv 3, 4 and 5
- a novel visualization scheme introduced which helped to find problems in the initial layer
    - showed that visualization of activations can help with model understanding and its improvement

### VGGNet

- deeper network, shows that bigger models work better
- vertical stacking = small kernels & more layers
- smaller kernels on more layers have the same effective receptive field as a large kernel, but fewer parameters
    - receptive field = what locations taken into account for one value
- smaller width and height, more filters (higher depth)
- memory consumption in the beginning, most parameters in the end
- popularized 1×1 bottlenecks and global average pooling instead of FC layers

### GoogLeNet 

- deeper network
- inception modules - horizontal stacking
    - idea: apply parallel filter operations on the input from the previous layer (Network In Network idea)
    - naive
        - multiple receptive feature sizes, pooling operation
        - concatenate all filter responses channel-wise
        - problem: concatenation - large size
            - the module is computationally expensive and because the pooling layer preserves depth, the volume depth after concatenation only grows
    - improvement
        - bottleneck layer - 1x1 conv with less filters than original depth
            - added to reduce dimensionality
- computationally efficient deep networks - only 5M parameters
    - 12x less than AlexNet, 27x less than VGG16
    - classification output: No expensive FC layers! Average pooling is applied globally across all feature maps.
- auxiliary classification outputs to inject additional gradient at lower layers, used only for training
- popularized 1×1 bottlenecks and global average pooling instead of FC layers

### ResNet

- first network better than humans in ILSVRC classification competition
- very deep network using residual connections
    - in previous networks problem with vanishing gradient
        - the deeper model performed worse, but it is not caused by overfitting
        - hypothesis: the problem is an optimization problem, deeper models are harder to optimize
        - idea: the deeper model should be at least as good as the shallower model
            - solution: copy layers from shallow laers by identity mapping
- use network layers to fit a residual mapping instead of directly trying to fit a desired underlying mapping
    - fit residual F(x) = H(x) -x instead of fitting H(x) directly
- full architecture
    - stack of residual blocks
    - every residual block has two 3×3 CONV layers
    - for the same output feature map size the layers have the same number of filters
    - feature map size is halved the number of filters is doubled
- able to train very deep networks without degrading
- improvements
    - multi-scale ensembling of Inception, Inception-ResNet, ResNet, Wide ResNet models
- shows we can train extremely large models and that we are limited only by GPU & memory
- after ResNet the focus shifted to efficient networks
- ResNet and SENet are currently the default choice

### SENet

- Squeeze and Excitation (SE) blocks
    - idea: not all filters are equally important
    - sqeeze - global average pooling
    - excitation - fully connected layer
    - "channel-wise attention"
    - predicts the importance for features, multiply features by the importance
        - "small network in network" for prediction of importance - attention
- adaptive recalibration of channel-wise feature responses
- ResNet and SENet are currently the default choice

### EfficientNet

- balancing of network depth, width, and resolution lead to better performance

:::success
**You Should Know**
- *Key ideas of influential CNN architectures*
    - *VGG*
        - *Vertical stacking – more layers & small kernels*
            - smaller kernels on more layers have the same effective receptive field as a large kernel, but fewer parameters
            - more repated convolution layers before max-pooling
    - *GoogLeNet*
        - *Horizontal stacking and inception modules*
            - inception module consists of more parallel operations
            - parallel application of more different convolutions (1x1, 3x3, 5x5) and max pooling, the result is concatenated filter-wise
            - added dimensionality reduction to prevent too much growing of dimensions
        - *Global pooling*
            - avarage polloing, reduces $C\times H\times W$ to $C\times 1\times 1$, used to get rid of fully connected layers
        - *Bottlenecks*
            - 1x1 convolutions used for dimensionality reduction in the inception module
    - *ResNet*
        - *Residual block*
            - deep models have more representation power (aparmeters), but are hard to optimize
            - copying the learned layers from the shallower model and setting additional layers to identity mapping
            - residual block:
                - block input: x
                - path 1: CONV, ReLU, CONV (F(X))
                - path 2: identity
                - block output: F(x)+x, ReLU
    - *SENet*
        - *Squeeze and excitation block*
            - predicts importance of features and multiplies the features by the importance
            - squeeze - global average pooling
            - excitation - fully connected layer 
:::

## SEMANTIC SEGMENTATION

- classification problem on the pixel level
    - training: each pixel is labeled with a semantic category
    - inference: assign a label to each pixel of the image based on its semantic meaning
- idea: sliding window
    - we need context - subimage patches - classify the central pixel with a CNN
    - Poblem? Very inefficient! Many repeated computations on overlaps.
    - tried, papers published abot this idea
- another idea: design a network with only convolutional layers without downsampling to make prediction for all pixels at once
    - Convolutions at the full image resolution are very expensive!
- better: reduce feature map sizes or reduce the number of convolution operations
    - reduce feature map sizes 
        - but semantic segmentation requires the same output and input size 
        - upsampling
    - reduce the number of convolution operations
        - But how to keep a reasonable effective receptive field?
        - atrous (with holes) convolution
- invariance vs. equivariance
    - invariance - apply function, same result
        - pooling layers are approximately shift invariant
            - global pooling is invariant 
    - equivariance - apply function, appropriately shifted result
        - convolutional layers are (mostly) shift equivariant

### Fully Convolutional Networks

- idea: create fully convolutional layer instead of fully connected layer at the end
- design a network with downsampling and upsampling inside the network
    - downsampling: pooling or strided convolution
    - upsampling? in the first version no upsampling

### Upsampling
- interpolate discrete values by a continuous function
    - How?
        - convolution with a kernel
        - continuous function $f(x)$ can be created from discrete values $f_k$ by convolving with a kernel $\phi(x)$ as $f(x) = \sum_k f_k \phi(x-k), x\in \mathbb{R}, k \in \mathbb{Z}$
            - 1D $\phi(x)$ nearest neighbor, linear interpolation, cubic
            - higher orders are possible, but synthesis functions must be interpolants (bilinear, bicubic, ...)
    - interpolation of scattered data
        - triangulation based, inverse distance weighted, radial basis functions
- make the geometric transformation (e.g., scaling)
- resample the continuous function at new sample positions
- How to use in CNNs?
    - direct up-sampling layers - no learnable parameters
        - unpooling
            - bed of nails
            - nearest neighbor
        - max unpooling
            - remember position of maxima
    - learnable resampling layers - strided convolutions (down), transposed convolutions (up)
        - transposed convolution
            - filter is moved by given stride in the output for every input pixel
            - input values are weights
            - overlapeped pixels are summed
            - other names: deconvolution, upconvolution, fractionally strided convolution, backward strided convolution

### Atrous or Dilated Convolution

- we want to increase receptive field
- dense evaluation of layers lead to shift-equivariance
    - pooling/resampling is not needed
    - similar ideas as in undecimated wavelet transform

### U-Net

- popular in biomedical imaging
- many variants and improvements exist
- contains encoding (convolution) and decoding (upconvolution) part with skip connections
    - copy information from encoder layers and concatenate with upsampled features

### Dilated Residual Networks

- "ResNet with dilated convolutions"

### Deformable Convolutional Networks

- deformable convolution
    - generalization of the dilated convolution
- offsets instead of regular grid 
- predicting offsets in another layer
- receptive field is adaptive, not regular grid

## SINGLE OBJECT RECOGNITION

- classification + localization

:::success
**You Should Know**
- *Semantic segmentation can be understood as pixel level classification*
    - *FCN*
        - Fully convolutional network. We want a classification output for each pixel, so we design a network with only convolutional layers without downsampling to make prediction for all pixels at once. It is too expensive.
    - *U-Net*
        - Downsample/endocing (conv,conv,maxpool) and upsample/decoding (up-con,conv,conv), skip connections copy feature map of each size from encoding to decoding part, where it is concatenated.
    - *DCN*
        - Deformable convolutional networks, based on deformable convolution. Deformable convolution - learn weight and offset (receptive field is adaptive). 
- *You should understand*
    - *Upsampling*
        - From smaller grid to bigger, for example unpooling (copy value to other cells), max unpooling (remember position of maximum) or transposed convolution.
    - *Transposed convolutions*
        - Lernable upsampling, filter is moved by given stride in the output for every input pixels. If input 2x2 and stride 2, then output 4x4. Multiply kernel with input entry, write to output, if there is alread something, sum.
    - *Dilated (atrous) convolutions*
        - Kernel with empty spaces:
        ![dilated convolution](https://pub.mdpi-res.com/micromachines/micromachines-12-00545/article_deploy/html/images/micromachines-12-00545-g001.png?1620979121)
    - *Difference between invariance and equivariance*
    Invariance means that the result (e.g. class) is the same regardless the transformation (translatied or rotated, it is still a cat). Equvariance means that the result is changed according to the transformation (e.g. mask of an apple is shifted when the apple is shifted in the input).
:::

### Loss functions

- loss function = penalization of dissimilar results
    - application dependent
- error or cost function
    - often average loss for $m$ examples $E(\theta)=\frac{1}{m}\sum_{k=1}^m L(f(x^{(k)};\theta),y^{(k)})+L_{reg}(\theta)$ where $x^{(k)},y^{(k)}$ are training pairs, $\theta$ are trained parameters of model $f$ and $L_{reg}$ is an optional regularization function
    - normalization term $\frac{1}{m}$ is unnecessary
    - regularization: higher penalty to complex curves - prevents overfitting

**Categorical Loss Functions**

- typical for classification problems
- classifier outputs scores, apply softamx &rarr; predictions
- denote $c$ ground truth class, $1_c(t)$ indicator function $t=c$
    - indicator function causes that only predictions of ground-truth classes are penalized
- cross entropy: $L(p,c)=-\sum_{t=1}^c 1_c(t)\log(p_t)$ 
- **focal loss**: $L(p,c)=-\sum_{t=1}^c 1_c(t)(1-p_t)^\gamma\log(p_t)$ where $\gamma$ is fixed
    - generalization of cross entropy
    - useful in object detection
    - designed to alleviate class imbalance problem
    - "tolerate less certain predictions"
- How to change the loss function to be able to classify multiple objects?
    - Replace softmax with sigmoid
    - does not work really well in practice

**Segmentation Losses**

- "classificatin on pixel level"
- types of losses - distribution-based, compounded, region-based, boundary-based
- **region-based losses**
    - ground truth - set of pixels X
    - prediction - set of pixels Y
    - false negatives $FN= X \setminus Y$, true positives $FN= X \cap Y$, false positives $FN= Y \setminus X$, true negatives $TN = \overline{X \cup Y}$
    - precision $\frac{TP}{TP+FP}$, recall $\frac{TP}{TP+FN}$
    - IoU, Jaccard index $\frac{TP}{TP+FN+FP}$
    - F1 score, dice index $\frac{2}{\frac{1}{precision}+\frac{1}{recall}}=\frac{2TP}{2TP+FN+FP}=\frac{2|X\cap Y|}{|X|+|Y|}$
        - harmonic mean of precision and recall 
    - generalization: Tversky index $Tversky(\alpha,\beta)=\frac{TP}{TP+\alpha FN+\beta FP}$
        - $\alpha=0,\beta=1$ precision
        - $\alpha=1,\beta=0$ recall
        - $\alpha=1,\beta=1$ IoU
        - $\alpha=1/2,\beta=1/2$ F1 score

**Regression Loss Functions**

- used to predict real-valued quantities
- mean absolute error (MAE, L1 loss)
    - $MAE(y,p)=\frac{1}{n}\sum_{i=1}^n|y_i-p_i|$
- mean squared error (MSE, L2 loss)
    - $MSE(y,p)=\frac{1}{n}\sum_{i=1}^n(y_i-p_i)^2$
- mean squared logaritmic error
    - $MLSE(y,p)=\frac{1}{n}\sum_{i=1}^n(\log(y_i+1)-\log(p_i+1))^2$
    - if small values are more important then the large ones

**Classification + Localization**

- localization treated as a regression problem
    - L2 loss
- class prediction - cross-entropy
- how to combine together? multitask loss, e.g. sum

**Multiple Object Recognition Tasks**

- eqch image needs different number of outputs
- idea: apply a CNN to many different crops of the input image and classify each crop
    - problem: the network should be applied to a huge number of locations, at many different scales and aspect ratios, which is computationally expensive

### Viola & Jones Face Detection

- first real-time object detector
    - developed for faces and often still in use
- three key building blocks
    - integral image for fast calculation of Haar-like features
        - integral image - sum of values in rectangle from top left corner to current position
        - fast template matching
    - selection of important visual features using AdaBoost
        - edge features, line features, four rectangle features
    - focus-of-attention mechanism
        - combination of classifiers in cascades
            - start with prominent features
        - patch can be rejected in each phase

### HOG for Human Detection

- dense grids of histograms of oriented gradients
    - counts occurrences of gradient orientations in localized image portions
- how?
    - model average
    - SVM positive and negative weights
    - HOGs weighted by the SVM weights

### Deformable Part Models

- can be used with modern object detectors
- can be reformulated as neural network
- objects represented by a collection of parts arranged in a deformable configuration
    - spring-like connections between some pairs of parts
- models are defined by subwindows of a feature pyramids
- deformable part model
    - coarse root model - HOGs
    - part filters - HOGs
    - part locations - distance maps

### One stage vs two stage detectors

- one stage - input image, run network, gives output
- two stage - one stage, network, proposals, in second stage specialized network for each proposal
- what is shared in both approaches?
    - input - image, patch, pyramid
    - backbone - feature extraction network - VGG, ResNet, ...
    - neck - related to multiscale processing
    - prediction head
        - prediction of changes of bounding boxes
        - classification

**Spatial Pyramid Pooling**

- takes feature map, global pooling of the whole area
- separate to 4 subimages, pool each subimage
    - repeat the process 
- for various sizes of input images, same size of features 
- sumarization of the image

**Feature Pyramid Networks (FPN)**

- feature extractor
- bottom-up and a top-down pathway
    - bottom-up pathway 
        - convolutional network for feature extraction 
        - the spatial resolution decreases, the semantic value for each layer increases.
    - top-down pathway 
        - higher resolution layers from a semantic rich layer
        - upsampling
    - corresponding levels connected by 1×1 bottlenecks
- prediction on various levels
- leads to better detection of small objects

**Anchor boxes**

- allowed multiple object detection
- first appeared in Faster R-CNN
- used often in one-stage detectors
- for each pixel of input image, we generate k anchor boxes of various sizes and shapes
    - modify parameters to fit in the object
- outputs of network
    - for each pixel k anchor boxes
        - is there an object, class of object
        - change in coordinates
    - parameters relative to a window - translation invariance

:::success
**You should know**
- *Common loss functions*
    - *What they can be used for*
    - cross-entropy: $L(p,c)=-\sum_{t=1}^{n_c}1_c(t)\log(p_t)$ where $c$ is the ground truth class and $p$ predictions
        - other formulation: "loss is -log(p_{correct})"
        - typical for classification problems
    - focal loss: $L(p,c)=-\sum_{t=1}^c1_c(t)(1-p_t)^{\gamma}\log(p_t)$
        - classification problems with class imbalance, tolerate less certain predictions
    - IoU: $IoU=\frac{TP}{TP+FP+FN}$
        - segmantation
    - precision: $P=\frac{TP}{TP+FP}$
        - segmantation
    - recall: $R=\frac{TP}{TP+FN}$
        - segmantation
    - F1: $\frac{2PR}{P+R}$
        - segmantation
    - regression loss functions
        - MAE, MSE, MSLE
        - used for localization
- *Main ideas of single and multi-object detection*
    - *Integral images*
        - sum of values in rectangle from top left corner to current position 
            - easy to evaluate sum of values in all rectangles
            - just three aritmetic operations regardless of the window size
    - *Histograms of oriented gradients*
        - to get highest contrast areas, dense grid of histograms of oriented gradients
        - counts occurrences of gradient orientations in localized image portions
    - *Principles of modern CNN-based detectors*
        - *Multi-scale processing (FPN, SPP)*
            - Spatial Pyramid Pooling 
                - global pooling of whole feature map, quarters of feture map, quarters of quarters... &rarr; crates summarization of the image
                - results are concatenated in a vector
            - Feature Pyramid Networks
                - upsampling and downsampling path, corresponding levels connected, predictions for varios levels
                - better detection of smaller objects
        - *Anchor boxes*
            - allowed multiple object detection
            - for each pixel of the input, generate anchor boxes of various sizes and shapes
                - can be generated other way as well
            - localization is then change in anchor box coordinates
        - *Difference between one-stage and two-stage detectors*
            - one stage: input, network, output
            - two stage: input, region proposals, network used for each proposal, output
:::

### Evaluation of Object Detection

- for one detection - IoU
- How to handle multiple detections?
    - The evaluation metrics differ in details, but the basic ideas are the same in popular datasets.
    - determine TPs and FPs with greedy algorithm
        - find the best match among the ground truth boxes for each predicted box, using predefined minimal IoU
        - mark the best match as TP and ignore already matched GT boxes
    - common evaluation strategy: area under precision-recall curve (average precision, AP)
        - in relation to threshold IoU
        - often approximated

### How to limit the number of windows?

- exhaustive search - we generate all possible boxes
    - The network should be applied to a huge number of locations, at many different scales and aspect ratios, which is computationally expensive!
    - We need to save computations somewhere!
- region proposal
    - find a small set of boxes with high recall
    - often based on heuristics, e.g., blobs in a scale space
    - relatively fast to rum
    - selective search
        - hierarchical combination of segmentation and exhaustive search

### Non-Maxima Suppression

- problem: object detectors often output many overlapping detections
- solution: raw detections post-processing
- algorithm
    - select next highest-scoring box
    - eliminate lower scoring boxes with IoU > threshold (e.g. 0.7)
    - repeat if any box remain
- new problem: we may eliminate good boxes if objects are highly overlapping
    - no good solution

### R-CNN: Region Based CNN

- how it works?
    - input: image
    - proposal method: regions of interest (RoI)
    - warped RoIs: "small squares"
        - crop and resize
    - CNN for each warped Roi: class prediction and bounding box regression
        - we have to transform coordinates back
- problem: very slow
    - we calculate features for each proposal again (they may overlap)
    - solution: run CNN before warping &rarr; fast R-CNN

### Fast R-CNN

- how it works?
    - input: image
    - backbone CNN: extraction of image features
        - most computation happens in backbone network
        - saves work for overlapping RoIs
    - proposal method: regions of interest (RoI)
    - warped RoIs - rescaled to small squares
        - how to crop and resize features? RoI pooling
    - small CNN for each warped Roi: class prediction and bounding box regression
        - leightweight and fast
    - we have to transform coordinates back
- significant improvement in both training and test time
- problem: runtime dominated by region proposal

**RoI pooling**

- project proposal onto features 
- "snap" to fit to grid cells
- divide into a grid of roughly equal subregions
- max-pooling within each subregion
- Region features have always the same size even if input regions have different sizes.
- problem: slight misalignment due to snapping; different-sized subregions weird
    - RoI align boxes
        - no snapping
        - divide into equal sized subregions 
            - may not be aligned to grid!
        - sample features at regularly-spaced points in each subregion using bilinear interpolation

### Faster R-CNN: Learnable Region Proposals

- try to slove problem that runtime dominated by region proposal
- insert Region Proposal Network (RPN) to predict proposals from featurs
    - instead of selective search
- could be used for instance segmentation or pose estimation as well
    - mask prediction for each RoI

**RPN**
- use an anchor box of fixed size at each point in the feature map
    - or usually more anchor boxes
- for each point predit if the anchor contains an object
- for positive boxes predict a box transform from anchor box to object box

### Single Stage Object Detection

- faster, less accurate
- RPN: classify object/no-object
- single-stage: input, anchors, modify directly parameters of anchor boxes
    - output anchor category + box transforms
- YOLO
    - first one stage detector
    - CNN for feature map, fully connected layer
    - output: multiclass classification + bounding box regressor
- SSD
    - same principle, multiscale
    - differentt scales of features, after each stage classification

### RetinaNet

- famous one stage detecto
- introduced focal loss to adress class imbalance
- include feature pyramid network
    - for each level class+box subnet
- based on anchor boxes

### Object detection without anchor boxes

- CornerNet
- detects heatmaps + embeddings
    - heatmaps for top left and bottom right corners
    - corner pooling - new layer
- CenterNet
    - three key points: introduces center and cascade corner

:::success
**You should know**

- *Evaluation of object detectors*
    - *IoU*
        - intersection over union, $IoU=\frac{TP}{TP+FP+FN}$
        - only usable for one detection
    - *AP, mAP*
        - AP is average precision, area under the precision/recall curve, often approximated
            - in relation to a threshold IoU 
        - mAP: mean AP over all classes
- *Anchor boxes, ROI Pool, ROI Align, region proposals*
    - anchor boxes: predefined boxes, prediction is if there is something and a change in their coordinates
    - region proposals: way to limit the number of windows considered for further processing, "there might be something" - often based on heuristics (like blob detection)
    - ROI pool vs align: ROI after CNN - we have to project proposal onto the features (CNN output)
        - Pool snaps proposal to grid cells, divides the area into approximately same areas. Align does not use snapping, samples features at regularly spaced grid.
- *R-CNN, Fast R-CNN, Faster R-CNN, Mask R-CNN*
    - R-CNN - region proposals, CNN for each, class+bbox
    - Fast R-CNN - CNN, region proposals, small CNN for each,  class+bbox
    - Faster R-CNN - CNN, region proposal netowrk (predict propoals from features), small CNN for each,  class+bbox
    - mask R-CNN - add mask prediction to the last stage of Faster R-CNN
- *YOLO, SSD, RetinaNet*
    - YOLO - first one-stage detector, CNN for feature map, fully connected layer
    - SSD - same as YOLO, but multiscale - conv-predict-conv-predict-...
    - RetinaNet - famous one stage detector based on anchor boxes, introduced focal loss, include feature pyramid network
- *Non-maxima suppression*
    - select the highest scoring box, discard boxes with high overlap
:::

## GENERATIVE MODELS

### Supervised vs. Unsupervised Learning

- supervised: data and labels
    - goal: learn function to map data to labels
    - examples: slassification, regression, object detection, semantic segmentation, image captioning, ...
- unsupervised: just data, no labels
    - goal: learn some underlying hidden structure of the data
    - examples: clustering, dimensionality reduction, feature learning (autoencoders), density estimation, ...

### Discriminative vs. Generative Models

- discriminative
    - learn a probability distribution p(y|x)
    - probability of label with given data
    - competition between labels, not images
    - no way to handle unreasonable input
    - usage: assign labesl to data, feature learning with labels
- generative
    - learn a probability distribution p(x) 
    - no labels, just data, goal is to generate similar examples
    - all possible images compete between each other for probability mass
    - model can assign small values to rare or unreasonable inputs
    - usage: detect outliers, feature learning without labels, sample to generate new data
- conditional generative model
    - learn a probability distribution p(x|y)
    - for each label learn the distribution
    - usage: assign labels while rejecting outliers, generate new data conditioned on input labels

**Bayes rule**
$p(x|y)=p(x)\frac{p(y|x)}{p(y)}$

### Taxonomy of Generative Models

- explicit density (model can compute p(x))
    - tractable density - construct whole distribution precisely
        - autoregressive models
    - approximate density
        - variational autoencoders
        - Boltzman machine
- implicit density (model does not explicitly compute p(x))
    - Markov chain
    - direct

## AUTOREGRESSIVE MODELS

- goal: write down an explicit function for p(x)=f(x,W)
    - f can be neural network, W its parameters
- given dataset $x^{0},x^{1},...x^{N}$, train model by maximize probability of training data (maximum likelihood estimation)

$$W^*=argmax_W\prod_i p(x^{(i)})=argmax_W\sum_i \log p(x^{(i)})=argmax_W\sum_i \log f(x^{(i)},W)$$

- this gives us a loss function
- assume $x$ consists of multiple parts, $x=(x_0,x_1,...,x_T)$$
    - $x$ is an image, $x_i$ is a pixel
    - $p(x)=p(x_0,x_1,...,x_T)$, we can use chain rule
    - $p(x)=p(x_0,x_1,...,x_T)= p(x_0)p(x_1|x_0)p(x_2|x_0,x_1)...= \prod_{t=0}^Tp(x_t|x_1,...,x_{t-1})$
    - Sequence modeling with an RNN!

**Pixel RNN**
- generate image pixels one at a time
    - starting at upper left corner
- problem: very slow! (both training and inference)
    - improvement: **Pixel CNN**
        - still generate image pixels starting from corner
        - dependency on previous pixel modeled using a CNN over context region
- both models relatively simple
- outputs look reasonable (from distance)
- pros: 
    - explicit likelihood
    - explicit likelihood of training data gives good evaluation metric
    - good samples
- cons:
    - sequential generation, slow
- PixelRNN and PixelCNN explicitly parametrize density function with a neural network

### Variational Autoencoders

- define a density that cannot be explicitly calculated and only approximate it
    - possible to define a lower bound that can be optimized
    - by improving the lower bound we improve the variational model

**Autoencoders (non-variational)**

- unsupervised method for learning feature vectors from raw data x, without any labels
    - "encode image with lower dimensionality"
- use the features to reconstruct the input data with a decoder
- loss function - how far is the reconstruction from the original image?
- after training, we can remove the decoder and use encoder for downstream task (e.g. classification)
    - initialize supervised model
- autoencoder learns latent features 
    - without labels
- not probabilistic
    - no way how to sample data from learned model
    - irregular latent space prevent us from using autoencoders for new content generation

**Variational Autoencoders**

- modify decoder and encoder networks to behave in a probabilistic way
    - allow sampling from the model to generate new data
- enncoder: input data x, gives distribution over latent codes z
- decoder: inputs latent code z, giver distribution over data x
- we want to have a generative model that maximize likelihood of data
    - direct calculation is intractable, therefore we use Bayes’ rule
    - $p_\theta(x)=\frac{p_\theta(x|z)p_\theta(z)}{p_\theta(z|x)}$
    - it is not possible to compute $p_\theta(z|x)$, so we train encoder to estimate it
- train encoder and decoder together to maximize the variational lower bound on the data likelihood
    - loss = data reconstruction - KL-divergence (similarity of distributions , regularization of the latent space)
- training
    - maximizing variational lower bound
    - run input through encoder, get distribution over latent codes
        - encoder output should match the prior (e.g. Gaussian distribution)
            - KL-divergence in the loss function
    - sample code from encoder output
    - run sampled code throuh decoder
        - original input data should be likely under the obtained distribution
            - first term in the loss function
- generating data
    - sample latent code from prior distribution
    - run through decoder, sample from obtained distribution
- use for image modification - modify latent code
- pros:
    - allows inference of q(z|x), can be useful feature representation for other tasks
    - very fast image genaration
    - learn rich latent codes
    - principled approach to generative models
- cons:
    - generated images often blury and low quality
    - lower bound maximization (autoregressive models maximizes directly p(data))

**Combining VAE & Autoregressive - Vector-Quantized VAE**

- train VAE-like model to generate multiscale grids of latent codes
- use a multiscale pixelCNN to sample in latent code space

## GANs – GENERATIVE ADVERSARIAL NETWORKS

- assume we have data drawn from some distribution, we want to sample from the distribution
- idea:
    - introduce a latent variable z with simple prior p(z), sample from p(z) and pass the sample to generator
    - generate x from generator distribution
        - we want generator distribution same as data distribution
    - give one real and one generated sample to a discriminator network
    - discriminator decides, which is fake
- train generator and discriminator together - minimax game
    - no loss function
- problem: vanishing gradients
- training is not easy
- **conditional GANs**
    - semantic map as an input

:::success
**You should know**
- *Autoencoders*
An autoencoder is a special type of neural network that is trained to copy its input to its output. It is an unsupervised model and it learns how to transform input to some latent space (with less dimensions) and reconstruct back. It can be used as feature extractor. Loss function - how far is the reconstruction from the original image. Can be used for encoding as feature extraction after removal of the decoding part.
![autoencoder](https://lilianweng.github.io/posts/2018-08-12-vae/autoencoder-architecture.png)
- *Generative Models*
Learn a probability distribution of data (without labels), goal is to generate similar examples. Conditional generative model - based on class, learn the distibution for each class.
    - *Autoregressive models (PixelRNN, PixelCNN)*
    Explicitly estimate the density p(x)=f(x,W) where f can be neurak network with weights W, maximizing probability of training data using maximum likelihood estimation.
    PixelRNN - model probability based on probabilities of other pixels. Generate one pixel at time starting at upper left corner: $h_{x,y}=f(h_{x-1},{y-1},W)$. Problem: very slow.
    PixelCNN - similar to PixelRNN, but dependency modeled using a CNN over context region. Still very slow because of generating one by one.
    Good samples, explicit likelihooh computation.
    - *Variational Autoencoders (VAEs)*
    VAE trains an encoder to estimate probability distribution of latent codes given the data. Latent vector is sampled from the distribution and ecoder network is trained to give distribution over the original data given the latent vector.
    Direct MLE maximization is not possible, so we improve its lower bound as a loss function.
    When the network is trained, it is possible to sample from the latent distribution and use the decoder to generate images.
    Usage: image generation, image editing (modifiing the latent code)
    ![VAE](https://lilianweng.github.io/posts/2018-08-12-vae/vae-gaussian.png)
    - *Generative Adversarial Models (GANs)*
    Do not model probability distribution p(x), but allow us to draw data from the distribution. Train together generator and discriminator, generator generates image from a prior latent distribution (random noise), discriminator gets one generated and one real image and tries to decide which one is fake. No loss function, minimax game.
    Usage: image genaration, vector math (smiling woma - woman + men = smiling man), style transfer
    ![GAN](https://sthalles.github.io/assets/dcgan/GANs.png)
    - *Conditional GANs*
    Semantic map as an input of GAN, edges to fotot, BW to color, background removal, ...
- *Relation to other models*
Other models in this course were discriminative - learned probability distribution p(y|x), probability of some labels given the data.
:::

## ATTENTION

- attention in visual system
    - dynamic selection process
    - assists in analyzing complex scenes
    - "where are you looking"
    - human vision
        - central high-resolution vision
        - largest concentration of cone cells
- idea: daptive weighting of the features according to the importance in the input
- attention types
    - channel attention
        - emphasize important features (channels)
        - example: squeeze & excitation
            - channels that often represent different features are weighted
            - more important features get higher weights
    - channel & spatial
    - spatial attention
        - example: RAM – Recurrent Attention Model
            - multiresolution patches centered at a location l
            - patches at various locations used for feature learning
            - features used for prediction in RNN model
        - example: image captioning with attention and RNN
            - CNN to get features
            - create state vector, use it to compute alignment scores and attention weights
            - features + weights = context vector, new state vector
            - prediction, use state vector to compute new alignment scores and attention weights, repeat
    - spatial & temporal 
    - temporal attention
    - branch attention

### Attention layer

- inputs
    - guery vectors $Q$ (shape $N_q \times D_q$)
    - input vectors $X = (x_i)$ (shape $N_x \times D_q$)
    - attention function $f_{att}$, usually dot scaled product
- computations
    - similarities $E$ (shape $N_q \times N_x$), $e_i=f_{att}(q,x_i)= q\cdot (x_i /\sqrt{D_q})$
    - attention weights $a=softmax(e)$ (shape $N_x$)
    - output vector $Y=AX, y_i=\sum_j a_{ij}\cdot x_j$ (shape $N_q \times D_q$)

### Self-attention layer

- input vectors used twice in the attention layer
    - separate key and vlaues matrix learned from the input matrix
    - **learn queries from the input**
- inputs
    - guery matrix $W_q$ (shape $N_x \times D_x$)
    - input vectors $X = (x_i)$ (shape $N_x \times D_q$)
    - attention function $f_{att}$, usually dot scaled product
    - key matrix $W_k$ (shape $D_x \times D_q$)
    - value matrix $W_v$ (shape $D_x \times D_v$)
- computations
    - query vectors $Q=XW_q$ (shape $N_x \times D_q$)
    - key vectors $K=XW_k$ (shape $N_x \times D_q$)
    - value vectors $V=XW_v$ (shape $N_x \times D_v$)
    - similarities $e$ (shape $N_x$), $e_i=f_{att}(q,x_i)= q\cdot (k_i /\sqrt{D_q})$
    - attention weights $a=softmax(e)$ (shape $N_x$)
    - output vector $Y=AV, y_i=\sum_j a_{ij}\cdot v_j$ (shape $N_q \times D_q$)
- self attention is permutation equivariant
    - In order to make processing position-aware, concatenate or add positional encoding to the input (𝐸 could be learned lookup table or fixed function)
- multihead attention
    - use H independent "attention heads" in parallel
    - run self-attention in parallel on each set of input vectors (different weights per head)

### The Transformer

- transformer block
    - input: set of vectors x
    - output: set of vectors y
    - self-attention is the only interaction between vectors
    - layer norm and MLP work independently per vector
    - highly scalable, highly parallelizable
- how to use it for vision?
    - plug attention into existing CNN
    - replace convolution with "local attention"
    - standard transformer on pixels 
        - problem: memory
    - standard transformer on patches - Vision Transformer (ViT)
        - computer vision model with no convolutions
        - HW friendly computations

:::success
**You Should Know**
- *What is attention*
Attention is a system of focunsing on the more important parts of the image. It is adaptive weightening of the features according to their importance to the input. Different types: channel, spatial, temporal, spatial-temporal, channel-spatial
- *Examples of channel attention*
SENet, squeeze and excitation. Channels (representing different features) are weighted, more important features get higher weight.
- *Spatial attention*
    - *Idea of RAM*
    Recurrent Attention Model, extracts multiresolution patches from the image, the patches are used for feature learning, features are used for prediction in RNN model.
    Example: image captioning with RNN
    - *Attention and self-attention layer*
    Attention layer: 
    ![attention](https://erdem.pl/static/ebfde1299deed8b423937d678e593158/33c9c/key-value-matrixes.png)
    Self-attention alyer: queries computed from input X based on query weight matrix.
    Disadvantage: very memory intensive.
    - *Transformers and their usage in vision*
    Transforemer consists of transformer blocks - layer normalization, self-attention, layer normalization, multilayer perceprton for each vector. Self-attention is the only interaction between vectors
    How to use attention/transformers for vision? Add attention layer to existing architecture, for example ResNet, replace convolution with "local attention", use transformer on pixels (memory!) or patches.
    - *Vision Transformer*
    Transformer used on image patches, it is a vision model without convolutions. Can solve same tasks as CNN, main advantage is that it is more hardware friendly.
:::

## DIFFUSION MODELS

- text-to-Image generation
- Midjourney, DALL-E, Stablediffusion
- problem: text, counting (e.g. seven apples)
- diffusion models are probabilistic models used for image generation
- they involve reversing the process of gradually degrading the data
    - forward process progressively adds noise, backward sequentially removes noise
- three different formulations
    - denoising diffusion probabilistic model (DDPM)
    - noise conditioned score networks (NCSN)
    - stochastic differential equations (SDE)

### Denoising diffusion probabilistic model (DDPM)

- forward process - the image is gradually replaced by noise
    - image is sampled from a distribution conditioned by the previous image
    - Markovian property assumed - the next image depends on the previous one only
        - we can explicitly write a formula for an image using the input image
- backward process - approximation of mean and standard deviation by the neural network
    - U-net like enural network with positional information
- training algortihm
    - repeat until convergence:
        - sample an image from our dataset
        - randomly choose time-step of the forward process
        - get noisy image
        - update network weights
    - the network is learning the parameters of the noise distribution
- inefficient, require multiple network evaluations

### Latent diffusion models (LDM)

- idea: use a diffusion model in the latent space of a powerful pre-trained autoencoder
    - diffusion model not applied in the pixel space
- cross-attention layers introduced for conditioning inputs

:::success
**You Should Know**
- *The principle of diffusion models*
Probabilistic models for image generation. Main idea of the training process is gradually degrading the data by adding Gaussian noise (modeled as a Markov chain) and than removing the noise (modeled as a Markov chain). The noisy image can be seen as a vector in a latent space.
Training procedure: sample image from data, randomly choose time step, sample noise, get noisy image, update network weghts.
![diff principle](https://lilianweng.github.io/posts/2021-07-11-diffusion-models/DDPM.pngt)
Applications: text to image, inpainting, image generation
:::

![diffusion](https://miro.medium.com/v2/resize:fit:1400/1*WTe5olMSFC-T6No0Y_gKWg.png)