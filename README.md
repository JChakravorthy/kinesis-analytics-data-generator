# using-kinesis-analytics

Kinesis analytics is a way to analyze near real-time streams of data in a very SQL-like way.
Anyone who knows regular SQL from using a database like Oracle say will have no trouble getting to grips with the 
slightly unusual windowing features of Kinesis Analytics (or KA as I'll call it from now on). Testing KA apps just got a little bit easier
with the intoduction by Amazon recently of the Kinesis Data Generator utility. This utility which is web-based allows you define a 
template of test data in pretty much any format you like and send up to 1000's of records per second into your chosen Kinesis Stream or 
Fire Hose. As it's web-based and password protected you need to set up an AWS coginto user. AWS make this easy by using a pre-defined 
lambda and cloudformation template. You can set up a cognito user by following this link 
https://awslabs.github.io/amazon-kinesis-data-generator/web/help.html#configAccount

Once you've uploaded a cognito JSON set-up file you will be presented with a screen where you only need to input a stack name and 
your chosen username and password. You can choose all defaults and click NEXT to the rest of the screens/prompts and once the 
process has been completed you will be presented with a URL link which will take you to the Kinesis Data Generator web page. 
From here you just need to type in your cognito login credentials and you can start to create a data feed schema and start 
sending a ton of test data to your chosen stream. In my own case I chose to create the following schema to produce a set of 
dummy temperature data:

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
was coming through to the stream. After this you need to set up your KA streaming job. You give the stream a  name, and a source 
(this is the Firehose stream you have already set up and is receiving your dummy sensor data). After that you choose the Reatime 
Analytic TAB and ensure that source data is streaming into KA. If it is, you then move on to defining 
some KA SQL to run against your data input and optionally a source such as S3 to store the results of your analytics. My SQL was 
really simple, just a sliding windowing function that displays the average temperature per sensorID over the preceding moving 10 
second window.

KA rocks!

