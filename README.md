HadoopLang
==========
Introduction
------------
###Motivation (1)
HadoopLang is a domain specific language for the Hadoop framework.
The main motivation for the existence of this language is because the implementation is an exercise for "model driven software development" (MDSD). This is a course which is taught at Delft University of Technology, the Netherlands.


###Motivation (2)
In certain computer applications it is not unusual to have datasets containing gigabytes, or terabytes of data. In order to find information relatively quickly from these datasets, a MapReduce framework can be used. Using such a framework, information is stored at multiple computers in a network. Individual computers in such a network do not necessarily have all information available. With MapReduce information can be gathered and analyzed from this dataset. Note that analyzing datasets on large clusters may be expensive, because a lot of computers are running in parallel for one MapReduce job.

One popular MapReduce framework is Hadoop. With Hadoop, map and reduce tasks can be specified in Java, by writing mapper and reducer classes. After this, a job, or an sequence of jobs, can be created and executed in Java. The job specifies the which input data should be used, and how it should be map/reduced. 

Unfortunately using Hadoop with Java comes with some difficulties. Not all errors can be detected by the Java compiler. The kind of errors can be wrong job setup, or wrong map/reduce specifications. One can also easily make traditional programming errors, such as getting uninitialized indices from a list. As said before, a MapReduce job can be expensive, failing a job several times before getting it right can make it even more expensive.  HadoopLang tries to overcome these difficulties; using HadoopLang all (or most) boilerplate code is hidden for the developer. As a MapReduce job may also be time-expensive, HadoopLang tries to optimize code for speed where possible, by using fast constructions.


###MapReduce in a nutshell
In this section the MapReduce jobs are explained in a nutshell.

* Input: Each job takes input from an existing dataset. The input is parsed and split up into different input entries. Each input entry is given to a mapper.
* Mapper: For each input entry a mapper is started. The mapper can analyze and rewrite the data in any way, but independently of all other mappers. After it, the mapper outputs a key, value pair.
* Reducer: For each unique key, all values (from different computers) are aggregated and given to a reducer. In other words: for each unique term in the mapper keys output, one reducer is started. This reducer can iterate over all all mapper-output-values, so one can analyze this data. The reducer also writes a key, value pair.

Once a job is finished, one can start a new MapReduce job, by taking a reducer output as new input.

Language compilation
---------------------
HadoopLang -> Java + Hadoop API sourcecode -> Java Bytecode -> Runnable JAR

Compilation is not possible when there are code-related errors given in the IDE.


Language types
--------------
Java and Hadoop both have different types. For example, text in Hadoop is represented by Text-objects, where Java uses String-objects. When implementing an Hadoop-job in Java, the developer has to convert types in an explicit way. Using HadoopLang, this is done automatically, and the developer does not have to concern these type conversions.

The available types are:

* ``String``
* ``Number (double)``
* ``Boolean (True, False)``
* ``List(t)``
* ``Dict(t), is a map with String keys.``
* ``Iterator(t)``

, where ``t`` is a subtype, which can be any of the HadoopLang types.


Variables
------------
All variables in HadoopLang have a fixed type. Once a variable has been initialized, its type can not change. The type of a variable is determined implicitly. To help the developer determine the type of a variable quickly, a variable-hover is used.

Examples:

    x := "This is a String";
    y := 12;            //Number

    x  = "This is an update."
    y  = y + 12;        //makes 24

    x  = y;             //error! y is of type Number, expected String
    x := "new string";  //also error, x is already defined!

    z := True;          // Boolean, however warning: unused!

    b := a;             // Error: a is not defined!
    a := 1;

Lists, dicts, and iterators
---------------------------
###Instantiation
Creating lists and dicts can only be done by specifying the type on forehand. Lists and dicts can be initialized with initial values. It is not possible to create iterator objects with HadoopLang.

    //Create an empty string-list
    l1 := String[];
    
    //Create an initialized number-list
    l2 := Number[1, 2, 3, 4, 5];
    
    //Create an empty dict
    d1 := String{};
    
    //Create an initialized number-dict
    // note that indices should always be strings
    d2 := Number{"number1": 1, "number2": 99};
    
    
###Alteration
Adding items to a list of dict is possible. See the following code examples:

    //Add item to a list
    l1 := String[];
    l1[] = "one";
    
    //Add item to a dict
    d1 := String{};
    d1["name"] = "OBrien";
    
    //update the value
    d1["name"] = "Sisko";

    
Note that it is not possible to update values of a list. It is also not possible to get values by index. This is to prevent lookups of non-instantiated list indices. In traditional programming, one has to check on forehand whether an index is in the list or dict. If accessing an index which is not set, errors are to be expected. Since failures can be very expensive in a map-reduce job, it is undesired to have them. Therefore it is not possible to access lists and dicts directly by index. One can get values from a list or dict by looping over them.

Note that is also not possible to remove elements from a list, or unset values in a dict. It is however to filter out elements while looping. See the next subsection for more information.

