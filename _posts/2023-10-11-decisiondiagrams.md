---
layout: post
title: "Decision Diagrams: Overcoming Replication And Fragmentation in Decision Trees"
---

{% assign exampleExpression = "(A·B)+(C·D)" %}
{% assign carEvaluationExpression = "(Affordable and FourDoors) or (WellMaintained and HighSafety)" %}
{% assign heartRiskClassifExpression = "(Over60 and Smoker) or (HasFamilyHistory and Sedentary)" %}

Decision trees are still quite relevant in both academia and industry, even while deep neural networks take the world by storm with the advent of transformers, large language models, and stable diffusion. Tree-based models like [Random Forest](https://en.wikipedia.org/wiki/Random_forest) and [XGBoost](https://en.wikipedia.org/wiki/XGBoost) are reasonably simple to implement and are known for their ease of use and good performance, especially when working with tabular data.[^tabular-data]

Decision trees are also considered highly interpretable, since the model's internal decisions can be inspected in the learned tree itself[^interpretability]. This is a valuable feature for models applied in high-stakes decision-making and other settings where transparency and intelligibility are a concern.

While the tree structure is greatly responsible for the model's simplicity, it also can act as a constraint that leads to practical issues. A fundamental aspect of (binary) decision trees is that the number of internal nodes grows exponentially with depth. Additionally, certain concepts can only be expressed as a tree by replicating specific subtrees, which also results in a large number of nodes and a fragmentation of training samples used to learn splitting decisions. These issues may negatively impact the interpretability, robustness, and the memory requirements of a learned tree.

This short post aims to discuss the **replication and fragmentation problems** of decision trees in the context of machine learning and classification, present **decision diagrams** as a possible solution, and point to **recent advances** in algorithms and applications of decision diagrams as classifiers.

## Replication and fragmentation

Consider a decision tree for a binary classification problem using binary variables. The logic that follows can easily be extended for continuous variables, multiclass classification, and regression, but for the sake of simplicity we will stick with this setting.

Let's suppose our features are binary variables `A`, `B`, `C`, and `D`, and that the real classification function we are trying to learn is simply `{{exampleExpression}}`. In a perfect world and with enough training data, we might learn <a href="#decision-tree-visualization" class="link-button" onclick="render(decisionTree, { expression: '{{exampleExpression}}' });">the following decision tree</a> which perfectly describes the classification rule. In this visualization, solid lines represent a variable assignment of `1` and dashed lines an assignment of `0`.

By the way, if dealing with raw binary variables feels too abstract, you could very well interpret this example function as something like <a href="#" class="link-button" onclick="return render(decisionTree, { expression: '{{carEvaluationExpression}}' });">evaluating if you should buy a particular car or not</a> or maybe <a href="#" class="link-button" onclick="return render(decisionTree, { expression: '{{heartRiskClassifExpression}}' });">classifying patients into high (1) or low (0) risk of a heart attack</a>.

_Tip: by clicking on <span class="link-button">links with this color</span>, the related visualization is updated accordingly._

<iframe id="decision-tree-visualization" src="/assets/beard/dist/index.html" width="100%" height="500px"></iframe>
<p style="text-align: center;font-size: 0.75rem;">Tree visualization using <a href="https://github.com/pedrosbmartins/beard" target="_blank">beard</a>.</p>

Notice that the subtree that expresses the second term -- `C·D` -- had to be duplicated in order to represent the classification rule. In fact, for this expression and variable ordering, there will be one replicated subtree for each variable in the first term (you can verify this by <a href="#" class="link-button" onclick="return incrementVariable();">adding new variables to the first term</a>). This is an example of a phenomenon known as the **replication problem** of decision trees. For a tree to properly express certain concepts, it must replicate part of its structure to account for all feature-splitting tests.

The replication problem leads to **data fragmentation**. As different subtrees expressing the same concept must be replicated across the tree structure, the statistical support given by training samples for each split decision has to decrease with depth necessarily, for a given finite data set. In other words, more data is needed to fully learn the underlying concept.

Proposed solutions to the replication and fragmentation problems in decision trees usually involve constructing new features[^replication-problem]. The idea is to capture the interaction between certain variables to avoid the kind of expression that leads to replication. These solutions have a few disadvantages, such as the requirement to first construct a full decision tree to discover the new features, and the possibility of affecting the interpretability of the final tree, since the constructed variables will represent interactions instead of pure features.

In truth, the root of the replication problem lies in the tree structure itself. Can we tweak it to get rid of replicated subtrees?

## Decision diagrams

The defining property of directed rooted trees is that there is _exactly_ one path from the root node to any single leaf (you may use the [visualization above](#decision-tree-visualization) to convince yourself of this fact by trying different expressions). Take away this property, and you have now a general rooted _directed acyclic graph_ (DAG). That is, for a rooted DAG, there may be multiple paths from the root to a leaf.

Now, instead of replicating subtrees to express a concept such as `{{exampleExpression}}`, we can have a single subgraph representing the previously replicated concept, and connect different nodes of the graph to the root of this subgraph. Just like it's <a href="#decision-diagram-visualization" class="link-button" onclick="render(decisionDiagram, { expression: '{{exampleExpression}}', variant: 'diagram' });">demonstrated below</a> with the same expression of our running example. You can of course interpret it again as <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: '{{carEvaluationExpression}}' });">a car evaluation</a>, <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: '{{heartRiskClassifExpression}}' });">a heart attack risk classification</a>, or something else.

This type of graph is known as a _reduced ordered binary decision diagram_ ([ROBDD](https://en.wikipedia.org/wiki/Binary_decision_diagram)).[^robdd] Notice how the diagram is a more compact representation of the same binary function, since it does not replicate subgraphs. You may use the visualization selector to toggle between <a href="#" class="link-button" onclick="return render(decisionDiagram, { variant: 'tree' });">Tree</a> and <a href="#" class="link-button" onclick="return render(decisionDiagram, { variant: 'diagram' });">Diagram</a> visualizations.

In fact, ROBDDs have been extensively applied as an efficient data structure and set of algorithms for representing and manipulating binary expressions, with applications ranging from logic synthesis{{link}} and formal verification{{link}} to extending support vector machines for multiclass classification{{Platt2000}} and approximating neural network rules{{Chorowski2011}}.

<iframe id="decision-diagram-visualization" src="/assets/beard/dist/index.html" width="100%" height="500px"></iframe>
<p style="text-align: center;font-size: 0.75rem;">Decision diagram visualization using <a href="https://github.com/pedrosbmartins/beard" target="_blank">beard</a>.</p>

Our running example is a binary expression in _disjunctive normal form_ (DNF), but there are other formulas that illustrate the compactness of decision diagrams when compared to decision trees. For instance, we could visualize <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: '(A+B)·(C+D)' });">a similar expression in _conjunctive normal form_ (CNF)</a>, the <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: 'A·B + A·C + B·C' });">3-input</a> and <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: 'A·B·C+A·B·D+A·C·D+B·C·D+A·B·E+A·D·E+A·C·E+B·C·E+C·D·E+B·D·E' });">5-input majority function</a>, or the <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: 'A XOR B XOR C' });">3-input</a> and <a href="#" class="link-button" onclick="return render(decisionDiagram, { expression: 'A XOR B XOR C XOR D' });">4-input XOR function</a>. Notice how the diagram does not grow exponentially with depth, in contrast with the tree, especially in the XOR examples.

### Diagrams as classifiers

While decision diagrams have not been widely adopted as a classification model, several constructive approaches have been proposed. The inherent difficulty in learning diagrams arises from having to decide both traditional splits (as in trees) and node merges (i.e. linking two nodes to a single child node, to form a DAG), and also from determining the best diagram topology. Therefore, most early algorithm proposals consisted of building a decision tree and then finding merges in a post-processing step, which can be computationally expensive for large problems, or of fixing the diagram structure a priori, which can be a considerable disadvantage for a general-purpose algorithm.

Some proposals were advanced over the years to overcome these problems, but decision diagrams haven't received much attention yet. While most recent papers applying diagrams as classifiers report improved metrics over trees (such as accuracy or sparsity), the fact is that the overhead of learning diagrams hinders their adoption. (Fairly brief literature reviews can be found in the papers on _optimal decision diagrams_ cited in the next section.)

That being said, decision diagrams are definitely an important addition to the toolbox of machine learning practitioners. A recent application of decision diagrams in the healthcare setting is _CARNA_[^carna], an interpretable framework for predicting high risk heart failure, which outputs, along with the patient's risk classification, interpretable patient phenotypes that characterize its predicted risk score.

#### Optimal decision diagrams

A recent trend in the literature has been the proposal of _optimal methods_ for the construction of decision diagrams for classification. This comes in step with the rise of optimal methods for decision trees, pushed by advances in hardware and in combinatorial optimization software. The work of Bertsimas and Dunn on optimal decision trees[^optimal-decision-trees] has been the root for continuous improvement of mathematical programming techniques for learning these models, while other research paths include
SAT, dynamic programming, and branch-and-bound search.

For decision diagrams, SAT[^optimal-robdds-sat] and MaxSAT-based[^optimal-robdds-maxsat] models for learning ROBDDs have been proposed. Simultaneously, another research stream in the optimal methods space is the Optimal Decision Diagrams[^optimal-decision-diagrams] (ODD) -- of which I am a co-author.

In the ODD approach, a mixed-integer linear program (MILP) is used to train decision diagrams for classification, closely following the approach of Bertsimas and Dunn, while using exponentially fewer binary variables when applied to decision tree topologies. The MILP semantic readily allows the use of additional constraints to capture fairness, parsimony, and stability requirements. Optimal decision diagrams can be trained in short computational times and achieve better accuracy than optimal decision trees, and the approach also allows for improved stability without significant accuracy losses. Its main drawback, as is the case with most optimal methods, is that the MILP formulation gets computationally intractable for very large datasets with several thousands of data points.

## Conclusion

Decision diagrams are a generalization of decision trees that help overcome the replication and fragmentation problems. Recent comparative studies have shown that they can be generally more sparse and accurate than trees as a classification model, although the overhead of learning both split and merge decisions has hindered its adoption. There is no one-size-fits-all model to solving machine learning problems, but decision diagrams can be considered an important addition to the practitioner toolbox, especially in settings where interpretability and/or sparsity is a concern.

<br />
<hr />
<br />

[^tabular-data]:
    Tabular data is a setting where deep-learning has yet to blow traditional tree-based models out of the water, as opposed to, say, computer vision and natural language processing. Not that it never will, but tree-based approaches are still preferred for general applications. For recent discussions, you may refer to these papers:

    - Gorishniy, Y.; Rubachev, I.; Khrulkov, V.; Babenko, A. Revisiting Deep Learning Models for Tabular Data. _Advances in Neural Information Processing Systems_, v. 34, 2021. [Online access](https://arxiv.org/abs/2106.11959).
    - Grinsztajn, L.; Oyallon, E.; Varoquaux, G. Why Do Tree-Based Models Still Outperform Deep Learning On Tabular Data? _NeurIPS 2022 Datasets and Benchmarks Track_, 2022. [Online access](https://arxiv.org/abs/2207.08815).
    - Shwartz-Ziv, R.; Armon, A. Tabular Data: Deep Learning is Not All You Need. _Information Fusion_, v. 81, p. 84–90, 2022. [Online access](https://arxiv.org/abs/2106.03253).

[^interpretability]:
    Do note that interpretability claims require some care. It is generally claimed that decision trees are highly intelligible for humans and that sparsity leads necessarily to better interpretability, but there are few actual user studies that really demonstrate this well, as far as I'm aware. Comparatively, at least, decision trees are inherently more transparent than the bare parameters of deep neural networks or even explanatory methods for black boxes, such as [LIME](https://christophm.github.io/interpretable-ml-book/lime.html) and other surrogate model approaches. For a discussion on explainability vs interpretability, see:

    - Rudin, C. Stop Explaining Black Box Machine Learning Models For High Stakes Decisions And Use Interpretable Models Instead. _Nature Machine Intelligence_, v. 1, n. 5, p. 206–215, 2019. [Online access](https://arxiv.org/abs/1811.10154).

[^replication-problem]:
    For a discussion on the replication problem and a feature-construction based approach for dealing with it, see:

    - Yang, DS.; Blix, G.; Rendell, L.A. The replication problem: A constructive induction approach. _Lecture Notes in Computer Science_, v. 482, 1991. [Online access](https://link.springer.com/chapter/10.1007/BFb0017003#chapter-info).

[^robdd]: For an overview of binary decision diagram algorithms, [these lecture notes](https://web.archive.org/web/20110304135553/http://configit.com/fileadmin/Configit/Documents/bdd-eap.pdf) are quite good.
[^carna]: Lamp, J.; Wu, Y.; Lamp, S.; Afriyie, P.; Bilchick, K.; Feng, L.; Mazimba, S. CARNA: Characterizing Advanced heart failure Risk and hemodyNAmic phenotypes using learned multi-valued decision diagrams. _arXiv preprint_, 2023. [Online access](https://arxiv.org/abs/2306.06801).
[^optimal-decision-trees]: Bertsimas, D.; Dunn, J. Optimal Classification Trees. _Machine Learning_, v. 106, n. 7, p. 1039–1082, 2017. [Online access](https://doi.org/10.1007/s10994-017-5633-9).
[^optimal-robdds-sat]: Cabodi, G.; Camurati, P. E.; Ignatiev, A.; Marques-Silva, J.; Palena, M.; Pasini, P. Optimizing Binary Decision Diagrams for Interpretable Machine Learning Classification. _Design, Automation & Test in Europe Conference & Exhibition_, 2021. p. 1122–1125. [Online access](https://ieeexplore.ieee.org/document/9474083).
[^optimal-robdds-maxsat]: Hu, H.; Huguet, M.-J.; Siala, M. Optimizing Binary Decision Diagrams with MaxSAT for classification. _Proceedings of the AAAI Conference on Artificial Intelligence_, v. 36, 2022. [Online access](https://arxiv.org/abs/2203.11386).
[^optimal-decision-diagrams]: Florio, A. M.; Martins, P.; Schiffer, M.; Serra, T.; Vidal, T. Optimal Decision Diagrams for Classification. _Proceedings of the AAAI Conference on Artificial Intelligence_, v. 37, 2022. [Online access](https://arxiv.org/abs/2205.14500).

<script type="text/javascript">
    let increment = 0
    const E_CHAR_CODE = 'E'.charCodeAt(0)

    function render(beardInstance, state) {
        increment = 0
        beardInstance && beardInstance.render({ ...state })
        return false
    }

    function incrementVariable() {
        if (!decisionTree || increment >= 5) return false
        increment += 1
        const expression = `(A·B·${additionalTermList().join('·')})+(C·D)`
        decisionTree.render({ expression })
        return false
    }

    function additionalTermList() {
        return [...new Array(increment)].map((_,i) => String.fromCharCode(E_CHAR_CODE+i))
    }

    const decisionTreeIframe = document.getElementById('decision-tree-visualization')
    let decisionTree

    const decisionDiagramIframe = document.getElementById('decision-diagram-visualization')
    let decisionDiagram

    window.addEventListener('message', event => {
        if (event.data.name !== 'graphvizloaded') return
        if (event.data.frameId === decisionTreeIframe.id) {
            decisionTree = decisionTreeIframe.contentWindow.beard
            render(decisionTree, { expression: '{{exampleExpression}}', variant: 'tree', options: { hideVariantSelector: true } })
        }
        if (event.data.frameId === decisionDiagramIframe.id) {
            decisionDiagram = decisionDiagramIframe.contentWindow.beard
            render(decisionDiagram, { expression: '{{exampleExpression}}', variant: 'diagram', options: { hideVariantSelector: false } })
        }
    })
</script>
