<h1 align="center">
<p>NeuSpell: A Neural Spelling Correction Toolkit
</h1>

# Contents

- [Installation & Quick Start](#Installation)
- Toolkit
    - [Introduction](#Introduction)
    - [Pretrained models](#Pretrained-models)
    - [Datasets](#Datasets)
    - [Demo Setup](#Demo-Setup)
- [Finetuning on custom data and creating new models](#Finetuning-on-custom-data-and-creating-new-models)
- [Applications](#Potential-applications-for-practitioners)
- [Additional Requirements](#Additional-requirements)

# Updates

- April 2021:
    - `neuspell` is now available through pip. See [Installation through pip](#Installation-through-pip)
    - Added support for different transformer-based models such DistilBERT, XLM-RoBERTa, etc.
      See [Finetuning on custom data and creating new models](#Finetuning-on-custom-data-and-creating-new-models)
      section for more details.
- March, 2021: Code-base reformatted. Addressed some bug fixes.
- November, 2020: Neuspell's ```BERT``` pretrained model is now available as part of huggingface models
  as ```murali1996/bert-base-cased-spell-correction```. We provide an example code snippet
  at [./scripts/huggingface](./scripts/huggingface/huggingface-snippet-for-neuspell.py) for curious practitioners.
- September, 2020: This work is accepted at EMNLP 2020 (system demonstrations)

# Installation

```bash
git clone https://github.com/neuspell/neuspell; cd neuspell
pip install -e .
```

To install extra requirements,

```bash
pip install -r extras-requirements.txt
```

or individually as:

```bash
pip install -e .[elmo]
pip install -e .[spacy]
```

NOTE: For _zsh_, use ".[elmo]" and ".[spacy]" instead

Additionally, ```spacy models``` can be downloaded as:

```bash
python -m spacy download en_core_web_sm
```

Follow [Additional Requirements](#Additional-requirements) for installing non-neural spell checkers- ```Aspell```
and ```Jamspell```.

Then, download pretrained models of `neuspell` following [Pretrained models](#Pretrained-models)

Here is a quick-start code snippet (command line usage). (See [```test.py```](test.py) for more usage patterns)

```python
import neuspell

""" see available checkers """
print(f"available checkers: {neuspell.__all__}")
# → available checkers: ['BertsclstmChecker', 'CnnlstmChecker', 'NestedlstmChecker', 'SclstmChecker', 'SclstmbertChecker', 'BertChecker', 'SclstmelmoChecker', 'ElmosclstmChecker']

""" select spell checkers & load """
checker = BertChecker()
checker.from_pretrained()

""" spell correction """
checker.correct("I luk foward to receving your reply")
# → "I look forward to receiving your reply"
checker.correct_strings(["I luk foward to receving your reply", ])
# → ["I look forward to receiving your reply"]
checker.correct_from_file(src="noisy_texts.txt")
# → "Found 450 mistakes in 322 lines, total_lines=350"

""" evaluation of models """
checker.evaluate(clean_file="bea60k.txt", corrupt_file="bea60k.noise.txt")
# → data size: 63044
# → total inference time for this data is: 998.13 secs
# → total token count: 1032061
# → confusion table: corr2corr:940937, corr2incorr:21060,
#                    incorr2corr:55889, incorr2incorr:14175
# → accuracy is 96.58%
# → word correction rate is 79.76%
```

Alternatively, once can also select and load a spell checker differently as follows:

```python
from neuspell import SclstmChecker

checker = SclstmChecker()
checker = checker.add_("elmo", at="input")  # "elmo" or "bert", "input" or "output"
checker.from_pretrained()
```

This feature of adding ELMO or BERT model is currently supported for selected models.
See [List of neural models in the toolkit](#List-of-neural-models-in-the-toolkit) for details.

# Installation through pip

```bash
pip install neuspell
```

In v1.0, `allennlp` library is not installed which is used for models containing ELMO. To utilize them, do a source
install as in [Installation & Quick Start](#Installation)

# Introduction

NeuSpell is an open-source toolkit for context sensitive spelling correction in English. This toolkit comprises of 10
spell checkers, with evaluations on naturally occurring mis-spellings from multiple (publicly available) sources. To
make neural models for spell checking context dependent, (i) we train neural models using spelling errors in context,
synthetically constructed by reverse engineering isolated mis-spellings; and  (ii) use richer representations of the
context.This toolkit enables NLP practitioners to use our proposed and existing spelling correction systems, both via a
simple unified command line, as well as a web interface. Among many potential applications, we demonstrate the utility
of our spell-checkers in combating adversarial misspellings.

##### Live demo available at <http://neuspell.github.io/>

<p align="center">
    <br>
    <img src="https://github.com/neuspell/neuspell/blob/master/images/ui.png?raw=true" width="400"/>
    <br>
<p>

##### List of neural models in the toolkit:

- [```CNN-LSTM```](https://drive.google.com/file/d/14XiDY4BJ144fVGE2cfWfwyjnMwBcwhNa/view?usp=sharing)
- [```SC-LSTM```](https://drive.google.com/file/d/1OvbkdBXawnefQF1d-tUrd9lxiAH1ULtr/view?usp=sharing)
- [```Nested-LSTM```](https://drive.google.com/file/d/19ZhWvBaZqrsP5cGqBJdFPtufdyBqQprI/view?usp=sharing)
- [```BERT```](https://huggingface.co/transformers/bertology.html)
- [```SC-LSTM plus ELMO (at input)```](https://drive.google.com/file/d/1mjLFuQ0vWOOpPqTVkFZ_MSHiuVUmgHSK/view?usp=sharing)
- [```SC-LSTM plus ELMO (at output)```](https://drive.google.com/file/d/1P8vX9ByOBQpN9oeho_iOJmFJByv1ifI5/view?usp=sharing)
- [```SC-LSTM plus BERT (at input)```](https://huggingface.co/transformers/bertology.html)
- [```SC-LSTM plus BERT (at output)```](https://huggingface.co/transformers/bertology.html)

<p align="center">
    <br>
    <img src="https://github.com/neuspell/neuspell/blob/master/images/pipeline.jpeg?raw=true" width="400"/>
    <br>
    This pipeline corresponds to the `SC-LSTM plus ELMO (at input)` model.
<p>

##### Performances

| Spell<br>Checker    | Word<br>Correction <br>Rate | Time per<br>sentence <br>(in milliseconds) |
|----------|----------------------|--------------------------------------|
| ```Aspell``` | 48.7 | 7.3* |
|``` Jamspell``` | 68.9 | 2.6* |
|```CNN-LSTM``` |75.8 |  4.2|
|```SC-LSTM``` | 76.7| 2.8 |
|```Nested-LSTM``` |77.3 | 6.4|
|```BERT``` | 79.1| 7.1|
|```SC-LSTM plus ELMO (at input)``` |<b> 79.8</b>|15.8 |
|```SC-LSTM plus ELMO (at output)``` | 78.5| 16.3|
|```SC-LSTM plus BERT (at input)``` | 77.0| 6.7|
|```SC-LSTM plus BERT (at output)``` | 76.0| 7.2|

Performance of different correctors in the NeuSpell toolkit on the  ```BEA-60K```  dataset with real-world spelling
mistakes. ∗ indicates evaluation on a CPU (for others we use a GeForce RTX 2080 Ti GPU).

### Pretrained models

##### Download Checkpoints

Run the following to download checkpoints of all neural models

```
cd data/checkpoints
python download_checkpoints.py 
```

See ```data/checkpoints/README.md``` for more details. You can alternatively choose to download only selected models'
checkpoints.

### Datasets

##### Download datasets

Run the following to download datasets

```
cd data/traintest
python download_datafiles.py 
```

See ```data/traintest/README.md``` for more details.

##### Synthetic Training Dataset Creation

The toolkit offers 4 kinds of noising strategies to generate synthetic parallel training data to train neural models for
spell correction.

- ```RANDOM```
- ```Word Replacement```
- ```Probabilistic Replacement```
- A combination of ```Word Replacement``` and ```Probabilistic Replacement```

Train files are dubbed with names ```.random```, ```.word```, ```.prob```, ```.probword``` respectively. For each
strategy, we noise ∼20% of the tokens in the clean corpus. We use 1.6 Million sentences from
the [```One billion word benchmark```](https://arxiv.org/abs/1312.3005) dataset as our clean corpus.

### Demo Setup

In order to setup a demo, follow these steps:

- Do [Installation](#Installation)
- Download [checkpoints](#Pretrained-models)
- Start a flask server at [./scripts/flask-server](./scripts/flask-server) by
  running `CUDA_VISIBLE_DEVICES=0 python app.py`
  (on GPU) or `python app.py` (on CPU)

# Finetuning on custom data and creating new models

### Finetuning on top of `neuspell` pretrained models

```python
from neuspell import BertChecker

checker = BertChecker()
checker.from_pretrained()
checker.finetune(clean_file="sample_clean.txt", corrupt_file="sample_corrupt.txt", data_dir="default")
```

This feature is only available for `BertChecker` and `ElmosclstmChecker`.

### Training other Transformers/BERT-based models

We now support initializing a huggingface model and finetuning it on your custom data. Here is a code snippet
demonstrating that:

First mark your files containing clean and corrupt texts in a line-seperated format

```python
from neuspell.commons import DEFAULT_TRAINTEST_DATA_PATH

data_dir = DEFAULT_TRAINTEST_DATA_PATH
clean_file = "sample_clean.txt"
corrupt_file = "sample_corrupt.txt"
```

```python
from neuspell.seq_modeling.helpers import load_data, train_validation_split
from neuspell.seq_modeling.helpers import get_tokens
from neuspell import BertChecker

# Step-0: Load your train and test files, create a validation split
train_data = load_data(data_dir, clean_file, corrupt_file)
train_data, valid_data = train_validation_split(train_data, 0.8, seed=11690)

# Step-1: Create vocab file. This serves as the target vocab file and we use the defined model's default huggingface
# tokenizer to tokenize inputs appropriately.
vocab = get_tokens([i[0] for i in train_data], keep_simple=True, min_max_freq=(1, float("inf")), topk=100000)

# # Step-2: Initialize a model
checker = BertChecker(device="cuda")
checker.from_huggingface(bert_pretrained_name_or_path="distilbert-base-cased", vocab=vocab)

# Step-3: Finetune the model on your dataset
checker.finetune(clean_file=clean_file, corrupt_file=corrupt_file, data_dir=data_dir)
```

You can further evaluate your model on a custom data as follows:

```python
from neuspell import BertChecker

checker = BertChecker()
checker.from_pretrained(
    bert_pretrained_name_or_path="distilbert-base-cased",
    ckpt_path=f"{data_dir}/new_models/distilbert-base-cased"  # "<folder where the model is saved>"
)
checker.evaluate(clean_file=clean_file, corrupt_file=corrupt_file, data_dir=data_dir)
```

### Multilingual Models

Following usage above, once can now seamlessly utilize multilingual models such as `xlm-roberta-base`,
`bert-base-multilingual-cased` and `distilbert-base-multilingual-cased` on a non-English script.

# Potential applications for practitioners

- Defenses against adversarial attacks in NLP
    - example implementation available in folder ```./applications/Adversarial-Misspellings-arxiv```.
      See [README.md](./applications/README.md).
- Improving OCR text correction systems
- Improving grammatical error correction systems
- Improving Intent/Domain classifiers in conversational AI
- Spell Checking in Collaboration and Productivity tools

# Additional requirements

Requirements for ```Aspell``` checker:

```
wget https://files.pythonhosted.org/packages/53/30/d995126fe8c4800f7a9b31aa0e7e5b2896f5f84db4b7513df746b2a286da/aspell-python-py3-1.15.tar.bz2
tar -C . -xvf aspell-python-py3-1.15.tar.bz2
cd aspell-python-py3-1.15
python setup.py install
```

Requirements for ```Jamspell``` checker:

```
sudo apt-get install -y swig3.0
wget -P ./ https://github.com/bakwc/JamSpell-models/raw/master/en.tar.gz
tar xf ./en.tar.gz --directory ./
```

# Citation

```
@inproceedings{jayanthi-etal-2020-neuspell,
    title = "{N}eu{S}pell: A Neural Spelling Correction Toolkit",
    author = "Jayanthi, Sai Muralidhar  and
      Pruthi, Danish  and
      Neubig, Graham",
    booktitle = "Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing: System Demonstrations",
    month = oct,
    year = "2020",
    address = "Online",
    publisher = "Association for Computational Linguistics",
    url = "https://www.aclweb.org/anthology/2020.emnlp-demos.21",
    doi = "10.18653/v1/2020.emnlp-demos.21",
    pages = "158--164",
    abstract = "We introduce NeuSpell, an open-source toolkit for spelling correction in English. Our toolkit comprises ten different models, and benchmarks them on naturally occurring misspellings from multiple sources. We find that many systems do not adequately leverage the context around the misspelt token. To remedy this, (i) we train neural models using spelling errors in context, synthetically constructed by reverse engineering isolated misspellings; and (ii) use richer representations of the context. By training on our synthetic examples, correction rates improve by 9{\%} (absolute) compared to the case when models are trained on randomly sampled character perturbations. Using richer contextual representations boosts the correction rate by another 3{\%}. Our toolkit enables practitioners to use our proposed and existing spelling correction systems, both via a simple unified command line, as well as a web interface. Among many potential applications, we demonstrate the utility of our spell-checkers in combating adversarial misspellings. The toolkit can be accessed at neuspell.github.io.",
}
```

You can contact the authors at jsaimurali001 [at] gmail [dot] com
