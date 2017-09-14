# AWS Meetup Aurora

This repository is a demo of how to deploy an Aurora DB cluster
with one replica.

## Dataset

In our demo, we will use open data to ingest the database with ~5 millions rows.

Provide by amazon at: https://aws.amazon.com/public-datasets/
Another: http://snap.stanford.edu/data/

The detail of dataset headers can be found at http://data.gdeltproject.org/documentation/GDELT-Data_Format_Codebook.pdf

## Requirement

You need an AWS account with administrator console access.
The demo is deployed through AWS Cloudformation service.

If you are not familiar with this service, take a look at http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html

## Get Started!

 1. Go to the aws console on *AWS Cloudformation dashboard*
 2. Create new stack and select `Upload a template to Amazon S3` with file in `cloudformation/aurora-demo-one.yml`
 3. Fill the form of your choice, a Stack name like `aurora-demo-one`

 > The Username will create an IAM USER, it must not already exists.
 > Password must be strong, the instance will be public accessible to anyone!
 > Be sure to fill VPC id with a public one it should be like `vpc-1234a12b`
 
 4. Hit next button, twice, and make sure the checkbox of **Capabilities** is checked, at the end just before **create** button
    The word next right to the checkbox is `I acknowledge that AWS CloudFormation might create IAM resources with custom names.`
    
 5. Take a coffee, the cloudformation will do all the job for you.
    it will take only **15min** in order to have Cluster ready.
    
 6. Go to IAM console and generate API key on the Username built by the cloudformation
    configure a profile named `aws-meetup-aurora` in your terminal.
    
 7. Run the `bin/ingestion $MY_S3_BUCKET` script in order to generate manifest file with GDELT OpenData and upload it on AWS S3 Bucket.
 
 > the bucket reference by **$MY_S3_BUCKET** is generate by cloudformation and can be find in AWS S3 console or in Resources tab on Cloudformation stack panel.
    
 8. Go in `Clusters`on  RDS Dasboard in AWS Console, and use the `Cluster Endpoint` to connect on the database.
    **the database name is awsMeetupAurora**
 
```bash
$ mysql -u $MasterUserName -p -h $DbClusterWriterEndpoint
mysql> use awsMeetupAurora;
mysql> GRANT LOAD FROM S3 ON *.* TO "$MasterUserName@%";
       GRANT SELECT INTO S3 ON *.* TO "$MasterUserName@%";
```

 9. Create the SQL table by copy paste the SQL in `schemas/gdelt.sql`
 10. Import GDELT S3 file by using manifest file, execute this on the SQL console
 
 ```sql
 LOAD DATA FROM S3 MANIFEST '$S3_MANIFEST_FILE'
 INTO TABLE events
     FIELDS TERMINATED BY '\t'
     LINES TERMINATED BY '\n'
     (GLOBALEVENTID,SQLDATE,MonthYear,Year,FractionDate,Actor1Code,Actor1Name,Actor1CountryCode,Actor1KnownGroupCode,Actor1EthnicCode,Actor1Religion1Code,Actor1Religion2Code,Actor1Type1Code,Actor1Type2Code,Actor1Type3Code,Actor2Code,Actor2Name,Actor2CountryCode,Actor2KnownGroupCode,Actor2EthnicCode,Actor2Religion1Code,Actor2Religion2Code,Actor2Type1Code,Actor2Type2Code,Actor2Type3Code,IsRootEvent,EventCode,EventBaseCode,EventRootCode,QuadClass,GoldsteinScale,NumMentions,NumSources,NumArticles,AvgTone,Actor1Geo_Type,Actor1Geo_FullName,Actor1Geo_CountryCode,Actor1Geo_ADM1Code,Actor1Geo_Lat,Actor1Geo_Long,Actor1Geo_FeatureID,Actor2Geo_Type,Actor2Geo_FullName,Actor2Geo_CountryCode,Actor2Geo_ADM1Code,Actor2Geo_Lat,Actor2Geo_Long,Actor2Geo_FeatureID,ActionGeo_Type,ActionGeo_FullName,ActionGeo_CountryCode,ActionGeo_ADM1Code,ActionGeo_Lat,ActionGeo_Long,ActionGeo_FeatureID,DATEADDED,SOURCEURL);
 ```
 
 > Manifest import operation should take 10min
 
### Cloudformation Resources details

 **Aurora S3 Bucket**
 
 **Aurora Management User**
 
 **RdsMonitoringRole**
 
 **AuroraClusterRole**
 
 **SgRdsAllAllow**
 
 **RDSClusterParameterGroup**
 
 **RDSCluster**
 
 **DatabaseRdsMaster**
 
 **DatabaseRdsReplica**

