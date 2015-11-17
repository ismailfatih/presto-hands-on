# presto-hands-on

Welcome! This repo contains instructions for setting up Presto on an Amazon EMR cluster, loading some data and playing around with some queries. Feel free to open issues of pull requests if something with the instructions is off.

# Pre-requisites

We assume that you have provisioned a 3 node EMR cluster that you have access to via SSH and that all nodes in the cluster have access to the Internet.

For the Boston Presto hands on meetup, check out this [gist](https://gist.github.com/petroav/1ef757d1673bc6a7436e) for getting your credentials to an EMR cluster.

# Setting up the environment

1. Upload default.pem (who's contents are included in the gist linked above) key to your cluster, preferrably in `/home/ec2-user/default.pem`.
2. Log into your cluster as the `ec2-user` user. That is the user we'll be using throughout the tutorial because passwordless `ssh` for root is not setup by default.
3. We'll use Teradata's `presto-admin` CLI tool for the administrative tasks needed to setup our environment. You should download and set it up with the following instructions on the master node:
```
wget https://s3-us-west-2.amazonaws.com/presto-admin/meetup-2015-11/prestoadmin-1.2-offline.tar.bz2
tar -jxf prestoadmin-1.2-offline.tar.bz2
cd prestoadmin/
sudo ./install-prestoadmin.sh
sudo ./presto-admin --help
```
4. Download JRE8u66 from Oracle on your master node:
```
wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u65-b17/jre-8u65-linux-x64.rpm
```
5. Install the JRE using `presto-admin`:
```
# Fill out coordinator and slave internal IPs as posted in the gist and set the user to ec2-user
vi /etc/opt/prestoadmin/config.json
sudo mkdir /root/.ssh
sudo cp /home/ec2-user/default.pem /root/.ssh/id_rsa
cd /home/ec2-user/prestoadmin/
sudo ./presto-admin package install /home/ec2-user/jre-8u65-linux-x64.rpm
java -version
```

# Setting up the data

1. Download the data in the `/mnt` directory by running the following commands:
```
cd /mnt
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000000
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000001
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000002
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000003
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000004
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000005
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000006
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000007
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000008
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000009
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000010
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000011
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000012
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000013
```
2. Upload the newly downloaded data to HDFS:
```
cd /mnt
sudo -u hdfs hdfs dfs -mkdir /ontime
sudo -u hdfs hdfs dfs -copyFromLocal /mnt/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-* /ontime
```
3. Create the `flights` table in Hive and point it to the data you uploaded to HDFS.
```
sudo -u hdfs hive
CREATE EXTERNAL TABLE flights (
    Year INT,
    Quarter INT,
    Month INT,
    DayofMonth INT,
    DayOfWeek INT,
    FlightDate STRING,
    UniqueCarrier STRING,
    AirlineID INT,
    Carrier STRING,
    TailNum STRING,
    FlightNum INT,
    OriginAirportID INT,
    OriginAirportSeqID INT,
    OriginCityMarketID INT,
    Origin STRING,
    OriginCityName STRING,
    OriginState STRING,
    OriginStateFips INT,
    OriginStateName STRING,
    OriginWac INT,
    DestAirportID INT,
    DestAirportSeqID INT,
    DestCityMarketID INT,
    Dest STRING,
    DestCityName STRING,
    DestState STRING,
    DestStateFips INT,
    DestStateName STRING,
    DestWac INT,
    CRSDepTime INT,
    DepTime INT,
    DepDelay INT,
    DepDelayMinutes INT,
    DepDel15 INT,
    DepartureDelayGroups INT,
    DepTimeBlk STRING,
    TaxiOut INT,
    WheelsOff INT,
    WheelsOn INT,
    TaxiIn INT,
    CRSArrTime INT,
    ArrTime INT,
    ArrDelay INT,
    ArrDelayMinutes INT,
    ArrDel15 INT,
    ArrivalDelayGroups INT,
    ArrTimeBlk STRING,
    Cancelled TINYINT,
    CancellationCode STRING,
    Diverted TINYINT,
    CRSElapsedTime INT,
    ActualElapsedTime INT,
    AirTime INT,
    Flights INT,
    Distance INT,
    DistanceGroup INT,
    CarrierDelay INT,
    WeatherDelay INT,
    NASDelay INT,
    SecurityDelay INT,
    LateAircraftDelay INT,
    FirstDepTime INT,
    TotalAddGTime INT,
    LongestAddGTime INT,
    DivAirportLandings INT,
    DivReachedDest INT,
    DivActualElapsedTime INT,
    DivArrDelay INT,
    DivDistance INT,
    Div1Airport STRING,
    Div1AirportID INT,
    Div1AirportSeqID INT,
    Div1WheelsOn INT,
    Div1TotalGTime INT,
    Div1LongestGTime INT,
    Div1WheelsOff INT,
    Div1TailNum STRING,
    Div2Airport STRING,
    Div2AirportID INT,
    Div2AirportSeqID INT,
    Div2WheelsOn INT,
    Div2TotalGTime INT,
    Div2LongestGTime INT,
    Div2WheelsOff INT,
    Div2TailNum STRING,
    Div3Airport STRING,
    Div3AirportID INT,
    Div3AirportSeqID INT,
    Div3WheelsOn INT,
    Div3TotalGTime INT,
    Div3LongestGTime INT,
    Div3WheelsOff INT,
    Div3TailNum STRING,
    Div4Airport STRING,
    Div4AirportID INT,
    Div4AirportSeqID INT,
    Div4WheelsOn INT,
    Div4TotalGTime INT,
    Div4LongestGTime INT,
    Div4WheelsOff INT,
    Div4TailNum STRING,
    Div5Airport STRING,
    Div5AirportID INT,
    Div5AirportSeqID INT,
    Div5WheelsOn INT,
    Div5TotalGTime INT,
    Div5LongestGTime INT,
    Div5WheelsOff INT,
    Div5TailNum STRING  
)
STORED AS ORC
LOCATION '/ontime'
tblproperties ("orc.compress"="SNAPPY");

show tables;
select count(*) from flights;
```

# Install and configure Presto

1. Download and install the RPM:
```
wget https://s3-us-west-2.amazonaws.com/presto-admin/meetup-2015-11/presto-server-rpm-0.115t-1.x86_64.rpm
sudo ./presto-admin package install presto-server-rpm-0.115t-1.x86_64.rpm
rm -f presto-server-rpm-0.115t-1.x86_64.rpm
```
2. Get the Presto CLI:
```
wget https://s3-us-west-2.amazonaws.com/presto-admin/meetup-2015-11/presto-cli-0.115t-executable.jar
sudo mv presto-cli-0.115t-executable.jar /usr/lib/presto/bin/presto
chmod +x /usr/lib/presto/bin/presto
```
3. Configure Hive connector:
```
sudo mkdir /etc/opt/prestoadmin/connectors
sudo touch /etc/opt/prestoadmin/connectors/hive.properties
sudo bash -c 'echo "
connector.name=hive-cdh5
hive.metastore.uri=thrift://<IP OF MASTER HOST>:9083
hive.allow-drop-table=true
hive.allow-rename-table=true
" > /etc/opt/prestoadmin/connectors/hive.properties'
sudo ./presto-admin connector add hive
sudo ./presto-admin server start
```