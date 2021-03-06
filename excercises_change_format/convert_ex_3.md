Import products table from mysql 
* as text file to hdfs convert_ex3/import/text. Fields should be terminated by a tab character (“\t”) character and lines should be terminated by new line character (“\n”).
* as avro file to hdfs convert_ex3/import/avro

Convert avro data at convert_ex3/import/avro :
* as parquet file with snappy compression, save to convert_ex3/parquet-snappy
* as text file with gzip compression, save to /convert_ex3/text-gzip
* as text file with snapp compression, save to /convert_ex3/text-snappy (snappy did not work)
* as sequence file at /convert_ex3/sequence

Convert_ex3/parquet-snappy to 
* parquet file with no compression at convert_ex3/parquet-nocompress
* avro file with snappy compression at convert_ex3/avro-snappycompress


```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username retail_dba \
--password cloudera \
--table products \
--as-avrodatafile \
--target-dir /user/cloudera/convert_ex3/import/avro
```


```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username retail_dba \
--password cloudera \
--table products \
--as-textfile \
--target-dir /user/cloudera/products \
--fields-terminated-by '|'
--lines-terminated-by "\n"
```

Save as parquet file with snappy compression:
```
#import avro.schema
#from avro.datafile import DataFileReader, DataFileWriter
from pyspark.sql import SQLContext
sqlContext = SQLContext(sc)
products_avro = sqlContext.read.format("com.databricks.spark.avro").load("/user/cloudera/convert_ex3/import/avro")
products_avro.show(20)

sqlContext.setConf("spark.sql.parquet.compression.codec", "snappy")
products_avro.write.parquet("/user/cloudera/convert_ex3/parquet-snappy-compress")

```

Save as text file with gzip compression:
```
products_avro.repartition(1).rdd\
.map(lambda x: str(x[0])+","+str(x[1])+","+x[2]+","+x[3]+","+str(x[4])+str(x[5]))\
.saveAsTextFile("/user/cloudera/convert_ex3/text_gzip", compressionCodecClass="org.apache.hadoop.io.compress.GzipCodec")
```

Save as text file with snappy compression: gave error
```
codec = "org.apache.hadoop.io.compress.SnappyCodec"
products.repartition(1).rdd.map(lambda x: str(x[0]) + ',' + str(x[1]) + ',' +  x[2] + ',' + x[3] + ',' +  str(x[4]) + ',' +  x[5]) \
.saveAsTextFile("/user/cloudera/convert_ex3/text-snappy", compressionCodecClass=codec)
```

Save as sequenceFile:
```
products.repartition(1).rdd\
.map(lambda x:  (str(x[0]), str(x[0])+"\t"+str(x[1])+"\t"+x[2]+"\t"+x[3]+"\t"+str(x[4])+str(x[5])))\
.saveAsSequenceFile("/user/cloudera/convert_ex3/sequence2")

products.rdd\
.map(lambda x:  (str(x[0]), str(x[0])+"\t"+str(x[1])+"\t"+x[2]+"\t"+x[3]+"\t"+str(x[4])+str(x[5])))\
.saveAsSequenceFile("/user/cloudera/convert_ex3/sequence3")
```




Convert_ex3/parquet-snappy to 
* parquet file with no compression at convert_ex3/parquet-nocompress
* avro file with snappy compression at convert_ex3/avro-snappycompress
```
from pyspark.sql import SQLContext
sqlContext = SQLContext(sc)
products = sqlContext.read.parquet('/user/cloudera/convert_ex3/parquet-snappy-compress')

sqlContext.setConf("spark.sql.parquet.compression.codec", "uncompressed")
products.write.parquet('/user/cloudera/convert_ex3/parquet-uncompressed')

```


```
sqlContext.setConf("spark.sql.avro.compression.codec","snappy")
products.repartition(4).write.format("com.databricks.spark.avro").save("/user/cloudera/convert_ex3/avro-snappycompress")
```