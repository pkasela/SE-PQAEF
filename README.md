# SE-PQAEF

```
- download_and_extract_data.sh # script to download the stackexchange dump from archive.org  
- combine_data.py # combine the data downladed before
- combine_data_single_comm.py # for single community experiment only
- single_community_pipe.sh # for single community expertiment only
- mapping.json # mapping file for elasticsearch with best hyperparameters
- data_exploration.ipynb # jupyter notebook with dataexploration written in the paper

# the following folders are specific for expertiments and are described below
- 03_best_answers
- 03_best_answers_model
- 04_best_expert
- 04_best_expert_model
```

```
- 03_best_answers: # in this folder we create the dataset
  - create_best_answer_data.py # create the first portion of data
  - create_final_run.py # combines data with bm25 to create the final dataset
  - elastic.py # helper class for elastic server
  - get_bm25_run.py # creates the bm25 first stage rank using the mapping file (in the main folder) 
  - get_remaining_bm25_run.py # not needed unless elastic search returns an error on some queries (timeout etc.)
  - optimize_bm25.py # finds the best values for b and k for elasticsearch
  - pipeline.sh # script with the various flags for each script that needs to run to create the dataset
- 03_best_answers_model: # train and test the model for cQ&A
  - dataloader: # files needed for loading data for training and testing 
    - dataloader.py
    - utils.py
  - model: # files defining the models
    - loss.py
    - model.py
  - saved_models: # store the tranined model here
    - model_0.pt
    - model_10.pt
  - community_average.py # for separate community experiment
  - create_answer_embeddings.py # create embeddings for all answers and stores it
  - create_model_zero.py # create a .pt file from the pre-trained MiniLM
  - fuse_optimizer.py # optimizes the values for lambda for each run using validation set and computes score on test set
  - testing_bm25.py # create re-ranking combining bm25 and bert like models
  - testing_pers_tag.py # create the TAG model run
  - training.py # training DistilBERT (or any BERT like model)
  - pipeline.sh # pipeline to run the whole training and testing procedure
  - testing_pipeline.sh # just if you need to test without tranining
```

```
- 04_best_expert:
  - data_creation.py
  - elastic.py
  - get_bm25_run.py
  - pipeline.sh
- 04_best_expert_model:
  - dataloader: # files needed for loading data for training and testing 
    - dataloader.py
    - utils.py
  - model: # files defining the models
    - loss.py
    - model.py
  - fuse.py # beware you need to change test weights manually
  - testing_reranker.py
  - testing_tag_based.py
  - pipeline.sh # set flags and run the whole experiments
```

The dataset is divided as follows:
```
- answers.csv (Id,CreationDate,Score,Title,Tags,CommentCount,ParentId,AccountId,Text)
- comments.csv (Id,PostId,Score,Text,CreationDate,ContentLicense,AccountId)
- postlinks.csv (PostId,RelatedPostId,LinkType)
- questions_with_answer.csv (Id,PostTypeId,AcceptedAnswerId,CreationDate,Score,ViewCount,Body,OwnerUserId,Title,Tags,AnswerCount,CommentCount,FavoriteCount,ParentId,Id_,AccountId)
- questions.csv (Id,AcceptedAnswerId,CreationDate,Score,ViewCount,Body,Title,Tags,AnswerCount,CommentCount,FavoriteCount,AccountId,Text,Community)
- tags.csv (Id,TagName,Count,PostId,Body,Community)
- users.csv (Id,Reputation,CreationDate,DisplayName,AboutMe,Views,UpVotes,DownVotes,AccountId)
```
These are the complete dump of various communities with cleaned text and combined toghether. This can be used to create data for other tasks as well, for example using the postlinks.csv one can create duplication question retrieval for example, or one can create a classification task for communities using the community field of the questions and/or answers.
The two dataset we created are organized as follows:

```
answer_retrieval:
- train: 
  - bm25_run.json # the first stage ranker list for each training query
  - data_pers.jsonl # pers version of the dataset with first stage results excluded
  - data.jsonl # base version of the dataset with first stage results excluded
  - queries_pers.jsonl # pers version of the dataset with first stage results included
  - queries.jsonl # base version of the dataset with first stage results included
- val:
  - *other_models*_run.json # various run used in the papers
  - bm25_run.json 
  - data_pers.jsonl 
  - data.jsonl 
  - queries_pers.jsonl 
  - queries.jsonl 
- test
  - *other_models*_run.json # various run used in the papers
  - bm25_run.json 
  - data_pers.jsonl 
  - data.jsonl 
  - queries_pers.jsonl 
  - queries.jsonl 
- answer_collection.json # the complete answer collection from which you need to retrieve the answers
- question_collection.json # the questions used to create train, val and test splits (can be useful for some specific models)
```

