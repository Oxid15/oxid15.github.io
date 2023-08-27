---
layout: post
title:  "Review of Image Smoothing Algorithms"
date:   2020-09-22 00:00:00 +0000
categories: posts en
tags: image_smoothing
---

What is the task of image smoothing, when is it used and what approaches to it exist in image processing practice.

Image smoothing task is considered to be fundamental image processing task, but it's not exclusively "image" task. There exists for example dataset smoothing in statistics, which generalizes the concept of smoothing images to smoothing datasets. However, the task is similar – to capture general pattern in data. If we would try to formulate the task of image smoothing more precisely, we would say that we are trying to construct the new image in such way that we preserve general pattern and leave the noise and small perturbations out. How do we approach this task if we talk about digital image smoothing?

![](../../../../../images/bilateral-filter.png)

## Why do we need image smoothing?

Or it can be said the other way: what are the applications of smoothing algorithms? That's an interesting question, that is usually putted away by authors of articles. I would put it this way: edge-preserving smoothing gives the image some particular properties. What types of tasks these properties could be useful for? Algorithms can provide image abstraction (only significant objects remain) and smooth some noise, outliers and artifacts.
So I've tried to make a list of image smoothing applications. It is sure incomplete, but this is my view of problem. If you have some additions or simply feedback, please let me know by email.

## Image enhancement and visual effects

Smoothing is extensively used for image enhancement task and for creating different visual effects. The tasks themselves can vary from smoothing noise and compression artifacts to edge enhancement and detail magnification. Removing noise sounds natural for image smoothing algorithms, but they are usually not specialized in those types of tasks. This is actually could be the case, when algorithms for image smoothing wouldn't do so well on noise as it expected to, or as good as specialized noise removal algorithms. Noise removal algorithms have usually analogous principles, but they have different goals. The goal is to remove noise and preserve image as it was before the noise was added with as much detail as possible. We have a lot of specialized approaches for this type of tasks, but still can address them in some way. However, if we talk about compression artifacts, some works show [4] that smoothing algorithms can be successfully used for removing clipart compression artifacts.

![](../../../../../images/clipart.png)

We can also attribute image dehazing to this category. The task of dehazing is in removing fog from photos to achieve more pleasant view. Here is the example of applying dehazing algorithm that is based on bilateral filter.

![](../../../../../images/dehazing.png)

When we are smoothing the image, we obtain another one where details were removed. Hence, if we know what part of the image belongs to small details, we know what we should enchance. An ideal smoothing algorithm for the task of details magnification should neither blur nor over-sharpen the salient image structures, as either operation can lead to “ringing” artifacts in the residual image, resulting in halo or gradient reversals in the detail-enhanced images [5].

![](../../../../../images/enhancement.png)

The example of detail magnification done with various algorithms. Comparison from [5]

## Image segmentation

Image segmentation is another example of the field where properties of smoothed images could be useful. In work [6] authors show the synthesis of edge-preserving smoothing and image segmentation algorithm. They use properties of the smoothed image to segment it more effectively. Their approach uses contours, but that kind of approaches are sensitive to inhomohenous structures on the image and tend to perform poorly. To address this drawback, they used total variation based smoothing approach.
Here we have another example of using algorithms to simplify image before trying to find segments on it [7]. Fernand Meyer uses special algorithm that is based on filters also called levelings. This is specialized image smoothing approach (as also in [6]) but the idea of usage is the same.

## Edges extraction

This field should be mentioned in section above because all principles that can be applied to segmentation, can be also applied to the task of edges extraction. Let's see the effect of smoothing with the following example from my recent article devoted to image smoothing algorithm that is based on gradient analysis [8].

![](../../../../../images/edge_detection.png)

Images and corresponding edges before and after applying our smoothing algorithm. (Edges were extracted with Canny edge detector)

Different approaches
Approaches to image smoothing task can be divided into two distinct categories: explicit and implicit ones. I encountered this quite common and natural way of categorization first in the article [2]. It isn’t universal and widespread in the form in which I would use it, but in different reviews different authors use similar categorizations with minor deviations.So what are the explicit and implicit methods in image smoothing? Explicit methods are based around some explicit rule of calculating pixel values of new image. They are usually non-iterative and are formulated usually as weighed sum of values in pixel’s neighborhood.

