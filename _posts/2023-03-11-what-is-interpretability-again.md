---
layout: post
title:  "What is Interpretability Again?"
date:   2023-03-11 00:00:00 +0000
categories: posts en
tags: xai
---

Brief overview of definitions

## Foreword

During the work on my master-thesis that is devoted to building a benchmark for explanation solutions I should have read a lot of articles on the topic. And the idea of this overview came to me when reading another article's introductory part with definitions of the central term - interpretability. I noticed that there are different (and sometimes very different) definitions present in literature. The field is still developing, but it is not so new to have definition disagreements like we see present. This is not only me noticing this fact, because it is a part of the critique of the field for a considerable time now [1, 4].
Interpretability and Explainability are the two key terms in XAI field. They are used interchangeably in literature, however some authors distinguishing between them. This overview is aimed to give a picture of how different researches define the key term, what main undercurrents on this topic present, what is the most popular option and how distant are the outliers.
The overview also has a lot of interesting links, so for anyone who wants to dive into the topic the list of references may be useful.

## Interpretability

In interpretability-related literature there are a lot of statements about the essence of the field - the interpretability itself. Foundational concepts like these are usually hard to define. A multitude of answers for a question of some key concept in a common situation in every field of knowledge, however in other fields some agreements or mainstream definitions are established, which is not true for XAI field.The most common way to approach a question of definition is the following:

> interpretability is the ability to explain or to present in understandable terms to a human [1].

This is the definition given in the work of Doshi-Velez and Been Kim, who argue for uniformity and rigor in the science of interpretable machine learning. They also state that interpretability has different definitions in social and computer science respectively.

Similar approaches can be found in works of [9], [1] and [15]. The formulation authors provide is more or less the same, stating:
>interpretability is the ability to explain the model outcome in understandable ways to a human” in [9].

Barredo et al. adds 

>"or to provide the meaning" to "the ability to explain" [17].

[1] goes along the same line adding "to present" supposedly amplifying the role of presentation and the interface when communication an explanation to a user. Nauta et al. in their overview make no distinction between interpretability and explainability for inclusion reasons - they didn't want to exclude some paper from the overview because of the difference in terminology. They state: 

>an explanation is a presentation of (aspects of) the reasoning, functioning and/or behavior of a machine learning model in human-understandable terms [2].

This definition is more broad, but goes along the lines of "presenting something in human-understandable form". However, they specify the subject of presentation as reasoning, functioning or behavior or aspects of any of this, which other authors usually omit.

Another similar in words, but very different in meaning way to put the definition is the causation approach. Some argue that causation is not needed for an explanation whereas some disagree. For example the definition from the work of Tim Miller [8] which is claimed to stem from the work of Biran and Cotton [7] is formulated as:

> interpretability in machine learning is the degree to which an observer can understand the cause of a decision from an ML model.

The explainability in the work of Miller is equated with interpretability. Explanation is also regarded as an answer to a "why-question". A why question is defined as a complex entity which is composed of presupposition, whether-question and obviously a word "why". For example the question "why the model gave this answer" is composed of the word "why", question "did the model gave this answer?" and presupposition being "the model gave this answer". Miller also argues that the structure is not always that simple because explanations are also contrastive and the opposition of some decision that was made to the other that wasn't is very important.

The work of [19] takes the perspective of predictability or simulatability [4] - the ability of a human to predict model's behavior. They state:

> Interpretability refers to the intrinsic properties of a deep model measuring in which degree the inference result of the deep model is predictable or understandable to human beings [19].

The interpretability is by their definition an intrinsic property of a model which corresponds to the predictability of its behavior by a human. This goes in contrast to some other works because here interpretability becomes measure of what is called simulatability and considering this loses its own distinctive meaning.

