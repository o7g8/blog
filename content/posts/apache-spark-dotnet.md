+++
title = "Running Apache Spark/.NET and getting data from Oracle DB"
date = 2019-05-06T16:00:05+02:00
draft = false
tags = []
categories = ["Windows", ".NET", "Spark", "Oracle"]
+++

## Running Apache Spark/.NET on Windows

MS has announced support of [.NET for Apache Spark](https://dotnet.microsoft.com/apps/data/spark).

The main Spark/.NET documentation is [here](https://github.com/dotnet/spark/blob/master/README.md).

In the article I will highlight steps necessary to get the .NET for Apache Spark running in Windows.

Download and install all components, mentioned in the [README.md](https://github.com/dotnet/spark/blob/master/README.md), initialize the necessary environment variables and create the `HelloSpark` program.

Get `winutils.exe` as described in the step 3 of [Spark 2: How to install it on Windows in 5 steps](https://medium.com/@dvainrub/how-to-install-apache-spark-2-x-in-your-pc-e2047246ffc3)) and copy it into your Spark bin-directory.

Define environment variables `SPARK_HOME` and `HADOOP_HOME` as described in the article above.

Start your example in the `HelloSpark` deployment directory (use your version of `microsoft-spark-XXX.jar`):

```cmd

spark-submit --class org.apache.spark.deploy.DotnetRunner --master local microsoft-spark-2.4.x-0.2.0.jar dotnet HelloSpark.dll

```

You may expect a warning `WARN SparkEnv: Exception while deleting Spark temp dir ..` in your output, which is most likely related to Spark issues [SPARK-12216](https://issues.apache.org/jira/browse/SPARK-12216) and [SPARK-8333](https://issues.apache.org/jira/browse/SPARK-8333). I didn't find how to get rid of the warning on Windows.

## Get data from Oracle DB into Spark/.NET

Get `ojdbc6.jar` (`ojdbc8.jar` works too) from [Oracle Database 11.2.0.4 JDBC Driver & UCP Downloads](https://www.oracle.com/technetwork/apps-tech/jdbc-112010-090769.html).

You can either copy the `ojdbc8.jar` into `$SPARK_HOME/jars` or you can specify path to the jar as an argument of `spark-submit` (see below).

Here is the C# code accessing data in Oracle:

```csharp
using Microsoft.Spark.Sql;

namespace HelloSpark {
    class Program {
        static void Main(string[] args) {
            var spark = SparkSession.Builder().GetOrCreate();
            var df = spark
                .Read()
                .Format("jdbc")
                .Option("url", "jdbc:oracle:thin:@<db-ip-addr>:<port>:<SID>")
                .Option("dbtable", "<table>")
                .Option("user", "<user>")
                .Option("password", "<pwd>")
                .Option("driver", "oracle.jdbc.driver.OracleDriver")
                .Load();
            df.PrintSchema();
            df.Show();
        }
    }
}
```

If you have copied  `ojdbc8.jar` into `$SPARK_HOME/jars` then run the test program as:

```cmd
spark-submit --class org.apache.spark.deploy.DotnetRunner --master local microsoft-spark-2.4.x-0.2.0.jar dotnet HelloSpark.dll
```

Otherwise add `--driver-class-path` argument:

```cmd
spark-submit --driver-class-path <path-to-ojdbc>\ojdbc8.jar --class org.apache.spark.deploy.DotnetRunner --master local microsoft-spark-2.4.x-0.2.0.jar dotnet HelloSpark.dll
```

In both cases you should see the table schema and data dumped to console.