Note that in the IDE one is informed with an error containing this information when accessing lists or dicts by index.
    
###Looping
The following code demonstrates how to loop over lists, dicts and iterators.

    //looping over a list
    // looping over an iterator is the same, but then l should be an iterator
    l := Number[1, 2, 3, 4, 5];
    for value in l {
        //value contains one list element and is of type Number
    }
    
    //looping over a dict
    d := Number{"number1": 1, "number2": 99};
    for key : value in d {
        //key is a String
        //value is a Number
    }
    
Note that it is allowed to iterate over an iterator multiple times. However the developer is _warned_ that the iterator may be empty. This warning is shown on all uses after a first use. _A small note: the iterator may be unused because a loop has not been executed as a result of if/else branching._

####With filter
It is also possible to filter out list elements while looping. A filter is a boolean expression. The content of the for loop is only executed for the elements for which the boolean expression holds.

    l := Number[1, 2, 3, 4, 5];
    for value in l where(value > 3){
        //only for 4 and 5 this body is executed
    }
    

Modules
-------
HadoopLang is modular. All code goes into modules. Each module should be placed in a separate file. A module starts with
``module <modulename>``. After it, imports, rewriter-, mapper-, reducer-, dataset definitions,  and store statements can follow.


###Re-usability
__Before using imports, please have a look at the known bugs section!__

It is possible to re-use rewriters, mappers and reducers from other modules. They can easily be imported with the following commands

* ``from <modulename> import rewriter <name>;``
* ``from <modulename> import mapper <name>;``
* ``from <modulename> import reducer <name>;``
 

Rewriters
---------
Rewriters can be considers to be methods, however rewriters can not store their state; on each call, the rewriter is its initial state. A rewriter always takes one argument, and optionally more arguments. It always returns one value. A single point of return is forced, as the last expressions of a rewriter should always be a return.

    rewriter <name> <inputtype> <inputvar> [, <typeargi> <varargi>]* {
        <statements>
        return <expression>;
    }


Example:

    rewriter customadd Number numa, Number numb {
        z := numa + numb;
        return z;
    }

    //use (in a mapper, reducer, or other rewriter)
    newnumber := rewrite 12 with customadd, 8;
    //newnumber now has value 20
   
    
Mappers and reducers
--------------------

Defining mappers and reducers can be done with the following statements:

    mapper <mappername> :  <keytype> <keyname> : <valuetype> : <valuename> {
        <statements>
    }

    reducer <reducername> :  <keytype> <keyname> : Iterator(<iteratortype>) : <iteratorname> {
        <statements>
    }
Mappers and reducers can emit keys and values by issuing a write with the write-statement:

    write <keyvar> : <valuevar>;

Some notes:

* Each map, and each reduce should have at least one write-statement
* When there are multiple write-statements, they should all write the same types per mapper or reducer.
* A rewriter value can only be of type ``Iterator(t)``, which can be used to iterate over all map outputs for ``<keyname>``.


Creating a job
--------------
The syntax for job creating is as following:

    <set> := <input> -> map with <mapper> -> reduce with <reducer>

, where input can be one of both ``input <inputmethod>(<args>)``, or ``input <set>``.
In order to store output in a human readable format, use the command ``store <set>``.

Example:

    x1 := input lines("dataset") -> map with parseinput -> reduce with idf;
    x2 := input x1 -> map with copyA -> reduce with copyB;
    store x2; 

Editor errors are raised when inputs for a mapper or reducer are of a wrong type. It is currently not possible to re-use a _stored_ set as input for a new job. This is because there was no time left in the assignment for writing serialize and deserialize methods for the custom Hadoop types (dict and list).

**Important** Existing output folders are automatically deleted. For now outputfolders get the name ``job_<varname> `` when calling ``<varname> := input (...);``.


Datainputs
----------
HadoopLang has support for three different input types: _lines_, _keyvalue_, and _xml_. Currently, these types are part of the language and none can be added or removed.

* _lines_ takes one argument: the input directory. Each line in the input files is treated as a new input entry. The key is the byte offset of the line start in its file (Number). The value is the lines content (String).
* _keyvalue_ takes two arguments: the input directory, and a separator. As with _lines_, each line is read as a new input entry. The part of the line up to the separator is given to the mapper as the _key_ (String), the rest of the line is given to the mapper as _value_ (String).
* _xml_ takes three arguments: the input directory, open tag, and close tag. All text between a open-close tag match is given to the mapper as value. The key is an offset value (Number).


Examples:

    x := input lines("dataset") -> (...);
    x := input keyvalue("dataset", "\t") -> (...);
    x := input xml("movies", "<movie>", "</movie>") -> (...);
    
It is also possible to chain results from an already executed map/reduce. See the previous section ("Creating a job").
    

