Inverted index Example
======================

Main idea
---------
In this example, it is demonstrated how to build an inverted index for a set of documents.
An inverted index is a list of documents for a term, in which the term occurs.
The map/reduce is almost the same as in the wordcount example.

Input dataset
-------------
A folder on the HDFS with plain text-documents. All words in the documents are counted, and a final result is returned.
Each line should consist of a docid, and a ":". The keyvalue input is used, it takes the part of the line before ":" as key, the rest is taken as value. 

The map
-------
Writes for each word a word:1 pair to the reducer.
For example we have 2 documents

    1: the dog is walking in the sun
    2: the sun shines very bright 

The mapper then writes the following to the reducer

    the: 1
    dog: 1
    is:  1
    walking: 1
    in: 1
    the: 1
    sun: 1
    the: 2
    sun: 2
    shines: 2
    (etc)
    
The reducer
-----------
The reducers builds a list of docids per term. Note that duplicates are not automatically removed.  
The output will then be:

    the: [1, 1, 2]
    dog: [1]
    is: [1]
    walking: [1]
    in: [1]
    sun: [2]
    (etc)
    
    