Similar situation is in the work of Gilpin et al. [11]. Despite them distinguishing between interpretability and explainability, they argue that interpretability goes in contrast with another property being completeness. Completeness in their terms measures how accurate the explanation describes the functioning of the system. However, this definition of completeness is contrasted with a common one, used by authors of some other works [3, 10]. 
Common definition of completeness revolves around showing the user as much relevant information as possible, trying not to omit details of little influence in pursuit for a pleasant explanation. "Completeness" Gilpin et al. describe is best suited for another property called correctness or soundness in some works [3]. This property states for "nothing but the truth" and measures exactly what is described in their work. Here interpretability once again loses its own meaning and the position of the ground-term, becoming something else. Considering contrast authors make between interpretability and completeness one can suppose that under interpretability here they mean such property as completeness and by contrasting completeness they mean correctness. This tradeoff is common and well studied, however using other terms [2, 15]. Considering this, one can see the reasons behind the critique of the field for a lack of proper definitions and rigor.

Some more definitions include one made by Doran et al. [18]. They argue that system is interpretable only if it is transparent or white-box. For other class of methods, where an explanation is given for a black-box model they employ the term "comprehensible". This framework runs counter common definitions. Where other variants of definitions disagree only in terms, this changes the subject, what is called interpretable. In specific words they define interpretable system as "a system where a user cannot only see but also study and understand how inputs are mathematically mapped to outputs", which corresponds to the definition of white-box models.Murdoch et al. take the perspective of data science providing following definition of interpretable machine learning: "... as the extraction of relevant knowledge from a machine-learning model concerning relationships either contained in data or learned by the model" [10]. By the relevant knowledge they mean one that provides some new and useful information to the user.

## Interpretability vs Explainability

Some authors in their work separate terms "interpretability" and "explainability" giving them different meanings. In previous section points of authors that do not distinct between the two were overviewed. In this section the definitions of their counterparts will be reviewed. This section will also include definitions of explainability itself.

Rudin et al. in their work on customized rules for example state that explainability can be applied only to the black-box models and interpretability can be applied only to transparent or white-box models [5]. Burkart and Huber in their survey go along the same line contrasting two terms in the following way:

> interpretability is about understanding the work of model as a whole and explainability is used when the model cannot be fully understandable and explanations are used to understand it [14].

Gaur et al. gives different definitions contrasting the terms by the type of question that is answered. Explainability answers the question of why the decision was made and interpretability answers the question of how the model made certain decision [6].

## Other related terms

XAI research field has also a number of terms that are close to the central term "interpretability" or "explainability". This section demonstrates some of these terms along with the meanings authors give them.Montavon et al. propose the term "understandability" that means the ability of a model to explain its decisions without revelation of its internal workings [12]. The work itself focuses on explanation methods for neural networks. This is useful in this context because in order to be understanded neural network models should be able to explain themselves without revealing their internal state since even having the information on every weight in the network it is difficult to grasp the reasons behind each decision.

Multiple works use the term "comprehensibility" similarly to the "interpretability" [15, 16, 17]. For example in the work of Gleicher comprehensibility refers to the ability of a user to understand relevant aspects of the model's decision [16]. However, Gleicher argues that authors using the term "interpretability" narrow the picture focusing only on the model and not on the stakeholders around modelling process.
Justification is another term used in the work of Biran et al. It means the explanation why the decision is good, but does not necessarily reveal any information on the process of making the decision [7].

Authors of many works focusing around the degrees of interpretability or types of it. For example Lipton in the notable work on the interpretability gives the term "transparency" the following meaning: if the models is understandable then it is transparent. Different models can show different degrees of transparency. Lipton divides models into three categories being: simulatable models, decomposable models and algorithmically transparent models [13].

The work of Doran et al. focuses on definitions and meanings and proposes another categorization of models by their interpretability. Opaque systems are what is usually referred to as black-boxes. Interpretable systems allow users to understand the mathematical mapping from inputs to outputs and comprehensible systems emit not only the outputs, but special symbols that allow the user to relate inputs to outputs [18]. Authors here use comprehensibility more or less the same way as Gleicher which is close to the interpretability.

## Conclusion