Optimizations
-------------
As described above, writing applications with HadoopLang lets the developer focus on solving problems with Map/Reduce, without the need to concern about the Java code and Hadoop job setup. It is however, as described in the motivation, also desirable to get 'fast' code. HadoopLang has some optimizations implemented to achieve this goal. Currently the implemeted optimizations are:

* No type conversions for unused key, value input pairs. When, for example, a mapper input is not used, it will not be converted to the Java equivalent.

* Appending strings is done with StringBuilder, which is faster than the Java internal string concatenation. (This is since the Java string concatenation ``x + y`` uses a StringBuffer and converts it back by the ``.toString()`` on bytecode level. According to the Java API StringBuilder is faster than a StringBuffer, since StringBuffer is synchronized and StringBuilder is not.)

* Splitting a String can be done with the provided HadoopLang lib. It returns a token iterator instead of list with all values spitted. StringTokenizer is faster than a normal split.

Example of using the tokenizer:

    from hdplib import tokenizer;
  
      //in the correct scope
      { t := rewrite "I am a short sentence." with tokenizer;
        for token in t { /* ... */ }
      }


Note that the above optimizations are actually implemented. However a lot of other optimizations can be thought of. One, for example, is creating private statics for (non-iterator) variables which are only read. This will save time on object creation. Another optimization which can be implemented is when an input value of a mapper or reducer is directly written as output value (without reading or altering it in the steps in between). Here the object is currently converted from Hadoop type, to Java type, back to Hadoop type. This step can be omitted as an speed optimization.


Future work
-----------
The described language allows the development of an Hadoop applications by focusing only on the Map/Reduce job, without concerning type conversions, speed optimizations and boilerplate code. Unfortunately not all Hadoops strong points are possible with this language (remember: it was only a study excessive of roughly 1.5 / 2 EC). Some ideas on how this language can be improved:

* Other code optimizations (see also the previous section)
* Support for combiners, which are actually reducers. I have to study some theory on the subject first.
* Support for partitioners.
* Support for sorters.
* Let the developer choose output folders. Now the jobname is used.
* Distinguish integers and doubles, may also increase speed (but that needs further investigation to be sure).
* Give arguments to mappers and reducers, can be easily implemented by setting values to a job-config.
* Chain several maps. This is possible in Hadoop with a chained mapper job. Hadooplang may generate better code for this, by merging mappers on forehand. This may reduce the number of required type-conversions from Hadoop to Java and the other way around.
* Add a built-in type DONTCARE: _. This can for example be used to omit ``unused`` errors at places were variables must be defined (e.g. you are not interested in dict-keys).
* More, or custom, input formats.
* Adding custom types. For example have pre-defined datastructs (see example below). This will also involve serialization and de-serialization of these datastructs.


Example for custom datastructs
    
    datastruct Person {
        String person;
        Number age;
        String email;
    }


Examples
--------
Several examples can be found in the examples directory (which is ./utest/_demoapps).


Known bugs
----------
As far as I know, there are only a few bugs at the moment. On a save operation, the analysis may sometimes crashes. All included .hdp modules are required to be opened in the editor, and resolving objects to other files does not work. 

There seems also to be a bug related to Spoofax. When the HadoopLang syntax is altered, make sure you also alter both the ``/syntax/Hadooplang.sdf`` and ``/systax/grammar/Expressions.sdf``. Adding and removing a space is fine. My guess is that something goes wrong with caching compilations.

Finally: Java keywords should not be used as variable names. I know it's stupid and I blame myself for it. I only found this out at a very late stage of development. A fix will be underway, but after handing in the assignment.

Other than that, I am not aware of other bugs. Unfortunately there is no way to be sure. If you find one, please contact me at ``TheUnknownCylon``@GitHub or ``TheUC``@freenode.


Similar Work
------------
I did not do an extensive research on similar work. However you may want to have a look at PigLatin (http://pig.apache.org/). PigLatin is a query language for large datasets, which also has an interactive mode. PigLatin also uses Hadoop for data analysis.


Advanced topics
-------------------
###Calling external Java

It is possible to call external Java methods. Note that currently no checks are done,; once using these Java methods, there is no guarantee that the Hadooplang compaliber will produce to valid Javacode. _Note that the developer is informed in its IDE when creating this construction._

Code for calling external java functions:

	rewriter <name> <type> <key> [, <argtype> <argname>]*
	    alias <package>.<class>.<method>
	    returns <type>

Example:

	rewriter uppercase String text
	    alias org.hdplib.StringLib.uppercase
	    returns String

Input and output types for these methods related to the Java class types are defined in the following table. Note that interfaces should be fully implemented, or otherwise no assertions can be made of good or bad behavior. Also note that external calls get references to Java objects, not copies.

    
    HadoopLang <--> Java
     String           String
     Number           Double
     Boolean          Boolean
     List(t)          List<t>        (interface implementation)
     Dict(t)          Map<String, t> (interface implementation)
     Iterator(t)      Iterable<t>    (interface implementation)
     