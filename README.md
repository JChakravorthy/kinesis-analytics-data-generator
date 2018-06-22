# using-kinesis-analytics

Kinesis analytics is a way to analyze near real-time streams of data in a very SQL-like way.
Anyone who knows regular SQL from using a database like Oracle say will have no trouble getting to grips with the 
slightly unusual windowing features of Kinesis Analytics (or KA as I'll call it from now on). 

Testing KA apps just got a little bit easier with the introduction by Amazon recently of the Kinesis Data Generator 
utility. This utility which is web-based allows you define a template for test data in pretty much any format you 
like - structured like CSV for example or completely unstructued -and will send 1000's of test records per second into your 
chosen Kinesis Stream or Fire Hose that correspond to the format defined in your schema. As it's web-based and password protected you need to set up an AWS coginto user. Fortunately AWS make this really easy by using a pre-defined lambda and cloudformation template. 

You can set up a cognito user by following this link 
https://awslabs.github.io/amazon-kinesis-data-generator/web/help.html#configAccount

Once you've uploaded a cognito JSON set-up file you will be presented with a screen where you only need to input a stack name and 
your chosen username and password. You can choose all defaults and click NEXT to the rest of the screens/prompts and once the 
process has been completed you will be presented with a URL link which will take you to the Kinesis Data Generator web page itself. 
From here you just need to type in your cognito login credentials and you can start to create a data feed schema and start 
sending a ton of test data to your chosen Kinesis stream. The schema defintion can range from the very simple to  more complex 
examples like apache log entries. You can even specify a schema where some data points can only take a limited or random set
of values. In my own case I chose to create the schema shown below to produce a set of dummy temperature data. This schema defines 
a sensorId field that is a random value between 1 and 50, the currentTemperature field must lie between 10 and 150 and the 
status field can only take one of three values.


```
{
    "sensorId": {{random.number(50)}},
    "currentTemperature": {{random.number(
        {
            "min":10,
            "max":150
        }
    )}},
    "status": "{{random.arrayElement(
        ["OK","FAIL","WARN"]
    )}}"
}
```

I connected this data to a Kinesis Firehose stream, set the throughput to 100 records per second and set an S3 bucket to store the 
stream  output as a back-up ( you can send the output to AWS ElasticSearch, Splunk or Redshift too). I then checked that the data 
was coming through to the stream. After this you need to set up your KA streaming job. You give the stream a name, and an source 
(the Firehose stream that is receiving your dummy test data). After that you choose the Reatime Analytic TAB and ensure that the 
source data is streaming into KA. KA will infer the data types of the fields from its stream input. If all is OK, you then move 
on to defining the SQL that you want to run against your input data. My SQL was really simple, just a sliding windowing function 
that displays the average temperature per sensorID over the preceding moving 10 second window. 

KA SQL is based on the 2008 SQL standard but some parts of it are a little unusual in that you have to define two 
so-called in-application streams at the top of your SQL, a STREAM (analagous to a database table) and a PUMP which is analagous 
to a continuous INSERT statement into the STREAM. For aggregations you have to use windowing functions which are also a little 
unusual but basically these allow you to do sliding or tumbling windows based on rowcounts and/or time intervals. 

One final thing of note is that its possible for a KA application to aslo import a file as reference data. For example 
you could create a CSV file in an S3 bucket that mapped the sensorId in the schema definition to an actual physical location and 
inner join the streamimg data on the reference data to bring out the location corresponding to the sensorId. For example:

sensorId,Location
1, New York
2, London,
3, Hong Kong
4, Singapore
5, Edinburgh

To quickly recap, to use Kiensis Analtyics and the data generator you need to do the following:

1) Create a Kinesis FireHose to send your data into 
2) Create a cognito user
3) Login to the Kinesis Data Generator webpage, create a data schema and start sending data to your chosen Kinesis Firehose
4) Create a Kinesis Analytics job, set the source as your FireHose and set up a SQL to analyze your data


Kinesis rocks!