Making a summary of interpretability definitions some general picture of this research field forms. There are two main and considerable lines of thought: interpretability defined through causation and without it. The most common way to put interpretability exists with some deviations, but it was not to this moment formed into general consensus. Aside of that main lines a number of other, more exotic frameworks exist, which usually give interpretability very different meanings.
It seems necessary to state what definition will be used in my thesis. To not further increase the amount of different interpretations of interpretability definitions is would be reasonable to stick with some already existing definition that is also supported by a number of notable researchers in the field. In my work the following variant will be used:

> "interpretability is the ability to explain or to present in understandable terms to a human".

It is general so it is able to cover more approaches and it does not conflict with any of the properties of interpretability that are used to evaluate XAI methods. The distinction between terms "interpretability" and "explainability" will not be made in the work also.

## References

1. Doshi-Velez, Finale, and Been Kim. "Towards a rigorous science of interpretable machine learning." arXiv preprint arXiv:1702.08608 (2017).
2. Nauta, Meike, et al. "From anecdotal evidence to quantitative evaluation methods: A systematic review on evaluating explainable ai." arXiv preprint arXiv:2201.08164 (2022).
3. Kulesza, Todd, et al. "Too much, too little, or just right? Ways explanations impact end users' mental models." 2013 IEEE Symposium on visual languages and human centric computing. IEEE, 2013.
4. Hase, Peter, and Mohit Bansal. "Evaluating explainable AI: Which algorithmic explanations help users predict model behavior?." arXiv preprint arXiv:2005.01831 (2020).
5. Rudin, Cynthia, and Şeyda Ertekin. "Learning customized and optimized lists of rules with mathematical programming." Mathematical Programming Computation 10 (2018): 659-702.
6. Gaur, Manas, Keyur Faldu, and Amit Sheth. "Semantics of the black-box: Can knowledge graphs help make deep learning systems more interpretable and explainable?." IEEE Internet Computing 25.1 (2021): 51-59.
7. Biran, Or, and Courtenay Cotton. "Explanation and justification in machine learning: A survey." IJCAI-17 workshop on explainable AI (XAI). Vol. 8. No. 1. 2017.
8. Miller, Tim. "Explanation in artificial intelligence: Insights from the social sciences." Artificial intelligence 267 (2019): 1-38.
9. Doshi-Velez, Finale, and Been Kim. "A roadmap for a rigorous science of interpretability." arXiv preprint arXiv:1702.08608 2.1 (2017).
10. Murdoch, W. James, et al. "Definitions, methods, and applications in interpretable machine learning." Proceedings of the National Academy of Sciences 116.44 (2019): 22071-22080.
11. Gilpin, Leilani H., et al. "Explaining explanations: An overview of interpretability of machine learning." 2018 IEEE 5th International Conference on data science and advanced analytics (DSAA). IEEE, 2018.
12. Montavon, Grégoire, Wojciech Samek, and Klaus-Robert Müller. "Methods for interpreting and understanding deep neural networks." Digital signal processing 73 (2018): 1-15.
13. Lipton, Zachary C. "The mythos of model interpretability: In machine learning, the concept of interpretability is both important and slippery." Queue 16.3 (2018): 31-57.
14. Burkart, Nadia, and Marco F. Huber. "A survey on the explainability of supervised machine learning." Journal of Artificial Intelligence Research 70 (2021): 245-317.
15. Fernandez, Alberto, et al. "Evolutionary fuzzy systems for explainable artificial intelligence: Why, when, what for, and where to?." IEEE Computational intelligence magazine 14.1 (2019): 69-81.
16. Gleicher, Michael. "A framework for considering comprehensibility in modeling." Big data 4.2 (2016): 75-88.
17. Arrieta, Alejandro Barredo, et al. "Explainable Artificial Intelligence (XAI): Concepts, taxonomies, opportunities and challenges toward responsible AI." Information fusion 58 (2020): 82-115.
18. Doran, Derek, Sarah Schulz, and Tarek R. Besold. "What does explainable AI really mean? A new conceptualization of perspectives." arXiv preprint arXiv:1710.00794 (2017).
19. Li, X., et al. "Interpretable deep learning: Interpretation, interpretability, trustworthiness, and beyond. arXiv 2021." arXiv preprint arXiv:2103.10689.