```
best_expert:
- train:
  - bm25_run.json # the first stage ranker list for each training query
  - data.jsonl # each query with relevant user associated and other metadata
- val:
  - bm25_run.json # the first stage ranker list for each training query
  - data.jsonl # each query with relevant user associated and other metadata
  - *various_models_runs*.json # the runs generated with the experiment in paper
- test:
  - bm25_run.json # the first stage ranker list for each training query
  - data.jsonl # each query with relevant user associated and other metadata
  - *various_models_runs*.json # the runs generated with the experiment in paper
- answer_collection.json # the complete answer collection from which you need to retrieve the answers
- question_collection.json # the questions used to create train, val and test splits (can be useful for some specific models)
- expert_test_data.json # collection of answers of only the experts before the test split data (used in experiments)
```




EF Results

 Model                         | P@1             | R@3             | R@5             | MRR@5           | $\lambda$    
-------------------------------|-----------------|-----------------|-----------------|-----------------|--------------
 BM25                          | 0.134           | 0.255           | 0.314           | 0.200           | -            
 BM25 + TAG                    | 0.150           | 0.286           | 0.361           | 0.226           | (.4,.6)      
 MiniLM $_{\text{SBERT}}$      | 0.141           | 0.268           | 0.327           | 0.209           | -            
 MiniLM $_{\text{SBERT}}$ + TAG| 0.154           | 0.291           | 0.367           | 0.230           | (.4, .6      
 BM25 + DistilBERT             | 0.141           | 0.262           | 0.322           | 0.207           | (.6, .4)     
 BM25 + DistilBERT + TAG       | **0.160**       | **0.296**       | **0.369**       | **0.236**       | (.3, .2, .5) 




Performance on all communities separately!

| Community      | Model (BM25 +) | P@1   | NDCG@3 | NDCG@10 | R@100 | MAP@100 | $\lambda$  |
|----------------|----------------|-------|--------|---------|-------|---------|------------|
| Academia       | MiniLM         | 0.438 | 0.382  | 0.395   | 0.489 | 0.344   | (.1,.9)    |
|                | MiniLM + TAG   | 0.453 | 0.392  | 0.403   | 0.489 | 0.352   | (.1,.8,.1) |
| Anime          | MiniLM + TAG   | 0.650 | 0.682  | 0.714   | 0.856 | 0.683   | (.1,.9,.0) |
| Apple          | MiniLM         | 0.327 | 0.351  | 0.381   | 0.514 | 0.349   | (.1,.9)    |
|                | MiniLM + TAG   | 0.335 | 0.361  | 0.389   | 0.514 | 0.357   | (.1,.8,.1) |
| Bicycles       | MiniLM         | 0.405 | 0.380  | 0.421   | 0.600 | 0.365   | (.1,.9)    |
|                | MiniLM + TAG   | 0.436 | 0.405  | 0.441   | 0.600 | 0.386   | (.1,.8,.1) |
| Boardgames     | MiniLM         | 0.681 | 0.694  | 0.728   | 0.866 | 0.692   | (.1,.9)    |
|                | MiniLM + TAG   | 0.696 | 0.702  | 0.736   | 0.866 | 0.699   | (.1,.8,.1) |
| Buddhism       | MiniLM + TAG   | 0.490 | 0.387  | 0.397   | 0.544 | 0.334   | (.3,.7,.0) |
| Christianity   | MiniLM         | 0.534 | 0.505  | 0.555   | 0.783 | 0.497   | (.2,.8)    |
|                | MiniLM + TAG   | 0.549 | 0.521  | 0.564   | 0.783 | 0.507   | (.1,.8,.1) |
| Cooking        | MiniLM         | 0.600 | 0.567  | 0.600   | 0.719 | 0.553   | (.1,.9)    |
|                | MiniLM + TAG   | 0.619 | 0.583  | 0.614   | 0.719 | 0.568   | (.1,.8,.1) |
| DIY            | MiniLM         | 0.323 | 0.313  | 0.346   | 0.501 | 0.302   | (.1,.9)    |
|                | MiniLM + TAG   | 0.335 | 0.324  | 0.356   | 0.501 | 0.312   | (.1,.8,.1) |
| Expatriates    | MiniLM + TAG   | 0.596 | 0.653  | 0.682   | 0.832 | 0.645   | (.1,.9,.0) |
| Fitness        | MiniLM + TAG   | 0.568 | 0.575  | 0.613   | 0.760 | 0.567   | (.2,.8,.0) |
| Freelancing    | MiniLM + TAG   | 0.513 | 0.472  | 0.506   | 0.654 | 0.457   | (.1,.9,.0) |
| Gaming         | MiniLM         | 0.510 | 0.534  | 0.562   | 0.686 | 0.532   | (.1,.9)    |
|                | MiniLM + TAG   | 0.519 | 0.547  | 0.571   | 0.686 | 0.541   | (.1,.8,.1) |
| Gardening      | MiniLM         | 0.344 | 0.362  | 0.396   | 0.520 | 0.359   | (.1,.9)    |
|                | MiniLM + TAG   | 0.345 | 0.369  | 0.399   | 0.520 | 0.363   | (.1,.8,.1) |
| Genealogy      | MiniLM + TAG   | 0.592 | 0.605  | 0.631   | 0.779 | 0.594   | (.3,.7,.0) |
| Health         | MiniLM + TAG   | 0.718 | 0.765  | 0.797   | 0.934 | 0.765   | (.2,.8,.0) |
| Gaming         | MiniLM         | 0.510 | 0.534  | 0.562   | 0.686 | 0.532   | (.1,.9)    |
|                | MiniLM + TAG   | 0.519 | 0.547  | 0.571   | 0.686 | 0.541   | (.1,.8,.1) |
| Hermeneutics   | MiniLM         | 0.589 | 0.538  | 0.593   | 0.828 | 0.526   | (.2,.8)    |
|                | MiniLM + TAG   | 0.632 | 0.570  | 0.617   | 0.828 | 0.552   | (.1,.8,.1) |
| Hinduism       | MiniLM         | 0.388 | 0.415  | 0.459   | 0.686 | 0.416   | (.2,.8)    |
|                | MiniLM + TAG   | 0.382 | 0.410  | 0.457   | 0.686 | 0.412   | (.1,.8,.1) |
| History        | MiniLM + TAG   | 0.740 | 0.735  | 0.764   | 0.862 | 0.730   | (.2,.8,.0) |
| Hsm            | MiniLM + TAG   | 0.666 | 0.707  | 0.737   | 0.870 | 0.690   | (.2,.8,.0) |
| Interpersonal  | MiniLM + TAG   | 0.663 | 0.617  | 0.653   | 0.739 | 0.604   | (.2,.8,.0) |
| Islam          | MiniLM         | 0.382 | 0.412  | 0.453   | 0.642 | 0.410   | (.1,.9)    |
|                | MiniLM + TAG   | 0.395 | 0.427  | 0.464   | 0.642 | 0.421   | (.1,.8,.1) |
| Judaism        | MiniLM + TAG   | 0.363 | 0.387  | 0.432   | 0.649 | 0.388   | (.2,.8,.0) |
| Law            | MiniLM         | 0.663 | 0.647  | 0.678   | 0.803 | 0.639   | (.2,.8)    |
|                | MiniLM + TAG   | 0.677 | 0.657  | 0.687   | 0.803 | 0.649   | (.1,.8,.1) |
| Lifehacks      | MiniLM         | 0.714 | 0.601  | 0.617   | 0.703 | 0.553   | (.1,.9)    |
|                | MiniLM + TAG   | 0.714 | 0.621  | 0.631   | 0.703 | 0.568   | (.1,.8,.1) |
| Linguistics    | MiniLM + TAG   | 0.584 | 0.588  | 0.630   | 0.794 | 0.587   | (.2,.8,.0) |
| Literature     | MiniLM + TAG   | 0.871 | 0.878  | 0.889   | 0.934 | 0.876   | (.3,.7,.0) |
| Martialarts    | MiniLM         | 0.630 | 0.599  | 0.645   | 0.796 | 0.596   | (.1,.9)    |
|                | MiniLM + TAG   | 0.640 | 0.628  | 0.660   | 0.796 | 0.612   | (.1,.8,.1) |
| Money          | MiniLM         | 0.545 | 0.535  | 0.563   | 0.706 | 0.515   | (.2,.8)    |
|                | MiniLM + TAG   | 0.559 | 0.542  | 0.571   | 0.706 | 0.523   | (.1,.8,.1) |
| Movies         | MiniLM         | 0.713 | 0.722  | 0.753   | 0.865 | 0.724   | (.1,.9)    |
|                | MiniLM + TAG   | 0.728 | 0.735  | 0.762   | 0.865 | 0.735   | (.1,.8,.1) |
| Music          | MiniLM         | 0.508 | 0.447  | 0.476   | 0.602 | 0.418   | (.2,.8)    |
|                | MiniLM + TAG   | 0.522 | 0.460  | 0.486   | 0.602 | 0.427   | (.1,.8,.1) |
| Musicfans      | MiniLM + TAG   | 0.531 | 0.531  | 0.560   | 0.693 | 0.539   | (.1,.9,.0) |
| Opensource     | MiniLM         | 0.574 | 0.593  | 0.621   | 0.771 | 0.581   | (.2,.8)    |
|                | MiniLM + TAG   | 0.577 | 0.598  | 0.622   | 0.771 | 0.581   | (.1,.8,.1) |
| Outdoors       | MiniLM + TAG   | 0.681 | 0.643  | 0.675   | 0.819 | 0.629   | (.1,.9,.0) |
| Parenting      | MiniLM + TAG   | 0.485 | 0.430  | 0.452   | 0.602 | 0.399   | (.1,.9,.0) |
| Pets           | MiniLM         | 0.509 | 0.531  | 0.565   | 0.685 | 0.523   | (.1,.9)    |
|                | MiniLM + TAG   | 0.519 | 0.549  | 0.581   | 0.685 | 0.541   | (.1,.8,.1) |
| Philosophy     | MiniLM + TAG   | 0.568 | 0.514  | 0.546   | 0.707 | 0.491   | (.2,.8,.0) |
| Politics       | MiniLM + TAG   | 0.659 | 0.630  | 0.659   | 0.814 | 0.608   | (.1,.9,.0) |
| Rpg            | MiniLM         | 0.657 | 0.646  | 0.685   | 0.849 | 0.640   | (.2,.8)    |
|                | MiniLM + TAG   | 0.677 | 0.660  | 0.695   | 0.849 | 0.651   | (.1,.8,.1) |
| Scifi          | MiniLM         | 0.532 | 0.563  | 0.596   | 0.745 | 0.559   | (.2,.8)    |
|                | MiniLM + TAG   | 0.549 | 0.574  | 0.606   | 0.745 | 0.569   | (.1,.8,.1) |
| Skeptics       | MiniLM + TAG   | 0.862 | 0.869  | 0.887   | 0.969 | 0.867   | (.2,.8,.0) |
| Sound          | MiniLM         | 0.377 | 0.410  | 0.451   | 0.626 | 0.405   | (.2,.8)    |
|                | MiniLM + TAG   | 0.380 | 0.423  | 0.454   | 0.626 | 0.413   | (.1,.8,.1) |
| Sports         | MiniLM         | 0.673 | 0.721  | 0.756   | 0.902 | 0.724   | (.2,.8)    |
|                | MiniLM + TAG   | 0.692 | 0.743  | 0.775   | 0.902 | 0.740   | (.1,.8,.1) |
| Sustainability | MiniLM         | 0.657 | 0.677  | 0.735   | 0.895 | 0.675   | (.1,.9)    |
|                | MiniLM + TAG   | 694   | 0.716  | 0.763   | 0.895 | 0.706   | (.1,.8,.1) |
| Travel         | MiniLM + TAG   | 0.546 | 0.547  | 0.576   | 0.700 | 0.530   | (.1,.9,.0) |
| Vegetarianism  | MiniLM + TAG   | 0.623 | 0.626  | 0.678   | 0.869 | 0.641   | (.3,.7,.0) |
| Woodworking    | MiniLM + TAG   | 0.656 | 0.654  | 0.692   | 0.847 | 0.645   | (.2,.8,.0) |
| Workplace      | MiniLM + TAG   | 0.574 | 0.444  | 0.429   | 0.495 | 0.359   | (.3,.7,.0) |
| Writers        | MiniLM + TAG   | 0.561 | 0.490  | 0.516   | 0.644 | 0.466   | (.2,.8,.0) |
| Average        | MiniLM         | 0.519 | 0.506  | 0.536   | 0.677 | 0.492   | -          |
|                | MiniLM + TAG   | 0.530 | 0.515  | 0.544   | 0.677 | 0.500   | -          |

