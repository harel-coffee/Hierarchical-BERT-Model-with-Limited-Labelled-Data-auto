# A Sentence-level Hierarchical BERT Model for Document Classification with Limited Labelled Data

This repository is temporarily associated with paper [Lu, J., Henchion, M., Bacher, I. and Mac Namee, B., 2021. A Sentence-level Hierarchical BERT Model for Document Classification with Limited Labelled Data. arXiv preprint arXiv:2106.06738.](https://arxiv.org/pdf/2106.06738.pdf)

## Usage

### Dependencies
Tested Python 3.6, and requiring the following packages, which are available via PIP:

* Required: [numpy >= 1.19.5](http://www.numpy.org/)
* Required: [scikit-learn >= 0.21.1](http://scikit-learn.org/stable/)
* Required: [pandas >= 1.1.5](https://pandas.pydata.org/)
* Required: [gensim >= 3.7.3](https://radimrehurek.com/gensim/)
* Required: [matplotlib >= 3.3.3](https://matplotlib.org/)
* Required: [torch >= 1.9.0](https://pytorch.org/)
* Required: [transformers >= 4.8.2](https://huggingface.co/transformers/)
* Required: [Keras >= 2.0.8](https://keras.io/)
* Required: [Tensorflow >= 1.14.0](https://www.tensorflow.org/)
* Required: [FastText model trained with Wikipedia 300-dimension](https://fasttext.cc/docs/en/pretrained-vectors.html)
* Required: [GloVe model trained with Gigaword and Wikipedia 200-dimension](https://nlp.stanford.edu/projects/glove/)
* Required: packaging >= 20.0


### Step 1. Data Processing

The first step is encoding raw text data into different high-dimensional vectorised representations. The raw text data should be stored in directory "raw_corpora/", each dataset should have its individual directory, for example, the "longer_moviereview/" directory under folder "raw_corpora/". The input corpus of documents should consist of plain text files stored in csv format (two files for one corpus, one for documents belong to class A and one for documents for class B) with a columan named as __text__. It should be noted that the csv file must be named in the format _#datasetname_neg_text.csv_ or _#datasetname_pos_text.csv_. Each row corresponding to one document in that corpus, the format can be refered to the csv file in the sample directory "raw_corpora/longer_moviereview/". Then we can start preprocessing text data and converting them into vectors by:

    python encode_text.py -d dataset_name -t encoding_methods
    
The options of -t are `hbm` (corresponding to the sentence representation generated by the token-level RoBERTa encoder in the paper), `roberta-base` and `fasttext`, for example `-t roberta-base,fasttext` means encoding documents by RoBERTa and FastText respectively. The encoded documents are stored in directory "dataset/", while the FastText document representations are stored in "dataset/fasttext/" and other representations are stored in "dataset/roberta-base/". It should be noted that the sentence representations for hbm is suffixed by ".pt" and the document representations generated by RoBERTa are suffixed by ".csv"(average all tokens to represent a document) or "\_cls.csv" (using classifier token "\<s\>" to represent a document). Due to the upload file size limit, we did not upload sample ".pt" files but you can generate yours. For encoding by FastText, you need to download the pretrained FastText model in advance (see __Dependencies__).

### Step 2. Run Hierarchical BERT Model (HBM)(our approach)

We can evaluate the Hierarchical BERT Model (HBM) with limited number of labelled data (in this experiment, we subsample the fully labelled dataset to simulate this low-shot scenario) by:

    python run_hbm.py -d dataset_name -l learning_rate -e num_of_epochs -r random_seeds -s training_set_size

The `training_set_size` can be random numbers up to 200 (also, you can customise the maximum number by editing the script), for example  `-s 50,100,150` means the training HBM with 50, 100 and 150 labelled instances respectively.  The `random_seeds` are random state for subsampling training set from the whole dataset. For example `-r 1988,1999 -s 50,100` will training HBM with four different training sets, i.e. 50 labelled instances sampled by seed 1988, 50 labelled instances sampled by seed 1999, 100 labelled instances sampled by seed 1988 and 100 labelled instances sampled by seed 1999.
The script then evaluate the performance of HBM in the rest testing set (i.e. the whole dataset minus the 200 instances that sampled out as the training set, the details can be referred in the paper). The evaluation results are stored in directory "outputs/". Furthermore, the concrete results of each step are stored in "outputs/hbm_results/". The results files starting with "auc_" store the AUC score results while files starting with "raw_" store the confusion matrix (tp, tn, fp, fn).

### Step 3. Run fine-tuned RoBERTa (baseline)

Similar to the above settings, we can evaluate the fine-tuned RoBERTa performance with limited number of labelled data by:

    python run_fine_tuned_roberta.py -d dataset_name -l learning_rate -e num_of_epochs -r random_seeds -s training_set_size
    
Similarly, the evaluation results are stored in directory "outputs/". Furthermore, the concrete results of each step are stored in "outputs/fine_tuned_results/". It should be noted that the directories "fine_tuned_data/", "fine_tuned_outputs/" and "fine_tuned_cache/" are used for stored auxiliary information generated during the fine-tuning and hence these three directories should be created in advance.

### Step 4. Run Hierarchical Attention Networks (HAN)(baseline)

Similar to the above settings, we can evaluate the Hierarchical Attention Networks with limited number of labelled data by:
    
    python run_han.py -d dataset_name -l learning_rate -e num_of_epochs -r random_seeds -s training_set_size
    
The preprocessing and text encoding (with GloVe) are also integrated into this script and you should download the GloVe model in advance (see __dependencies__). The evaluation results are stored in dictory "outputs/".

### Step 5. Run SVM-based methods (i.e. pretrained RoBERTa + SVM and FastText + SVM) (baseline)

We can also evaluate the performance of RoBERTa+SVM and FastText+SVM withi limited number of labelled data by:

    python run_svm-based.py -d dataset_name -t text_representation -r random_seeds -s training_set_size
   
It should be noted that `-t text_representation` indicates the encoding method you choose, the valid options are `fasttext` and `roberta-base`. The evaluation results are stored in directory "outputs/".

### Step 6. Visualise informative sentences inferred by HBM in a document 

When we run the script in __Step 2__, besides the AUC scores on testing set, we can also get the attention scores of each sentences that measure whether sentences contribute a lot in forming the document representation. Hence, these attention scores can serve as clue of whether the sentences are important or not. The attention scores are stored in "attentions/#dataset_name/". You can visualise this attention scores by playing with the notebook __Visualization_of_informative_sentences.ipynb__.

## Toy Experiments

We play with the code using the [MovieReview Sentiment dataset](https://www.cs.cornell.edu/people/pabo/movie-review-data/) consisting of 1000 negative movie reviews and 1000 positive movie reviews. The distribution of the number of sentences per document is shown below (maximum length 118 and avg length 33.97):
![image](https://user-images.githubusercontent.com/16153974/127650224-c0d33b13-3027-4125-834c-86c1ca549f70.png)

We evaluate the performance of various methods by the setting of training size [50, 100], randome states [1988, 1989]. The performance of each method is shown below (AUC score):
![image](https://user-images.githubusercontent.com/16153974/127651353-62ce7fc2-837f-4dd4-be5b-37904842dcab.png)

You can also play with the notebook to check the informative sentences suggested by the HBM as shown below (examples taken from the testing set and highlighted ones are important sentences):
### A postive movie review
![image](https://user-images.githubusercontent.com/16153974/127651682-06a330b2-09c8-424b-9da3-626c37205108.png)
### A negatvie movie review
![image](https://user-images.githubusercontent.com/16153974/127652134-0b5a72ea-72df-4c8c-8a3d-af5de2487e21.png)


## Setup of experiments in the paper

__FastText + SVM__: We use 300-dimensional word vectors constructed by a FastText language model pre-trained with the Wikipedia corpus ([Joulin et al., 2016](https://arxiv.org/pdf/1612.03651.pdf)). Averaged word embeddings are used as the representation of the document. For preprocessing, all text is converted to lowercase and we remove all punctuation and stop words. SVM is used as the classifier. We tune the hyper-parameters of the SVM classifier using a grid-search based on 5-fold cross-validation performed on the training set, after that, we re-train the classifier with optimised hyper-parameters. This hyper-parameter tuning method is applied in RoBERTa + SVM as well.

__RoBERTa + SVM__: We use 768-dimensional word vectors generated by a pre-trained RoBERTa language model ([Liu et al., 2019](https://arxiv.org/pdf/1907.11692.pdf)). We do not fine-tune the pre-trained language model and use the averaged word vectors as the representation of the document. Since all BERT-based models are configured to take as input a maximum of 512 tokens, we divided the long documents with _W_ words into _k = W/511_ fractions, which is then fed into the model to infer the representation of each fraction (each fraction has a "\<S\>" token in front of 511 tokens, so, 512 tokens in total). Based on the approach of ([Sun et al., 2020](https://arxiv.org/pdf/1905.05583.pdf)), the vector of each fraction is the average embeddings of words in that fraction, and the representation of the whole text sequence is the mean of all _k_ fraction vectors. For preprocessing, the only operation performed is to convert all tokens to lowercase. SVM is used as the classifier.

__Fine-tuned RoBERTa__: For the document classification task, fine-tuning RoBERTa means adding a softmax layer on top of the RoBERTa encoder output and fine-tuning all parameters in the model.  In this experiment, we fine-tune the same 768-dimensional pre-trained RoBERTa model with a small training set. The settings of all hyper-parameters follow ([Liu et al., 2019](https://arxiv.org/pdf/1907.11692.pdf)). we set the learning rate to _1\times10^-4_ and the batch size to 4, and use the Adam optimizer \cite{kingma2014adam} with _epsilon equals to _1*10^-8_ through hyperparameter tuning. However, since we assume that the amount of labelled data available for training is small, we do not have the luxury of a hold out validation set to use to implement early stopping during model fine tuning. Instead, after training for 15 epochs we roll back to the model with the lowest loss based on the training dataset. This rollback strategy is also applied to HAN and HBM due to the limited number of instances in training sets. For preprocessing, the only operation performed is to convert all tokens to lowercase.
    
__Hierarchical Attention Network__: Following ([Yang et al., 2016](https://www.cs.cmu.edu/~./hovy/papers/16HLT-hierarchical-attention-networks.pdf)), we apply two levels of Bi-GRU with attention mechanism for document classification. All words are first converted to word vectors using GloVe ([Pennington et al., 2014](https://aclanthology.org/D14-1162.pdf)) (300 dimension version pre-trained using the wiki gigaword corpus) and fed into a word-level Bi-GRU with attention mechanism to form sentence vectors. After that, a sentence vector along with its context sentence vectors are input into sentence-level Bi-GRU with attention mechanism to form the document representation which is then passed to a softmax layer for final prediction. For preprocessing, the only operation performed is to convert all tokens to lowercase, and separate documents into sentences.\footnote{We apply Python NLTK sent_tokenize function to split documents into sentences.}

__Hierarchical BERT Model__: For HBM, we set the number of BERT layers to 4, and the maximum number of sentences to 114, 64, 128, 128, 100, and 64 for the _Movie Review_, _Multi-domain Customer Review_, _Blog Author Gender_, _Guardian 2013_, _Reuters_ and _20 Newsgroups_ datasets respectively, these values are based on the length of documents in these datasets. After some preliminary experiments, we set the attention head to 1, the learning rate to _2*10^{-5}_, dropout probability to 0.01, used 50 epochs, set the batch size to 4 and used the Adam optimizer with _epsilon_ equals to _1*10^{-8}_. The only text preprocessing operation performed is to convert all tokens to lowercase and split documents into sentences. 