![](../../../../../images/ql_5681bb3bc87e482edac72b16c63abfb8_l3.png)

Where p is certain pixel of output image S and input image I, q is a pixel of neighborhood N of pixel p and w(p,q) is weight of the specific filter. Weights of filter should satisfy some additional conditions. Their sum should be equal to one and in general they should be positive.Implicit methods are based on some optimization problem. I would consider algorithms that are based on deep learning as implicit too. However the majority of algorithms are usually formulated as a problem of optimization of function that looks like this:

![](../../../../../images/ql_9dfe12a63da59344671d9fe580617127_l3.png)

Where Ed is data term, which measures the difference between images I and S, Eν is smoothing term, which is responsible for making color variability smaller and λ is the coefficient for balancing this two terms.So we have different types of algorithms. Which one is better? It depends on your goals, when you are trying to smooth the image. Explicit algorithms are usually faster, or have fast implementations [3], but implicit ones can usually provide better smoothing. However, the choice between types must be based on the goal of smoothing.

Here are some examples of images smoothed by both types of algorithms:

![](../../../../../images/bilateral-filter2.png)
Results of bilateral filter [1]. It belongs to explicit category of methods.

![](../../../../../images/L0minimization.png)

Results from [4], where L0 gradient minimization is used. It is definitely implicit approach in our classification.

## Explicit approaches

In this section I'm going to describe approaches that are explicit. I'm going to show fundamental algorithms and some new approaches from recent works.

### Bilateral filter [1]

We definitely should start from this filter if we talk about image smoothing in general and explicit algorithms in specific. It is fundamental and serves as a baseline for other approaches. Results of this algorithm are present in almost every new image smoothing article.
The idea behind it is simple and works just fine with almost all of the cases. Because we're dealing with explicit method, to define it we need to define the rule, by which its weights are computed. We can verbally formulate the rule as follows: the closest and the most similar pixels in the neighborhood have the biggest weights and vice versa.
Mathematically it would look like:

![](../../../../../images/ql_5a80066c36f9d0923c2724be52b38f19_l3.png)

Where W_BLFpq - is the weight of pixel q in the neighborhood of pixel p, ||p−q||^2 - is referred to as squared euclidean distance from pixel p to q and Ip (or Iq) is the intensity value of p (q). We can see that the functions of distance and intensity are the Gaussians with parameters σs and σr. It means that bilateral filter is based on Gaussian filter, but its weights are adaptive and take in consideration the difference in intensity values and the distance between central pixel and given pixel from neighborhood. These properties allow bilateral filter to be the baseline approach in the field of smoothing from its birth in 1998.
We can't start talking about bilateral filter while avoiding the topic of its computational complexity. Naive implementation of it gives unsatisfactory results - we have very computationally complex problem, but in the case of bilateral filter, we have a lot of improvements and approximations of it, which attest logarithmic [10] or even constant [11] complexity.

### Guided filter [2]

After some time, in 2013 image processing world met new smoothing algorithm called guided filter. Its idea is in using help of the guidance image to calculate weights. The rule of calculating the weights is given below. It was fomulated under the assumption of local linear model between the guidance and filtering output.

![](../../../../../images/ql_5d78ab99d6d99c5a9375aa012ea25deb_l3.png)

After some time, in 2013 image processing world met new smoothing algorithm called guided filter. Its idea is in using help of the guidance image to calculate weights. The rule of calculating the weights is given below. It was fomulated under the assumption of local linear model between the guidance and filtering output.

![](../../../../../images/ql_5d78ab99d6d99c5a9375aa012ea25deb_l3-1.png)

Where μk is mean intensity in neighborhood ωk of pixel p. ϵ is the parameter that allows to regulate the level of smoothing. The effect of ϵ in the guided filter is similar with the range variance σr2 in the bilateral filter. Both parameters determine what is an edge that should be preserved [2].
The formula of guided filter is based on the solving of optimization problem. What's interesting, that results of guided filter is pretty close to the results of guided filter. Similarity was measured by the PSNR metric [12].
PSNR is the metric that usually used for measuring similarity between two images in the task of noise removing. It measures how in process of noise removing structure of the image wasn't damaged, but here it was used in unusual way. Resulting images of bilateral and guided filter under similar parameters were compared. PSNR showed the values between 37 and 40 db. Images are considered to be almost not recognizable, when PSNR is greater than 40db, since the difference is often visually insensitive. Let's look at the results by ourselves and try to tell the difference.

