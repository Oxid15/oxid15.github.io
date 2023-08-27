---
layout: post
title:  "ML Fingerprint Matching Methods"
date:   2022-10-16 00:00:00 +0000
categories: posts en
tags: fingerprints
---

Brief overview

## Foreword

During first year of my master's program I was assigned the task to make a literature review on the topic of interest. Back then I haven't decided on my master thesis topic, so it wasn't the option. The topic I specialized on previously was not quiet a challenge for me, I was somewhat tired of image smoothing topic (for the interested paper, code) so I decided to choose another topic that I was working on previously.
So this is pretty much how this text emerged. It is an overview of how ML-techniques are used in the field of fingerprint matching. For the moment of me writing that overview, I haven't found any similar works in the literature, which is in my opinion an overlook for the field. The work on this topic should be more broad than my toy-overview, but in my opinion the field of fingerprint-tasks lacks the works like that.

## Introduction

Fingerprints are one of the most reliable ways to identify a person. Patterns contained in the fingerprints are consistent and are not changing during the lifetime of an adult. In addition to that, they are extremely unique and can't simply be lost. Based on these advantages, fingerprints have wide area of application and were first introduced as a method of an identification of a person almost 100 years ago.

Firgerprint matching was automated almost 50 years ago and since that, the tools which are used for identification were in the process of improvement and change [7]. This process continues to this day and new methods emerge. In the last years machine learning (ML) techniques achieved the results better than the classical methods in the number of image processing tasks. This change affected fingerprint tasks accordingly and in the last years there are a number of attempts to apply new methods. However, due to the specific traits of the fingerprint-related tasks new algorithms occasionally have a hard time competing with the classical ones in the quality of the results and computational performance.

This work is aimed to give an overview of the application of machine learning techniques in the field of fingerprint verification and identification.

## Methodology

The purpose of this review is to highlight the work that was done by the researchers and engineers in the development of automated fingerprint matching techniques with the usage of machine learning.

Before we can proceed to the methodology section, the definition of fingerprint matching should be given as it is considered in this work. In this work the term fingerprint matching is used to unite the related tasks in the field - verification and identification. Verification is the task in which a pair of fingerprints is given as input and the decision whether the pair of fingerprints is from the same finger is expected as the output. In the task of identification a query fingerprint is given and the goal is to retrieve corresponding fingerprint from a large database or return nothing if there is no such fingerprint.

Description of methodology will consist of the following parts: search databases, keywords which were used to search related papers, include criteria, exclude criteria and several other selection principles. For the search of the related articles Scopus, Web Of Science and Google Scholar databases were used. This set of databases considering its size provided broad overview of the topic in the discussion. Keywords that were used are: fingerprint recognition, machine learning, neural network, fingerprint verification. These keywords were used in different combinations to find related articles.

Include criteria were the following: the paper should be related to the task of verification or identification as close as possible. The language of the papers should be English. As exclude criteria “the absence of machine learning approaches” was selected. The field of automated fingerprint processing is relatively narrow and the use of machine learning in it is not so common so the last 10 years (papers since 2011) were used as the limit for the year of publication.

In the next section results of the literature review will be presented. Different comparison criteria will be described and applied to the selected papers.

## Results

Fingerprint matching is a challenging task due to the fact that image acquisition process is not perfect and nonrigid nature of the transformations that occur in that process. These transformations are the main source of the complexity of the task since making model of such transformations is challenging. Since machine learning approaches are achieving significant results in other fields of computer vision, researchers in the field of fingerprint processing are trying to apply them to the corresponding tasks. The type of the task is the most fundamental comparison criterion which can be used.

For this review papers that were relevant to the tasks of verification and identification were selected. On the figure the distribution of the tasks, selected papers was meant to tackle. Regardless of the particular task, all of them were relevant to the tasks mentioned earlier. For example minutia extraction was the part of the verification system [1].

![](../../../../../images/fingerprint_tasks.png)
Papers by the task

Classification of fingerprints is the task of defining the global pattern class. Pattern is formed by the ridges and valleys. All of the fingerprint patterns can be classified into several categories. Two papers on the topic of classification were added to the review, because methods that were applied are general and can in similar fashion be used for other purposes [2, 3].

![](../../../../../images/fingerprint-patterns.jpg)

Simple illustration of different fingerprint patterns. Image from the paper.

Generic pipeline of any fingerprint processing consists of the following stages:

