---
title: Tagger
tag: class
source: spacy/pipeline/tagger.pyx
teaser: 'Pipeline component for part-of-speech tagging'
api_base_class: /api/pipe
api_string_name: tagger
api_trainable: true
---

A trainable pipeline component to predict part-of-speech tags for any
part-of-speech tag set.

In the pre-trained pipelines, the tag schemas vary by language; see the
[individual model pages](/models) for details.

## Assigned Attributes {#assigned-attributes}

Predictions are assigned to `Token.tag`.

| Location     | Value                              |
| ------------ | ---------------------------------- |
| `Token.tag`  | The part of speech (hash). ~~int~~ |
| `Token.tag_` | The part of speech. ~~str~~        |

## Config and implementation {#config}

The default config is defined by the pipeline component factory and describes
how the component should be configured. You can override its settings via the
`config` argument on [`nlp.add_pipe`](/api/language#add_pipe) or in your
[`config.cfg` for training](/usage/training#config). See the
[model architectures](/api/architectures) documentation for details on the
architectures and their arguments and hyperparameters.

> #### Example
>
> ```python
> from spacy.pipeline.tagger import DEFAULT_TAGGER_MODEL
> config = {"model": DEFAULT_TAGGER_MODEL}
> nlp.add_pipe("tagger", config=config)
> ```

| Setting                                  | Description                                                                                                                                                                                                                                                                                            |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `model`                                  | A model instance that predicts the tag probabilities. The output vectors should match the number of tags in size, and be normalized as probabilities (all scores between 0 and 1, with the rows summing to `1`). Defaults to [Tagger](/api/architectures#Tagger). ~~Model[List[Doc], List[Floats2d]]~~ |
| `overwrite` <Tag variant="new">3.2</Tag> | Whether existing annotation is overwritten. Defaults to `False`. ~~bool~~                                                                                                                                                                                                                              |
| `scorer` <Tag variant="new">3.2</Tag>    | The scoring method. Defaults to [`Scorer.score_token_attr`](/api/scorer#score_token_attr) for the attribute `"tag"`. ~~Optional[Callable]~~                                                                                                                                                            |

```python
%%GITHUB_SPACY/spacy/pipeline/tagger.pyx
```

## Tagger.\_\_init\_\_ {#init tag="method"}

> #### Example
>
> ```python
> # Construction via add_pipe with default model
> tagger = nlp.add_pipe("tagger")
>
> # Construction via create_pipe with custom model
> config = {"model": {"@architectures": "my_tagger"}}
> tagger = nlp.add_pipe("tagger", config=config)
>
> # Construction from class
> from spacy.pipeline import Tagger
> tagger = Tagger(nlp.vocab, model)
> ```

Create a new pipeline instance. In your application, you would normally use a
shortcut for this and instantiate the component using its string name and
[`nlp.add_pipe`](/api/language#add_pipe).

| Name                                     | Description                                                                                                                                                                                                                                           |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `vocab`                                  | The shared vocabulary. ~~Vocab~~                                                                                                                                                                                                                      |
| `model`                                  | A model instance that predicts the tag probabilities. The output vectors should match the number of tags in size, and be normalized as probabilities (all scores between 0 and 1, with the rows summing to `1`). ~~Model[List[Doc], List[Floats2d]]~~ |
| `name`                                   | String name of the component instance. Used to add entries to the `losses` during training. ~~str~~                                                                                                                                                   |
| _keyword-only_                           |                                                                                                                                                                                                                                                       |
| `overwrite` <Tag variant="new">3.2</Tag> | Whether existing annotation is overwritten. Defaults to `False`. ~~bool~~                                                                                                                                                                             |
| `scorer` <Tag variant="new">3.2</Tag>    | The scoring method. Defaults to [`Scorer.score_token_attr`](/api/scorer#score_token_attr) for the attribute `"tag"`. ~~Optional[Callable]~~                                                                                                           |

## Tagger.\_\_call\_\_ {#call tag="method"}

Apply the pipe to one document. The document is modified in place, and returned.
This usually happens under the hood when the `nlp` object is called on a text
and all pipeline components are applied to the `Doc` in order. Both
[`__call__`](/api/tagger#call) and [`pipe`](/api/tagger#pipe) delegate to the
[`predict`](/api/tagger#predict) and
[`set_annotations`](/api/tagger#set_annotations) methods.

> #### Example
>
> ```python
> doc = nlp("This is a sentence.")
> tagger = nlp.add_pipe("tagger")
> # This usually happens under the hood
> processed = tagger(doc)
> ```

| Name        | Description                      |
| ----------- | -------------------------------- |
| `doc`       | The document to process. ~~Doc~~ |
| **RETURNS** | The processed document. ~~Doc~~  |

## Tagger.pipe {#pipe tag="method"}

Apply the pipe to a stream of documents. This usually happens under the hood
when the `nlp` object is called on a text and all pipeline components are
applied to the `Doc` in order. Both [`__call__`](/api/tagger#call) and
[`pipe`](/api/tagger#pipe) delegate to the [`predict`](/api/tagger#predict) and
[`set_annotations`](/api/tagger#set_annotations) methods.

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> for doc in tagger.pipe(docs, batch_size=50):
>     pass
> ```

| Name           | Description                                                   |
| -------------- | ------------------------------------------------------------- |
| `stream`       | A stream of documents. ~~Iterable[Doc]~~                      |
| _keyword-only_ |                                                               |
| `batch_size`   | The number of documents to buffer. Defaults to `128`. ~~int~~ |
| **YIELDS**     | The processed documents in order. ~~Doc~~                     |

## Tagger.initialize {#initialize tag="method" new="3"}

Initialize the component for training. `get_examples` should be a function that
returns an iterable of [`Example`](/api/example) objects. The data examples are
used to **initialize the model** of the component and can either be the full
training data or a representative sample. Initialization includes validating the
network,
[inferring missing shapes](https://thinc.ai/docs/usage-models#validation) and
setting up the label scheme based on the data. This method is typically called
by [`Language.initialize`](/api/language#initialize) and lets you customize
arguments it receives via the
[`[initialize.components]`](/api/data-formats#config-initialize) block in the
config.

<Infobox variant="warning" title="Changed in v3.0" id="begin_training">

This method was previously called `begin_training`.

</Infobox>

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> tagger.initialize(lambda: [], nlp=nlp)
> ```
>
> ```ini
> ### config.cfg
> [initialize.components.tagger]
>
> [initialize.components.tagger.labels]
> @readers = "spacy.read_labels.v1"
> path = "corpus/labels/tagger.json
> ```

| Name           | Description                                                                                                                                                                                                                                                                                                                                                                                                |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `get_examples` | Function that returns gold-standard annotations in the form of [`Example`](/api/example) objects. ~~Callable[[], Iterable[Example]]~~                                                                                                                                                                                                                                                                      |
| _keyword-only_ |                                                                                                                                                                                                                                                                                                                                                                                                            |
| `nlp`          | The current `nlp` object. Defaults to `None`. ~~Optional[Language]~~                                                                                                                                                                                                                                                                                                                                       |
| `labels`       | The label information to add to the component, as provided by the [`label_data`](#label_data) property after initialization. To generate a reusable JSON file from your data, you should run the [`init labels`](/api/cli#init-labels) command. If no labels are provided, the `get_examples` callback is used to extract the labels from the data, which may be a lot slower. ~~Optional[Iterable[str]]~~ |

## Tagger.predict {#predict tag="method"}

Apply the component's model to a batch of [`Doc`](/api/doc) objects, without
modifying them.

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> scores = tagger.predict([doc1, doc2])
> ```

| Name        | Description                                 |
| ----------- | ------------------------------------------- |
| `docs`      | The documents to predict. ~~Iterable[Doc]~~ |
| **RETURNS** | The model's prediction for each document.   |

## Tagger.set_annotations {#set_annotations tag="method"}

Modify a batch of [`Doc`](/api/doc) objects, using pre-computed scores.

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> scores = tagger.predict([doc1, doc2])
> tagger.set_annotations([doc1, doc2], scores)
> ```

| Name     | Description                                      |
| -------- | ------------------------------------------------ |
| `docs`   | The documents to modify. ~~Iterable[Doc]~~       |
| `scores` | The scores to set, produced by `Tagger.predict`. |

## Tagger.update {#update tag="method"}

Learn from a batch of [`Example`](/api/example) objects containing the
predictions and gold-standard annotations, and update the component's model.
Delegates to [`predict`](/api/tagger#predict) and
[`get_loss`](/api/tagger#get_loss).

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> optimizer = nlp.initialize()
> losses = tagger.update(examples, sgd=optimizer)
> ```

| Name           | Description                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `examples`     | A batch of [`Example`](/api/example) objects to learn from. ~~Iterable[Example]~~                                        |
| _keyword-only_ |                                                                                                                          |
| `drop`         | The dropout rate. ~~float~~                                                                                              |
| `sgd`          | An optimizer. Will be created via [`create_optimizer`](#create_optimizer) if not set. ~~Optional[Optimizer]~~            |
| `losses`       | Optional record of the loss during training. Updated using the component name as the key. ~~Optional[Dict[str, float]]~~ |
| **RETURNS**    | The updated `losses` dictionary. ~~Dict[str, float]~~                                                                    |

## Tagger.rehearse {#rehearse tag="method,experimental" new="3"}

Perform a "rehearsal" update from a batch of data. Rehearsal updates teach the
current model to make predictions similar to an initial model, to try to address
the "catastrophic forgetting" problem. This feature is experimental.

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> optimizer = nlp.resume_training()
> losses = tagger.rehearse(examples, sgd=optimizer)
> ```

| Name           | Description                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `examples`     | A batch of [`Example`](/api/example) objects to learn from. ~~Iterable[Example]~~                                        |
| _keyword-only_ |                                                                                                                          |
| `drop`         | The dropout rate. ~~float~~                                                                                              |
| `sgd`          | An optimizer. Will be created via [`create_optimizer`](#create_optimizer) if not set. ~~Optional[Optimizer]~~            |
| `losses`       | Optional record of the loss during training. Updated using the component name as the key. ~~Optional[Dict[str, float]]~~ |
| **RETURNS**    | The updated `losses` dictionary. ~~Dict[str, float]~~                                                                    |

## Tagger.get_loss {#get_loss tag="method"}

Find the loss and gradient of loss for the batch of documents and their
predicted scores.

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> scores = tagger.predict([eg.predicted for eg in examples])
> loss, d_loss = tagger.get_loss(examples, scores)
> ```

| Name        | Description                                                                 |
| ----------- | --------------------------------------------------------------------------- |
| `examples`  | The batch of examples. ~~Iterable[Example]~~                                |
| `scores`    | Scores representing the model's predictions.                                |
| **RETURNS** | The loss and the gradient, i.e. `(loss, gradient)`. ~~Tuple[float, float]~~ |

## Tagger.create_optimizer {#create_optimizer tag="method"}

Create an optimizer for the pipeline component.

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> optimizer = tagger.create_optimizer()
> ```

| Name        | Description                  |
| ----------- | ---------------------------- |
| **RETURNS** | The optimizer. ~~Optimizer~~ |

## Tagger.use_params {#use_params tag="method, contextmanager"}

Modify the pipe's model, to use the given parameter values. At the end of the
context, the original parameters are restored.

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> with tagger.use_params(optimizer.averages):
>     tagger.to_disk("/best_model")
> ```

| Name     | Description                                        |
| -------- | -------------------------------------------------- |
| `params` | The parameter values to use in the model. ~~dict~~ |

## Tagger.add_label {#add_label tag="method"}

Add a new label to the pipe. Raises an error if the output dimension is already
set, or if the model has already been fully [initialized](#initialize). Note
that you don't have to call this method if you provide a **representative data
sample** to the [`initialize`](#initialize) method. In this case, all labels
found in the sample will be automatically added to the model, and the output
dimension will be [inferred](/usage/layers-architectures#thinc-shape-inference)
automatically.

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> tagger.add_label("MY_LABEL")
> ```

| Name        | Description                                                 |
| ----------- | ----------------------------------------------------------- |
| `label`     | The label to add. ~~str~~                                   |
| **RETURNS** | `0` if the label is already present, otherwise `1`. ~~int~~ |

## Tagger.to_disk {#to_disk tag="method"}

Serialize the pipe to disk.

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> tagger.to_disk("/path/to/tagger")
> ```

| Name           | Description                                                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `path`         | A path to a directory, which will be created if it doesn't exist. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| _keyword-only_ |                                                                                                                                            |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~                                                |

## Tagger.from_disk {#from_disk tag="method"}

Load the pipe from disk. Modifies the object in place and returns it.

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> tagger.from_disk("/path/to/tagger")
> ```

| Name           | Description                                                                                     |
| -------------- | ----------------------------------------------------------------------------------------------- |
| `path`         | A path to a directory. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| _keyword-only_ |                                                                                                 |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~     |
| **RETURNS**    | The modified `Tagger` object. ~~Tagger~~                                                        |

## Tagger.to_bytes {#to_bytes tag="method"}

> #### Example
>
> ```python
> tagger = nlp.add_pipe("tagger")
> tagger_bytes = tagger.to_bytes()
> ```

Serialize the pipe to a bytestring.

| Name           | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| _keyword-only_ |                                                                                             |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~ |
| **RETURNS**    | The serialized form of the `Tagger` object. ~~bytes~~                                       |

## Tagger.from_bytes {#from_bytes tag="method"}

Load the pipe from a bytestring. Modifies the object in place and returns it.

> #### Example
>
> ```python
> tagger_bytes = tagger.to_bytes()
> tagger = nlp.add_pipe("tagger")
> tagger.from_bytes(tagger_bytes)
> ```

| Name           | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| `bytes_data`   | The data to load from. ~~bytes~~                                                            |
| _keyword-only_ |                                                                                             |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~ |
| **RETURNS**    | The `Tagger` object. ~~Tagger~~                                                             |

## Tagger.labels {#labels tag="property"}

The labels currently added to the component.

> #### Example
>
> ```python
> tagger.add_label("MY_LABEL")
> assert "MY_LABEL" in tagger.labels
> ```

| Name        | Description                                            |
| ----------- | ------------------------------------------------------ |
| **RETURNS** | The labels added to the component. ~~Tuple[str, ...]~~ |

## Tagger.label_data {#label_data tag="property" new="3"}

The labels currently added to the component and their internal meta information.
This is the data generated by [`init labels`](/api/cli#init-labels) and used by
[`Tagger.initialize`](/api/tagger#initialize) to initialize the model with a
pre-defined label set.

> #### Example
>
> ```python
> labels = tagger.label_data
> tagger.initialize(lambda: [], nlp=nlp, labels=labels)
> ```

| Name        | Description                                                |
| ----------- | ---------------------------------------------------------- |
| **RETURNS** | The label data added to the component. ~~Tuple[str, ...]~~ |

## Serialization fields {#serialization-fields}

During serialization, spaCy will export several data fields used to restore
different aspects of the object. If needed, you can exclude them from
serialization by passing in the string names via the `exclude` argument.

> #### Example
>
> ```python
> data = tagger.to_disk("/path", exclude=["vocab"])
> ```

| Name    | Description                                                    |
| ------- | -------------------------------------------------------------- |
| `vocab` | The shared [`Vocab`](/api/vocab).                              |
| `cfg`   | The config file. You usually don't want to exclude this.       |
| `model` | The binary model data. You usually don't want to exclude this. |