![](../../../../../images/comparison.png)

Filter based on gradient analysis [8]
This approach on the contrary is from the recent paper, from 2020. It uses information about boundaries that are contained in image gradient field. In particular, this approach discriminates between two kinds of boundaries - regular and irregular ones. It is trying to preserve the first kind of boundaries, assuming that the second one represents a texture or a noise. The difference between regular and irregular boundaries and their gradient fields can be spotted on the following picture.

![](../../../../../images/boundaries.png)

Here is regular boundary on the left. It has low deviations of gradient angles, unlike the other boundary type. It has high deviations. However, how do filter measures deviations of gradient angles in order to dicern between these types? Let's look at filter's formula to see it.

![](../../../../../images/ql_f2bc237210a065357955bd4bc7073878_l31.png)

Deviations measured as cosine of the doubled difference between angle of center pixel νp and angle of pixel in its neighborhood νq. When we measure angles of gradients we obtain values between −π/2 and π/2. Doubled differences between angles of two gradients are distributed between −2π and 2π. We need to double differences to treat angles with difference near π like similar ones, because they appear to be the parts of one boundary, as we saw on the picture with boundary types.
The weights are also divided by the value α that was left unmentioned. α is the gradient magnitude or simply - length. Bigger the gradient length, bigger the intesity change, smaller the weight would be. Let's finally look at the results.

![](../../../../../images/ara.bmp53.png)

## Implicit approaches
This category of methods is mainly based on building an optimization problem to achieve image smoothing. However, these days there is emerging interest in posing it as a machine learning problem. In section about types of approaches we already saw how this problem usually formulated.

![](../../../../../images/ql_9dfe12a63da59344671d9fe580617127_l31.png)

This is a quite popular formulation, but it can be modified by adding new components, etc. There's emerging interest in this field recently along with the same process in using neural networks to address smoothing problem.

### L0 gradient minimization [4]

The main achievement of L0 gradient minimization approach is a method of confining the number of nonzero gradients and smoothing in global manner. The function to optimize is similar to that one that was described before, but smoothing term is the number of the gradients which amplitude is not equal to zero.

![](../../../../../images/ql_05c9365f5595dc893538ca7f5eeedef7_l3.png)

#{∙} is the operator of counting objects for which the condition in parentheses is true and p is the index of pixel in neighborhood.
The problem of optimizing that function is non-trivial. It contains two terms that describe respectively local and global image features. Authors found out that gradient descent or similar traditional algorithms are not suitable. Due to these difficulties authors created their own solver.

![](../../../../../images/L0comparison.png)

Here are the results in comparison with other methods on synthetic data. You could see other results on the natural image above.

You can see how this method generates strong edges and nearly constant inner areas of objects. It would made the task of segmentation much easier.

### Deep filtering network [14]
The last approach in this short review is Deep Filtering Network. The method is based on encoding the distinction between textures and structures in the deep neural network. Serious impediment with setting this problem as a machine learning one is a difficulty of dataset generation. Manual annotation of existing images is complex and time consuming, so authors generated their own dataset of images combining pure texture images with pure structure images. They have found out that cartoon images appear to be the ones that have only structures, so they have been used in generation process.
First, natural textures were extracted from images with only textures. Then, spatial deformation was applied to them and color was varied. Obtained textures were blended into cartoon images.
Architecture of the network is builded as follows: image structure is obtained by Structure Prediction Network (SPN), similarly with texture and Texture Prediction Network (TPN). Their output becomes an input for Texture and Structure Aware Filtering Network (TSAFN). All networks are mainly based on convolutional layers. Proposed method outperforms non-learning ones in visual quality of results when compared on generated dataset.
But this is very specific approach by itself. It distinguishes between textures and structures by means of high-level information that was obtained as the output of neural network. It means that its results could look unusual in a way that we don't expect from smoothing algorithm. It could remove textures even if it forms strong color changes. I can illustrate this with th following picture.