Image acquisition
Image preprocessing and enhancement
Feature extraction
Matching / Classification
Based on that, we can classify different approaches by the part, to which machine learning approach was applied. Some reviewed papers have their machine learning approaches in the feature extraction part, for example in the work of P. Marák and A. Hambalík [3] convolutional neural network (CNN) was used to refine the locations of the local fingerprint features - minutiae. In the number of papers several different classifiers are used in the last decision step - for matching or classification.
The paper of A. Balti et al. being not so recent is still a good example of that. It describes the process of classical preprocessing and feature extraction methods and the neural network-based classifier on the end of the pipeline that is trained to perform the task of verification given the vector of features [4]. Machine learning approach can be used even in between two stages of the generic pipeline described.
This was done in the paper [5], where authors used CNN (Google's Inception model) between image acquisition and image preprocessing stages to not allow the images that was acquired with poor quality to pass further down the pipeline, which increased the quality of verification system. Not only single stages can be replaced by machine learning algorithms. Since one of the strongest properties of the neural networks is feature extraction of different levels, many authors suppose that it is not necessary to manually engineer features for proper work of the fingerprint matchers. Based on that, several attempts were made to learn some fingerprint task in end to end fashion. This is not mean that the whole pipeline was replaced by the single network, but the last two stages at least.
The paper [6] can serve as an example of the kind of approach described. Pretrained on ImageNet dataset AlexNet model was used with shared weights to make a concatenated vector of a fingerprint pair to classify whether it is from the same finger. This approach is similar to siamese network, but it uses concatenated features from both fingerprints instead of computing some sort of contrastive loss as it is commony done with the similar architectures.To understand the distribution of where different authors position machine learning approaches the following plot is presented.

![](../../../../../images/fingerprint_position.png)

Papers by the position in the pipeline

It can be seen from the figure that the most popular solutions with machine learning include its methods in the classifier stage. It means that the authors manually produced features that are extracted from fingerprint image and used machine learning at the stage of the final solution. In the paper [8] the authors proposed exactly this kind of identification pipeline. Their work in the features they use is similar to the ones in [4], but they use the same features for another task. Authors use distance from the center point as a feature vector. The core of the method is simple neural network which is trained on the vector of sorted distances between the singularity and the minutiae. Authors propose it to use in the process of comparison of the fingerprint features to perform identification. The second popular solution is the attempt to incorporate as many stages as can be into the one neural network and train it end-to-end with the (preprocessed) images on the input and the desired decisions in the output. The authors of [9] choose this kind of solutions. They used end-to-end learning to extract features and use their network for verification and identification purposes. VGG-16 model was used as the framework. Obtained model was tested with verification task and as a feature extractor for the acceleration of identification process. Aside from the place of the machine learning model in the pipeline, there is also a way of comparing different approaches. This way is to look at the types of methods researchers use in their solutions. Since fingerprints are in their essence image data, it is expected to see convolutional neural networks in the solutions. The distribution of types of machine learning methods can be seen on the next figure.

![](../../../../../images/fingerprint_models.png)

Papers by the model used

The distribution presented on the figure shows the prevalence of CNN-based approaches, but as it was already said machine learning methods are commonly placed in the position of final classifier, so basic neural networks are also common. Another aspect that can be read from the chart is that non-neural algorithms are not so frequently used. This can be due to the scope of this review and the keywords used. Fingerprint matching approaches are commonly use support vector machines (SVM), but supposedly they were used before the emergence of the term “machine learning” itself. If SVM`s would be added to the keyword list, different results could be seen on the chart. The approach that is denoted with the type of method “Random forest” is implemented in the work [10]. The description on the chart was made for brevity - authors used several different tree-based methods on the classifier stage for the task of identification. Their results show that random forest achieves better results than REPTree, Random Tree, J48 and Decision Stump, when simple statistical measures are passed as input to identify the fingerprint. This method is not based on the extraction of the minutiae aside from singularity detection. When the singularity of the fingerprint is detected, 100x100 crop is made with it in the centre and in this area several statistical metrics are calculated. Metrics include variance, maximum probability, homogeneity, entropy, etc. These metrics are in some way reliable, since they are rotation invariant, which is important in the fingerprint matching since fingers can be rotated during the image acquisition stage. However, statistical metrics cannot be robust to the variations in the image itself. If the method of obtaining fingerprint image would change (e.g. the scanner changed from optical to capacital) statistics could change dramatically, while local minutia features could remain. Supposedly the most important part of the solutions is not the methods used, but the performance that is achieved with them. Performance in the tasks of identification of verification is measured with similar metrics, but the values could differ between the tasks. Direct comparison of the performance is also complicated by the fact that performance may be assessed using different databases and different metrics. Table 1 gives an overview of different databases that is used for performance assessment and the metrics themselves.


The approach |	Method description | Is method minutia-based | FVC2002 | FVC2004 | FVC 	| PolyU | CrossMatch Verifier 300 sensor | Custom data
B. O. Alijla et al. [1] | CNN for minutia extraction | Yes | Acc: 91.6 | - | - | - | - | -
S. Minaee et al. [2] | End2End CNN for classification | No | - | - | - | Acc: 95.7 | - | -
P. Marák et al. [3] | CNN as feature extractor | Yes | EER: 33 | - | - | - | EER: 8 | -
A. Balti et al. [4] | NN as the classifier | Yes | EER: 5.15 | - | - | - | - | -
V. M. Praseetha [5] | Pre-validation | Yes | - | - | - | - | - | FAR: 0.1
B. Bakhshi [6] | End2End CNN, in siamese manner for verification | No | EER: 17.5 | - | - | - | - | -
L. T. Nguyen [8] | NN as the classifier | Yes | - | - | Acc: 97.75 | - | - | -
Y. Liu [9] | End2End CNN verif. and identif. | No | EER: 2.91 | ERR: 3.72 | - | - | - | -
H. Jan [10] | Classifier of statist. descriptors | No | Prec: 59 | - | - | - | - | -
K. Wxh [11] | No | EER: 2.18 | - | - | - | - | - | -

The table gives several insights about the state of the field of ML-techniques in fingerprint matching tasks. First is about the usage of databases. The latest database now is the FVC onGoing, which is the evolution of the fingerprint verification competitions that took part earlier from 2000 to 2006 [12]. However, the most used database is the FVC2002. The explanation could be that this is due to the back-compatibility - so the previous works could be compared with the current, but the problem is that not so many authors actually do comparison of their results to other approaches to the same problem. The other result of the comparison that can be seen from the table is the difficulty of comparison, since the authors do not include ubiquitous quality metrics in theirs work such as EER. The metrics denoted in the table are Acc - accuracy, Prec - precision, FAR - false-acceptance rate and EER - equal error rate. The actual meaningful comparison that can be done is on the FVC2002 by the EER metric shows that the results of usage of machine learning methods vary in quality. If the relative values of EER are considered it can be seen that the methods, which use end to end learning (with no manual engineered features) achieve generally better results than the methods, where only one part of the matching process is replaced by the machine learning algorithm. Another characteristic of the current trend is that new approaches are no longer use minutiae as the features. This is also refers to the manual construction of features. The tendency is that minutiae-based approaches are replacing with the new automatically extracted features, which have no interpretation. The performance increase is understandable benefit, but the problem of interpretability is not currently solved, which is crucial for use new types of algorithms in law-enforcement and criminalistics.

## Conclusion

In this work the review of fingerprint matching methods with the usage of machine learning was made. The results show how machine learning algorithms are currently used, what is their performance and how they can be compared.

The most popular approach is to use machine learning method to classify existed manually-engineered features. However, the attempts to make an end to end architecture which could extract features and classify based on them are extensively made. The results of these attempts are comparable with other approaches, but the models may not be robust for rotations and heavily depend on the preprocessing stage, so the problem of making rotation-invariant and quality robust end to end model is still open and needs further investigation. The most popular approach in the model selection is the state-of-the-art CNN model pretrained on the general image classification dataset. This is a good approach for a general research and can achieve good results, but the real applications require computation on the edge devices, which in general cannot run such big models at least on acceptable speed, so the problem of computational performance and the usage of lightweight models is also still present.

Quality of the results can be directly compared only with the usage of the same databases and similar metrics. For the sake of fair comparison authors should provide the result of experiments on conventional databases and with the usage of common metrics.

## References

1. B. O. Alijla, M. Saad, and S. F. Issawi, “Neural Network-based Minutiae Extraction for Fingerprint Verification System,” 2017 8th Int. Conf.
Inf. Technol., pp. 435–441, 2017.
2. S. Minaee, E. Azimi, and A. Abdolrashidi, “FingerNet: Pushing The Limits of Fingerprint Recognition Using Convolutional Neural Network,”
2019.
3. P. Marák and A. Hambalík, “Fingerprint Recognition System Using Artificial Neural Network as Feature Extractor: Design and Performance
evaluation,” Tatra Mt. Math. Publ., vol. 67, no. 1, pp. 117–134, 2016.
4. A. Balti, M. Sayadi, and F. Fnaiech, “Fingerprint verification based on back propagation neural network,” Control Eng. Appl. Informatics, vol.
15, no. 3, pp. 53–60, 2013.
5. V. M. Praseetha, S. Bayezeed, and S. Vadivel, “Secure fingerprint authentication using deep learning and minutiae verification,” J. Intell.
Syst., vol. 29, no. 1, pp. 1379–1387, 2020.
6. B. Bakhshi and H. Veisi, “End to End Fingerprint Verification Based on Convolutional Neural Network,” ICEE 2019 - 27th Iran. Conf. Electr.
Eng., pp. 1994–1998, 2019.
7. D. Maltoni, D. Maio, A.K. Jain, “Handbook of Fingerprint Recognition,” Springer-Verlag, London, 494 p. 2009.
8. L. T. Nguyen, H. T. Nguyen, A. D. Afanasiev, and T. Van Nguyen, “Automatic Identification Fingerprint Based on Machine Learning
Method,” J. Oper. Res. Soc. China, 2021.
9. Y. Liu, B. Zhou, C. Han, T. Guo, and J. Qin, “A novel method based on deep learning for aligned fingerprints matching,” Appl. Intell., vol. 50,
no. 2, pp. 397–416, 2020.
10. H. Jan, A. Ali, and S. Mahmood, “Statistical Descriptors-based Automatic Fingerprint Identification: Machine Learning Approaches,” arXiv
Prepr., 2019.
11. K. Wxh and W. Likoei, “Small Area Fingerprint Verification Using Deep Convolutional Neural Network,” vol. 3, pp. 5–10, 2020.
12. FVC onGoing . Accessed: Sept. 24, 2021. Online.. Available: https://biolab.csr.unibo.it/FVCOnGoing/UI/Form/Home.aspx