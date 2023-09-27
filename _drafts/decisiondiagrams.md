---
layout: post
title: "Decision Diagrams: Overcoming Replication And Fragmentation in Decision Trees"
# title:  "Decision Diagrams: Generalizing Decision Trees to DAGs"
---

{% assign exampleExpression = "(A·B)+(C·D)" %}
{% assign carEvaluationExpression = "(Affordable and FourDoors) or (WellMaintained and HighSafety)" %}
{% assign heartRiskClassifExpression = "(Over60 and Smoker) or (HasFamilyHistory and Sedentary)" %}

_The good old decision tree{{link?}}._

This classical machine learning model is still quite relevant in both academia and industry, even while large language models{{link?}} and stable diffusion{{link?}} take the world by storm. Decision trees are reasonably simple to implement and debug, and tree-based models like Random Forest{{link}} and XGBoost{{link}} are known for their ease of use and good performance, especially when working with tabular data.[^tabular-data]

Decision trees are also considered highly interpretable[^interpretability] since the model's internal decisions can be inspected in the learned tree itself. This is a valuable feature for models applied in high-stakes decision-making and other settings where transparency is a concern.

The major shortcomings of decision trees include their high variance and tendency to overfit the training data. Pruning methods and ensemble algorithms have been extensively studied and applied to overcome these problems, to great effect. Several splitting criteria have been studied and compared{{footnote?}}. Multivariate splits{{link}}, model trees{{link}}, and other extensions have also been proposed. Even learning optimal decision trees, known to be NP-hard, has been successfully attained for reasonably-sized problems{{footnote}}.

So what else can be improved in the classical decision tree model?

## Replication and fragmentation

<!-- Introduce replication (interactive examples, simple & large) -->
<!-- Leads to fragmentation -->

Consider a decision tree for a binary classification problem using binary variables. The logic that follows can easily be extended for continuous variables, multiclass classification, and regression, but for the sake of simplicity, we will stick with this setting.

Let's suppose our features are binary variables `A`, `B`, `C`, and `D`, and that the real classification function we are trying to learn is simply `{{exampleExpression}}`. In a perfect world and with enough training data, we might learn <a href="#decision-tree-visualization" class="link-button" onclick="render(decisionTree, { expression: '{{exampleExpression}}' });">the following decision tree</a> which perfectly describes the classification rule. In this visualization, solid lines represent a variable assignment of `1` and dashed lines an assignment of `0`.

By the way, if dealing with raw binary variables feels too abstract, you could very well interpret this example function as something like <a href="#" class="link-button" onclick="return render(decisionTree, { expression: '{{carEvaluationExpression}}' });">evaluating if you should buy a particular car or not</a> or maybe <a href="#" class="link-button" onclick="return render(decisionTree, { expression: '{{heartRiskClassifExpression}}' });">classifying patients into high (1) or low (0) risk of a heart attack</a>.

_Tip: by clicking on <span class="link-button">links with this color</span> the related visualization is updated accordingly._

<iframe id="decision-tree-visualization" src="/assets/beard/dist/index.html" width="100%" height="500px"></iframe>
<p style="text-align: center;font-size: 0.75rem;">Tree visualization using <a href="https://github.com/pedrosbmartins/beard" target="_blank">beard</a>.</p>

Notice that the subtree that expresses the second term -- `C·D` -- had to be duplicated in order to represent the classification rule. In fact, for this expression and variable ordering, there will be one replicated subtree for each variable in the first term (you can verify this by <a href="#" class="link-button" onclick="return render(decisionTree, { expression: '(A·B·E)+(C·D)' });">adding new variables to the first term</a>). This is an example of a phenomenon known as the **replication problem** of decision trees. For a tree to properly express certain concepts, it must replicate part of its structure to account for all feature-splitting tests.[^replication-problem]

The replication problem leads to **data fragmentation**. As different subtrees expressing the same concept must be replicated across the tree structure, the statistical support given by training samples for each split decision has to decrease with depth necessarily, for a given finite data set. In other words, more data is needed to fully learn the underlying concept.

Proposed solutions to the replication and fragmentation problems in decision trees usually involve constructing new features. The idea is to capture the interaction between certain variables to avoid the kind of expression that leads to replication. These solutions have a few disadvantages, such as the requirement to first construct a full decision tree to discover the new features, and the possibility of affecting the interpretability of the final tree, since the constructed variables will represent interactions and not pure features.

In truth, the root of the replication problem lies in the tree structure itself. Can we tweak it to get rid of replicated subtrees?

## Decision diagrams

<!-- Decision diagrams -->
<!-- From ROBDDs to ML classification model -->