![](../../../../../images/deepfiltering.png)

Here we can see the results of other filters and the result of deep filtering network (the last image). Here's an example of what I posit before: network "extracted" textures from images in unusual way. It is interesting behavior and there hopefully would be more research of it.

## Conclusion

In this post, we described the task of image smoothing. We found out that it is fundamental low-level image processing operation, reviewed possible applications of image smoothing and several interesting methods that could be used in image processing practice.
Here we used the classification of methods that divides them into two categories: explicit and implicit. Explicit methods are easy to understand and compute, but they usually don't achieve the same level of smoothing as implicit methods that are in turn require more computations and more difficult to implement.
However, the research in this field wasn't stopped and image smoothing methods are still evolving and improving. And so we are looking forward for new algorithms in this field!

## References

1. C. Tomasi and R. Manduchi, Bilateral filtering for gray and color images, Proc. IEEE Int Conf. Comput. Vis., pp. 839–846, 1998.
2. X. Fang, M. Wang, A. Shamir and S. M. Hu, Learning Explicit Smoothing Kernels for Joint Image Filtering, in Comput. Graph. Forum, 2019, pp. 181–190, doi: 10.1111/cgf.13827
3. S. Paris and F. Durand, “Computer Science and Artificial Intelligence Laboratory Technical Report A Fast Approximation of the Bilateral Filter using a Signal Processing Approach A Fast Approximation of the Bilateral Filter“, Massachusetts Inst. of Technology Comput. Sci. and Artif. Intell. Lab., Cambridge, Massachusets, USA, Rep. MIT-CSAIL-TR-2006-073, Nov. 9, 2006.
4. L. Xu, C. Lu, Y. Xu, and J. Jia, Image smoothing via L0 gradient minimization, ACM Trans. Graph., vol. 30, no. 6, pp. 1–12, 2011.
5. Fan, Qingnan. Image Smoothing via Unsupervised Learning. vol. 37, no. 6, 2018, pp. 14.
6. K. He, D. Wang, and X. Zheng, Image segmentation on adaptive edge-preserving smoothing, J. Electron. Imaging, vol. 25, no. 5, pp. 053022, 2016, doi: 10.1117/1.jei.25.5.053022
7. F. Meyer, “Levelings, Image Simplification Filters for Segmentation,” J. Math. Imaging Vis., vol. 20, no. 1–2, pp. 59–72, 2004.
8. V. Gudkov and I. Moiseev, “Image Smoothing Algorithm Based on Gradient Analysis” 2020 Ural Symposium on Biomedical Engineering, Radioelectronics and Information Technology (USBEREIT), pp. 403–406, 2020, doi: 10.1109/usbereit48449.2020.9117646
9. A. Tripathi and S. Mukhopadhyay, "Single image fog removal using bilateral filter" In Proceedings of IEEE International Conference on Signal Processing, Computing and Control (ISPCC 2012), pp. 1-6, 2012.
10. S. Paris and F. Durand, “A Fast Approximation of the Bilateral Filter using a Signal Processing Approach”, MIT-CSAIL-TR-2006-073, 2006.
11. K. N. Chaudhury, D. Sage, and M. Unser, “Fast O(1) bilateral filtering using trigonometric range kernels,” IEEE Trans. Image Process., vol. 20, no. 12, pp. 3376–3382, 2011.
12. K. He, J. Sun, and X. Tang, “Guided image filtering,” IEEE Trans. Pattern Anal. Mach. Intell., vol. 35, no. 6, pp. 1397–1409, 2013.
13. Malik, K. and Smolka, B., “Improved Bilateral Filtering Scheme For Noise Removal in Color Images”, In Proceedings of the International Conference on Informatics and Applications (ICIA 2012), Page(s): 118-130, 2012.
14. K. Lu, S. You, and N. Barnes, “Deep Texture and Structure Aware Filtering Network for Image Smoothing“, Lect. Notes Comput. Sci. (including Subser. Lect. Notes Artif. Intell. Lect. Notes Bioinformatics). [Online]. Avaliable: https://link.springer.com/conference/eccv
