# **Overview**
The SpellCheck component is designed to provide inline query suggestions based on other, similar, terms. The basis for these suggestions can be terms in a field in Solr, externally created text files, or fields in other Lucene indexes.

## **Configuring the SpellCheckComponent**
### **Search Component**
Define Spell Check in solrconfig.xml.
Here, we follow a Index based solr spellcheker


The IndexBasedSpellChecker uses a Solr index as the basis for a parallel index used for spell checking. It requires defining a field as the basis for the index terms; a common practice is to copy terms from some fields (such as title, body, etc.) to another field created for spell checking. Here is a simple example of configuring solrconfig.xml with the IndexBasedSpellChecker:

>      <searchComponent name="spellcheck" class="solr.SpellCheckComponent">  
>      <lst name="spellchecker">
>      <str name="name">default</str>    
>      <str name="classname">solr.IndexBasedSpellChecker</str>    
>      <str name="spellcheckIndexDir">./spellchecker</str>    
>      <str name="field">content</str>
>      <str name="accuracy">0.7</str> 
>      <float name="thresholdTokenFrequency">.0001</float>
>      <str name="comparatorClass">freq</str>
>      <str name="buildOnOptimize">true</str>     
>      <str name="buildOnCommit">true</str>
>      </lst>
>      <str name="queryAnalyzerFieldType">text_general</str>
>      </searchComponent>

The first element defines the searchComponent to use the solr.SpellCheckComponent. The classname is the specific implementation of the SpellCheckComponent, in this case solr.IndexBasedSpellChecker. Defining the classname is optional; if not defined, it will default to IndexBasedSpellChecker.

The spellcheckIndexDir defines the location of the directory that holds the spellcheck index, while the field defines the source field (defined in schema.xml) for spell check terms. When choosing a field for the spellcheck index, it's best to avoid a heavily processed field to get more accurate results. If the field has many word variations from processing synonyms and/or stemming, the dictionary will be created with those variations in addition to more valid spelling data.

The accuracy can also be set, whose values range from 0 to 1. the accuracy here is set to be 0.7.

The Comparatorclass enables to fetch the suggestions based on the decreasing order of the word frequencies.

The queryAnalyzer field type is set to text_general.

Finally, buildOnCommit defines whether to build the spell check index at every commit (that is, every time new documents are added to the index). It is optional, and can be omitted if you would rather set it to false.

### **Request Handler**
Queries will be sent to a RequestHandler. If every request should generate a suggestion, then you would add the following to the requestHandler that you are using:

>     <str name="spellcheck">true</str>

One of the possible parameters is the spellcheck.dictionary to use, and multiples can be defined. With multiple dictionaries, all specified dictionaries are consulted and results are interleaved. Collations are created with combinations from the different spellcheckers, with care taken that multiple overlapping corrections do not occur in the same collation.

>     <requestHandler name="/spell" class="solr.SearchHandler" startup="lazy">
>     <lst name="defaults">
>     <str name="df">text</str>
>     <str name="spellcheck.dictionary">default</str>
>     <str name="spellcheck.build">true</str> 
>     <str name="spellcheck">true</str>
>     <str name="spellcheck.extendedResults">true</str>       
>     <str name="spellcheck.count">20</str>
>     <str name="spellcheck.alternativeTermCount">5</str>
>     <str name="spellcheck.maxResultsForSuggest">5</str>       
>     <str name="spellcheck.collate">true</str>
>     <str name="spellcheck.collateExtendedResults">true</str>  
>     <str name="spellcheck.maxCollationTries">1</str>
>     <str name="spellcheck.maxCollations">1</str>
>     <str name="spellcheck.onlyMorePopular">true</str>            
>     </lst>
>     <arr name="last-components">
>     <str>spellcheck</str>
>     </arr>
>     </requestHandler>

Querying the Spellchecker:
>     http://178.63.22.132:8983/solr/amc_test/spell?df=text&spellcheck.q=%22microsof%22&spellcheck=true&wt=xml&spellcheck.collateParam.q.op=AND

Results:
>     <response>
>     <lst name="responseHeader">
>     <int name="status">0</int>
>     <int name="QTime">1390</int>
>     </lst>
>     <str name="command">build</str>
>     <result name="response" numFound="0" start="0"/>
>     <lst name="spellcheck">
>     <lst name="suggestions">
>     <lst name="microsof">
>     <int name="numFound">3</int>
>     <int name="startOffset">1</int>
>     <int name="endOffset">9</int>
>     <int name="origFreq">0</int>
>     <arr name="suggestion">
>     <lst>
>     <str name="word">microsoft</str>
>     <int name="freq">222</int>
>     </lst>
>     <lst>
>     <str name="word">microsd</str>
>     <int name="freq">63</int>
>     </lst>
>     <lst>
>     <str name="word">#microsoft</str>
>     <int name="freq">8</int>
>     </lst>
>     </arr>
>     </lst>
>     <bool name="correctlySpelled">false</bool>
>     </lst>
>     </lst>
>     </response>

