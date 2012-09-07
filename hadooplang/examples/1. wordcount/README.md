Wordcount Example
=================

Input dataset
-------------
A folder on the HDFS with plain text-documents. All words in the documents are counted, and a final result is returned.

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
    the: 1
    sun: 1
    (etc)
    
The reducer
-----------
For each distinct term, a reducer is started, and count the occurences.
The output will then be:

    the: 3
    dog: 1
    is: 1
    walking: 1
    in: 1
    sun: 2
    (etc)
    
    