# Mapping MOSES models into the AtomSpace

This document contains suggestions to map models, their scores, and
their features into AtomSpace hypergraphs.

## Usage

scripts to import moses model csv files exported from R moses analyses here:

### export_models_and_fitness.sh  <MODEL_CSV_FILE>  -o <OUTPUT_FILE>
to convert models and their scores into a scheme readily dumpable into
the AtomSpace.  column headers:  "","Sensitivity","Pos Pred Value"

### relate_features_and_genes.sh  <FEATURE_CSV_FILE>  -o <OUTPUT_FILE>
to generate scheme code to relate MOSES features and their
corresponding genes.  column headers:  gene,Freq,level

## Models

Models are exported in the following format

```
EquivalenceLink <1, 1>
    PredicateNode <MODEL_PREDICATE_NAME>
    <MODEL_BODY>
```

## Features

We need to related GeneNodes <GENE_NAME>, used in the GO description,
and PredicateNodes <GENE_NAME>, used in the MOSES models. For that I
suggest to use the predicate "overexpressed" as follows:

```
EquivalenceLink <1, 1>
    PredicateNode <GENE_NAME>
    LambdaLink
        VariableNode "$X"
        EvaluationLink
            PredicateNode "overexpressed"
            ListLink
                GeneNode <GENE_NAME>
                VariableNode "$X"
```
which says that PredicateNode <GENE_NAME> over sample $X is
equivalent to "GeneNode <GENE_NAME> is overexpressed in sample $X".

## Fitnesses

I'm discussing 3 fitnesses, accuracy, sensitivity, and precision. Plus a word about confidence.

### Accuracy

ACC = (TP + TN) / (P + N)

We define an Accuracy predicate, that takes a model and dataset (or
target feature) as arguments. The model, $M, is itself a predicate
that evaluates to 1 (the confidence is let aside for now) when the
individual $X is classified positively, 0 when it is classified
negatively.

Similarity the target feature, $D, is also a predicate that evaluates
to 1 when the individual $X has its target feature active, 0
otherwise.

```
EquivalenceLink <1, 1>
    BindLink
        ListLink
            $M
            $D
        EvaluationLink
            PredicateNode "accuracy"
            ListLink
                $M
                $D
    BindLink
        ListLink
            $M
            $D
        AverageLink
            $X
            EquivalenceLink
                ExecutionOutputLink
                    GetStrength
                    EvaluationLink
                        $M
                        $X
                ExecutionOutputLink
                    GetStrength
                    EvaluationLink
                        $D
                        $X
```

It turns out the TV on the AverageLink is gonna match the accuracy,
given $M and $D. Indeed, the accuracy is the average number of times
the model is correct with respect to the dataset. With this
representation, given the dataset and the model, PLN can directly
build the Accuracy predicate.

In the absence of dataset, and given the accuracy of each model, one
may directly write down the Accuracy predicate for each model, and
target feature:

```
EvaluationLink <model accuracy>
    PredicateNode "accuracy"
    ListLink
        PredicateNode <MODEL>
        PredicateNode <TARGET FEATURE>
```

### Precision

The cool thing about precision is that it translates directly into an
Implication TV strength. that is

```
ImplicationLink <TV.s = model precision>
    PredicateNode <MODEL>
    PredicateNode <TARGET FEATURE>
```

Indeed, According to PLN (assuming all individuals are equiprobable)

TV.s = Sum_x min(P(x), Q(x)) / Sum_x P(x)

where P correspond to the predicate of a model, x runs over the
individuals of the dataset.

This corresponds indeed to the precision

precision = TP / (TP + FP)

as Sum_x P(x) is indeed the number of positively classified
individuals (TP + FP), and Sum_x min(P(x), Q(x)) the number of
correctly classified individuals, TP.

### Recall

Similarly recall is easily translated into an Implication TV
strength. that is

```
ImplicationLink <TV.s = model recall>
    PredicateNode <TARGET FEATURE>
    PredicateNode <MODEL>
```

given that

recall = TP / (TP + FN)

### Confidence

The confidence can be

c = n / (n+k)

where n is the number of individuals, and k is a parameter.
