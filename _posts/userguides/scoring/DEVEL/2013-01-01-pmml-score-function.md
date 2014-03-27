---
layout: post
title: FreshenerContext
categories: [userguides, scoring, devel]
tags : [scoring-ug]
order : 8
version : devel
description: Description of PMML compliant ScoreFunction.
---

<div id="accordion-container">
  <h2 class="accordion-header"> JpmmlScoreFunction.java </h2>
    <div class="accordion-content">
    <script src="http://gist-it.appspot.com/github/kijiproject/kiji-scoring/raw/{{site.scoring_devel_branch}}/src/main/java/org/kiji/scoring/lib/JpmmlScoreFunction.java"> </script>
  </div>
</div>

<h3 style="margin-top:0px;padding-top:10px;"> JpmmlScoreFunction </h3>
The JpmmlScoreFunction provides a way to deploy a [PMML](http://www.dmg.org/pmml-v4-2.html) compliant model that has been trained externally to the Kiji ecosystem to the kiji-model-repository. Scores will be calculated using the kiji-scoring-server. This score function internally uses the [Jpmml](https://github.com/jpmml) library to parse PMML models

TODO: describe what data is expected and what data is output.

<h3 style="margin-top:0px;padding-top:10px;"> Prerequisites </h3>
To use the JpmmlScoreFunction several conditions must be met:

* Must use a model that is supported: [https://github.com/jpmml/jpmml-evaluator](https://github.com/jpmml/jpmml-evaluator) (see README).
* Must already have a trained model xml file.
* Must have a table with a layout that has two columns:
    * One column containing a record with one field per predictor MiningField ([http://www.dmg.org/v4-2/MiningSchema.html](http://www.dmg.org/v4-2/MiningSchema.html)).
    * One column containing a record that will be the output from the model (one record field per predicted/output field).
    * The type of the fields in both the predictor record and result record must match the provided type in the pmml DataDictionary ([http://www.dmg.org/v4-2/DataDictionary.html](http://www.dmg.org/v4-2/DataDictionary.html)).
* Model must not use PMML extensions ([http://www.dmg.org/v4-2/GeneralStructure.html#xsdElement_Extension](http://www.dmg.org/v4-2/GeneralStructure.html#xsdElement_Extension))

<h3 style="margin-top:0px;padding-top:10px;"> Pmml-Avro Data Type Mapping </h3>
The JpmmlScoreFunction attempts to convert pmml data types into corresponding avro data types (if specified). If the type of a field is not specified, a best effort conversion will take place.

<table>
  <tr>
    <td>PMML Data Type</td>
    <td>Avro Data Type</td>
    <td>Notes</td>
  </tr>
  <tr>
    <td>STRING</td>
    <td>string</td>
    <td></td>
  </tr>
  <tr>
    <td>INTEGER</td>
    <td>long</td>
    <td>Pmml has no "long" type</td>
  </tr>
  <tr>
    <td>FLOAT</td>
    <td>float</td>
    <td></td>
  </tr>
  <tr>
    <td>DOUBLE</td>
    <td>double</td>
    <td></td>
  </tr>
  <tr>
    <td>BOOLEAN</td>
    <td>boolean</td>
    <td></td>
  </tr>
  <tr>
    <td>DATE/TIME/DATE_TIME</td>
    <td>string</td>
    <td>Formatted using ISO8601</td>
  </tr>
  <tr>
    <td>DATE_DAYS_SINCE_0DATE_DAYS_SINCE_1960
DATE_DAYS_SINCE_1970
DATE_DAYS_SINCE_1980
TIME_SECONDS
DATE_TIME_SECONDS_SINCE_0
DATE_TIME_SECONDS_SINCE_1960
DATE_TIME_SECONDS_SINCE_1970
DATE_TIME_SECONDS_SINCE_1980</td>
    <td>int</td>
    <td>JPMML only provides these as integers.</td>
  </tr>
</table>


<h3 style="margin-top:0px;padding-top:10px;"> Deploying a JpmmlScoreFunction with the 'model-repo pmml' tool </h3>

1. Place the pmml xml file in a location accessible to your account on hdfs or your local filesystem.

2. Generate a model container descriptor by running the model-repo pmml tool against the generated pmml xml file with syntax:kiji model-repo pmml \    --table=kiji://my/kiji/table \    --model-file=file:///path/to/pmml/xml/file.xml \    --model-name=nameOfModelInPmmlFile \    --model-version=0.0.1 \    --predictor-column=model:modelpredictor \    --result-column=model:modelresult \    --result-record-name=MyModelResult \    --model-container=/path/to/write/model-container.json

3. Create an empty jar (can't deploy right now without a jar file and JpmmlScoreFunction lives in the kiji-scoring jar):touch empty-filejar cf /path/to/empty-jar.jar empty-filerm empty-file

4. Deploy the generated model container:# The deps-resolver flag must be specified here even though# it won't get used (KIJIREPO-47).kiji model-repo deploy nameOfModelInPmmlFile /path/to/empty-jar.jar \    --kiji=kiji://my/model/repo/instance \    --deps-resolver=maven \    --production-ready=true \    --model-container=/path/to/written/model-container.json \    --message="Initial deployment of JPMML based model."

5. Attach the JpmmlScoreFunction to the result column for the model:# Freshness policy may need to be different depending on model.kiji model-repo fresh-model \    kiji://my/model/repo/instance \    nameOfModelInPmmlFile \    ax

After running through these steps, your model should be running/active.