The defining property of directed rooted trees is that there is _exactly_ one path from the root node to any single leaf (you may use the [visualization above](#decision-tree-visualization) to convince yourself of this fact by trying different expressions). Take away this property, and you have now a general rooted _directed acyclic graph_ (DAG). That is, for a rooted DAG, there may be multiple paths from the root to a leaf.

Now, instead of replicating subtrees to express a concept such as `{{exampleExpression}}`, we can have a single subgraph representing the previously replicated concept, and connect different nodes of the graph to the root of this subgraph. Just like it's <a href="#decision-diagram-visualization" class="link-button" onclick="render(decisionDiagram, { expression: '{{exampleExpression}}', variant: 'diagram' });">demonstrated below</a> with the same expression of our running example. You can of course interpret it again as <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: '{{carEvaluationExpression}}' });">a car evaluation</a>, <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: '{{heartRiskClassifExpression}}' });">a heart attack risk classification</a>, or something else.

This type of graph is known as a _reduced ordered binary decision diagram_ (ROBDD).[^robdd] You may use the visualization selector to toggle between <a href="#" class="link-button" onclick="return render(decisionDiagram, { variant: 'tree' });">Tree</a> and <a href="#" class="link-button" onclick="return render(decisionDiagram, { variant: 'diagram' });">Diagram</a> visualizations. Notice how the diagram is a more compact representation of the same binary function since it does not replicate subgraphs.

In fact, ROBDDs have been extensively applied as an efficient data structure and set of algorithms for representing and manipulating binary expressions, with applications ranging from logic synthesis{{link}} and formal verification{{link}} to extending support vector machines for multiclass classification{{Platt2000}} and approximating neural network rules{{Chorowski2011}}.

<iframe id="decision-diagram-visualization" src="/assets/beard/dist/index.html" width="100%" height="500px"></iframe>
<p style="text-align: center;font-size: 0.75rem;">Decision diagram visualization using <a href="https://github.com/pedrosbmartins/beard" target="_blank">beard</a>.</p>

<!-- - more examples CDF, 3-input XOR, parity?, multiplexor?, ... -->

Our running example is a binary expression in _disjunctive normal form_ (DNF), but there are other formulas that illustrate the compactness of decision diagrams when compared to decision trees. For instance, we could visualize <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: '(A+B)·(C+D)' });">a similar expression in _conjunctive normal form_ (CNF)</a>, the <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: 'A·B + A·C + B·C' });">3-input</a> and <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: 'A·B·C+A·B·D+A·C·D+B·C·D+A·B·E+A·D·E+A·C·E+B·C·E+C·D·E+B·D·E' });">5-input majority function</a>, or the <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: 'A XOR B XOR C' });">3-input</a> and <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: 'A XOR B XOR C XOR D' });">4-input XOR function</a>.

### Diagrams as classifiers

Decision diagrams have not been widely adopted as a classification model, but several constructive approaches have been proposed. The inherent difficulty in learning diagrams arises from having to decide both splits and merges, and also from determining the best diagram topology. Therefore, most early algorithm proposals consisted of building a decision tree and then finding merges in a post-processing step, being computationally expensive for large problems, or of fixing the diagram structure a priori, which can be a considerable disadvantage.

Some proposals used the Minimum Message Length Principle (MMLP) to build diagrams, while others [se aproveitam de] _reduced ordered_ decision diagrams, which are diagrams that have no redundant nodes and where the tests performed on the variables follow a fixed order for all paths in the graph. This allows the use of efficient algorithms known from logical synthesis for manipulating this kind of graph.

In a method called _decision jungles_, ensembles of decision diagrams are proposed, where split and merging decisions are optimized jointly by minimizing the weighted entropy sum at the leaves, and the algorithm is implemented using local search heuristics. The authors reported improved generalization compared to random forests while using substantially less memory.

### Optimal decision diagrams

A recent trend in the literature has been the proposal of _optimal methods_ for the construction of decision diagrams for classification. This comes in step with the rise of optimal methods for decision trees, pushed by advances in hardware and in combinatorial optimization software \cite{Bixby2012AComputation}. The work of Bertsimas and Dunn \cite{Bertsimas2017} has been the root for continuous improvement of mathematical programming techniques for learning decision trees \cite{Carrizosa2021}, and other research paths include SAT \cite{Narodytska2018LearningSAT}, dynamic programming \cite{Demirovic2020MurTree:Search}, and branch-and-bound search \cite{Aglin2020LearningSearch}. These techniques are usually limited to small and medium-sized problems, but the substantial improvements in combinatorial optimization software have yielded compelling results in data sets of practical importance.

\citeonline{Cabodi2021OptimizingClassification} present a SAT-based model for computing a decision tree as the smallest reduced ordered binary decision diagram. \citeonline{Hu2022OptimizingClassification} proposed a similar approach, with the key differences of using a MaxSAT solver and limiting the resulting diagram depth. Simultaneously, another research stream in the optimal methods space is the Optimal Decision Diagram (ODD) algorithm \cite{Florio2022OptimalClassification}, of which I am a co-author.

- advantages? generally more sparse, maybe more interpretable

[^tabular-data]: Tabular data and DL discussion.{{complete}}
[^interpretability]: Do note that interpretability claims require some care... {{complete}}
[^replication-problem]: [Replication problem overview](https://link.springer.com/chapter/10.1007/BFb0017003#chapter-info) {{complete}}
[^robdd]: [ROBDD overview](https://web.archive.org/web/20110304135553/http://configit.com/fileadmin/Configit/Documents/bdd-eap.pdf) {{complete}}

<script type="text/javascript">
    function render(beardInstance, state) {
        beardInstance && beardInstance.render({ ...state })
        return false
    }

    const decisionTreeIframe = document.getElementById('decision-tree-visualization')
    let decisionTree

    const decisionDiagramIframe = document.getElementById('decision-diagram-visualization')
    let decisionDiagram

    window.addEventListener('message', event => {
        if (event.data.name !== 'graphvizloaded') return
        switch (event.data.frameId) {
            case decisionTreeIframe.id:
                decisionTree = decisionTreeIframe.contentWindow.beard
                render(decisionTree, { expression: '{{exampleExpression}}', variant: 'tree', options: { hideVariantSelector: true } })
                break
            case decisionDiagramIframe.id:
                decisionDiagram = decisionDiagramIframe.contentWindow.beard
                render(decisionDiagram, { expression: '{{exampleExpression}}', variant: 'diagram', options: { hideVariantSelector: false } })
                break
        }
      })
</script>
