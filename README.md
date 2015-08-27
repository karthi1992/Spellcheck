# **Overview**
The SpellCheck component is designed to provide inline query suggestions based on other, similar, terms. The basis for these suggestions can be terms in a field in Solr, externally created text files, or fields in other Lucene indexes.

## **Configuring the SpellCheckComponent**
Define Spell Check in solrconfig.xml
Here, we follow a Index based solr spellcheker

IndexBasedSpellChecker
The IndexBasedSpellChecker uses a Solr index as the basis for a parallel index used for spell checking. It requires defining a field as the basis for the index terms; a common practice is to copy terms from some fields (such as title, body, etc.) to another field created for spell checking. Here is a simple example of configuring solrconfig.xml with the IndexBasedSpellChecker:

>      <searchComponent name="spellcheck" class="solr.SpellCheckComponent">  
<lst name="spellchecker">
<str name="name">default</str>    
<str name="classname">solr.IndexBasedSpellChecker</str>    
<str name="spellcheckIndexDir">./spellchecker</str>    
<str name="field">content</str>
<str name="accuracy">0.7</str> 
<float name="thresholdTokenFrequency">.0001</float>
<str name="comparatorClass">freq</str>
<str name="buildOnOptimize">true</str>     
<str name="buildOnCommit">true</str>
</lst>
