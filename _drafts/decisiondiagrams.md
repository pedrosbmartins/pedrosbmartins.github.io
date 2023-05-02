---
layout: post
title:  "Decision Diagrams: Generalizing Decision Trees to DAGs"
---

<!-- Decision trees, extensively used in academics and industry. -->
<!-- The "solved" problems (overfitting -> pruning, high variance -> ensembles) -->

*The good old decision tree{{link?}}.*

This classical machine learning model is still quite relevant in both academia and industry, even while large language models{{link?}} and stable diffusion{{link?}} take the world by storm. Decision trees are reasonably simple to implement and debug, and tree-based models like Random Forest{{link}} and XGBoost{{link}} are known for their ease-of-use and good performance, especially when working with tabular data.[^tabular-data]

Decision trees are also considered highly interpretable[^interpretability] since the model's internal decisions can be inspected in the learned tree itself. This is a valuable feature for models applied in high-stakes decision making and other setting where explainability is a concern.

The major shortcomings of decision trees include their high variance and tendency to overfit the training data. Pruning methods and ensemble algorithms have been extensively studied and applied to overcome these problems, to great effect. Entropy, information gain, Gini index, and other splitting criteria have been proposed and studied{{footnote?}}. Even learning optimal decision trees, known to be NP-hard{{footnote}}, has been successfully attained for reasonably-sized problems. 

So what else can be improved in the classical decision tree model?

## Replication and fragmentation
<!-- Introduce replication (interactive examples, simple & large) -->
<!-- Leads to fragmentation -->

Consider a decision tree for a binary classification problem using binary variables. The logic that follows can easily be extended for continuous variables, multiclass classification, and regression, but for the sake of simplicity we will stick with this setting.

Let's suppose our features are binary variables `A`, `B`, `C`, and `D`, and that the real classification function we are trying to learn is simply `(A·B)+(C·D)`. In a perfect world and with enough training data, we might learn <a href="#decision-tree-example-1" class="link-button" onclick="render(decisionTreeExample1Beard, { expression: '(A·B)+(C·D)' });">the following decision tree</a> which perfectly describes the classification rule. In this visualization, *solid lines* represent a variable assignment of `1` and *dashed lines* an assignment of `0`.

By the way, if dealing with raw binary variables feels too abstract, you could very well interpret this example function as something like <a href="#" class="link-button" onclick="return render(decisionTreeExample1Beard, { expression: '(Affordable and FourDoors) or (WellMaintained and HighSafety)' });">evaluating if you should buy a particular car or not</a> or maybe <a href="#" class="link-button" onclick="return render(decisionTreeExample1Beard, { expression: '(Over60 and Smoker) or (HasFamilyHistory and Sedentary)' });">classifying patients into high (1) or low (0) risk of a heart attack</a>.

*Tip: by clicking on <span class="link-button">link buttons</span> the related visualization is updated accordingly.*

<iframe id="decision-tree-example-1" src="/assets/beard/dist/index.html" width="100%" height="500px"></iframe>
<p style="text-align: center;font-size: 0.75rem;">Decision tree visualization using <a href="https://github.com/pedrosbmartins/beard" target="_blank">beard</a>.</p>

Notice that the subtree that expresses the second term `C·D` had to be duplicated in order to represent the classification rule. In fact, for this expression and variable ordering there will be one such subtree for each variable in the first term (you can verify this by <a href="#" class="link-button" onclick="return render(decisionTreeExample1Beard, { expression: '(A·B·E)+(C·D)' });">adding new variables to the first term</a>). This is an example of a phenomenon known as the **replication problem** of decision trees. For a tree to properly express certain concepts, it must replicate part of its structure to account for all assignment tests.[^replication-problem]

The replication problem leads to **data fragmentation**. As different subtrees expressing the same concept must be replicated across the tree structure, the statistical support given by training samples for each split decision has to decrease necessarily with depth, for a given finite data set. In other words, more data is needed to fully learn the underlying concept.

Proposed solutions to the replication and fragmentation problems in decision trees usually involve constructing new features. The idea is to capture the interaction between certain variables so as to avoid the kind of expression that leads to replication. These solutions have a few disadvantages, such as the requirement to first construct a full decision tree in order to discover the new features, and the possibility of affecting the interpretability of the final tree, since the constructed variables will represent interactions and not pure features.

In truth, the root of the replication problem lies is in the tree structure itself. Can we tweak it to get rid of replicated subtrees?

## Decision diagrams
<!-- Decision diagrams -->
<!-- From ROBDDs to ML classification model -->

...

### Optimal decision diagrams
<!-- Optimal models (from seminal Bert&Dunn 2017) -->
<!-- Optimal BDDs -->
<!-- Optimal Decision Diagrams -->

...


[^tabular-data]: This is the first footnote.{{complete}}

[^interpretability]: Do note that interpretability claims require some care {{complete}}

[^replication-problem]: [Replication problem overview](https://link.springer.com/chapter/10.1007/BFb0017003#chapter-info) {{complete}}

<script type="text/javascript">
    function render(beardInstance, state) {
        beardInstance && beardInstance.render(state)
        return false
    }

    const decisionTreeExample1 = document.getElementById('decision-tree-example-1')
    let decisionTreeExample1Beard
    window.addEventListener('message', event => {
        if (event.data.name !== 'graphvizloaded') return
        switch (event.data.frameId) {
            case decisionTreeExample1.id:
                decisionTreeExample1Beard = decisionTreeExample1.contentWindow.beard
                render(decisionTreeExample1Beard, { expression: '(A·B)+(C·D)', variant: 'tree', options: { hideVariantSelector: true } })
                break
        }
      })
</script>