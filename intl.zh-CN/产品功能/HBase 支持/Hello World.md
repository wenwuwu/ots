# Hello World {#concept_50163_zh .concept}

本节描述如何使用 Table Store HBase Client 实现一个简单的 Hello World 程序，主要包括下列操作：

-   配置依赖
-   连接表格存储
-   创建表
-   写数据
-   读数据
-   扫描数据
-   删表

## 代码位置 {#section_hd1_1kn_cfb .section}

当前示例程序使用了 HBase API 访问表格存储服务，完整的示例程序位于 Github 的[hbase](https://github.com/aliyun/aliyun-tablestore-hbase-client) 项目中，目录位置是 src/test/java/samples/HelloWorld.java。

## 使用 HBase API { .section}

-   配置项目依赖

    Maven 的依赖配置如下：

    ```language-xml
       <dependencies>
            <dependency>
                <groupId>com.aliyun.openservices</groupId>
                <artifactId>tablestore-hbase-client</artifactId>
                <version>1.2.0</version>
            </dependency>
        </dependencies>
    
    ```

    更高级的配置请参考[从 HBase 迁移到表格存储](intl.zh-CN/产品功能/HBase 支持/从 HBase 迁移到表格存储.md#)。

-   配置文件

    hbase-site.xml 中增加下列配置项：

    ```language-xml
    <configuration>
        <property>
            <name>hbase.client.connection.impl</name>
            <value>com.alicloud.tablestore.hbase.TablestoreConnection</value>
        </property>
        <property>
            <name>tablestore.client.endpoint</name>
            <value>endpoint</value>
        </property>
        <property>
            <name>tablestore.client.instancename</name>
            <value>instance_name</value>
        </property>
        <property>
            <name>tablestore.client.accesskeyid</name>
            <value>access_key_id</value>
        </property>
        <property>
            <name>tablestore.client.accesskeysecret</name>
            <value>access_key_secret</value>
        </property>
        <property>
            <name>hbase.client.tablestore.family</name>
            <value>f1</value>
        </property>
        <property>
            <name>hbase.client.tablestore.table</name>
            <value>ots_adaptor</value>
        </property>
    </configuration>
    
    ```

    更高级的配置请参考[从 HBase 迁移到表格存储](intl.zh-CN/产品功能/HBase 支持/从 HBase 迁移到表格存储.md#)。

-   连接表格存储

    通过创建一个 TableStoreConnection 对象来链接表格存储服务。

    ```language-java
    		Configuration config = HBaseConfiguration.create();
    		
    		// 创建一个 Tablestore Connection
    		Connection connection = ConnectionFactory.createConnection(config);
    		
    		// Admin 负责创建、管理、删除等
            Admin admin = connection.getAdmin();
    
    ```

-   创建表

    通过指定表名创建一张表，MaxVersion 和 TimeToLive 使用默认值。

    ```language-java
    		// 创建一个 HTableDescriptor，只有一个列族
    		HTableDescriptor descriptor = new HTableDescriptor(TableName.valueOf(TABLE_NAME));
    		
    		// 创建一个列族，MaxVersion 和 TimeToLive使用默认值，MaxVersion 默认值是 1，TimeToLive 默认值是 Integer.INF_MAX
    		descriptor.addFamily(new HColumnDescriptor(COLUMN_FAMILY_NAME));
    		
    		// 通过 Admin 的 createTable 接口创建表
    		System.out.println("Create table " + descriptor.getNameAsString());
    		admin.createTable(descriptor);
    
    ```

-   写数据

    写入一行数据到表格存储。

    ```language-java
    		// 创建一个 TablestoreTable，用于单个表上的读写更新删除等操作
    		Table table = connection.getTable(TableName.valueOf(TABLE_NAME));
    		
    		// 创建一个 Put 对象，主键是 row_1
    		System.out.println("Write one row to the table");
            Put put = new Put(ROW_KEY);
    		
    		// 增加一列，表格存储只支持单列族，列族名称在 hbase-site.xml 中配置，如果没有配置则默认是“f”，所以写入数据时 COLUMN_FAMILY_NAME 可以是空值
    		put.addColumn(COLUMN_FAMILY_NAME, COLUMN_NAME, COLUMN_VALUE);
    		
    		// 执行 Table 的 put 操作，使用 HBase API 将这一行数据写入表格存储
            table.put(put);
    
    ```

-   读数据

    读取指定行的数据。

    ```language-java
    		// 创建一个 Get 对象，读取主键为 ROW_KEY 的行
    		Result getResult = table.get(new Get(ROW_KEY));
    		Result result = table.get(get);
    		
    		// 打印结果
    		String value = Bytes.toString(getResult.getValue(COLUMN_FAMILY_NAME, COLUMN_NAME));
    		System.out.println("Get one row by row key");
    		System.out.printf("\t%s = %s\n", Bytes.toString(ROW_KEY), value);
    
    ```

-   扫描数据

    范围读取数据。

    ```language-java
    	扫描全表所有行数据
    	System.out.println("Scan for all rows:");
    	Scan scan = new Scan();
    
    	ResultScanner scanner = table.getScanner(scan);
    	
    	// 循环打印结果
    	for (Result row : scanner) {
    		byte[] valueBytes = row.getValue(COLUMN_FAMILY_NAME, COLUMN_NAME);
    		System.out.println('\t' + Bytes.toString(valueBytes));
    	}
    
    ```

-   删表

    使用 Admin API 删除一张表。

    ```language-java
    		print("Delete the table");
    		admin.disableTable(table.getName());
    		admin.deleteTable(table.getName());
    
    ```


## 完整代码 { .section}

```language-java
package samples;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HelloWorld {

    private static final byte[] TABLE_NAME = Bytes.toBytes("HelloTablestore");
    private static final byte[] ROW_KEY = Bytes.toBytes("row_1");
    private static final byte[] COLUMN_FAMILY_NAME = Bytes.toBytes("f");
    private static final byte[] COLUMN_NAME = Bytes.toBytes("col_1");
    private static final byte[] COLUMN_VALUE = Bytes.toBytes("col_value");

    public static void main(String[] args) {
        helloWorld();
    }

    private static void helloWorld() {

        try  {
            Configuration config = HBaseConfiguration.create();
            Connection connection = ConnectionFactory.createConnection(config);
            Admin admin = connection.getAdmin();

            HTableDescriptor descriptor = new HTableDescriptor(TableName.valueOf(TABLE_NAME));
            descriptor.addFamily(new HColumnDescriptor(COLUMN_FAMILY_NAME));

            System.out.println("Create table " + descriptor.getNameAsString());
            admin.createTable(descriptor);

            Table table = connection.getTable(TableName.valueOf(TABLE_NAME));

            System.out.println("Write one row to the table");
            Put put = new Put(ROW_KEY);
            put.addColumn(COLUMN_FAMILY_NAME, COLUMN_NAME, COLUMN_VALUE);
            table.put(put);

            Result getResult = table.get(new Get(ROW_KEY));
            String value = Bytes.toString(getResult.getValue(COLUMN_FAMILY_NAME, COLUMN_NAME));
            System.out.println("Get a one row by row key");
            System.out.printf("\t%s = %s\n", Bytes.toString(ROW_KEY), value);

            Scan scan = new Scan();

            System.out.println("Scan for all rows:");
            ResultScanner scanner = table.getScanner(scan);
            for (Result row : scanner) {
                byte[] valueBytes = row.getValue(COLUMN_FAMILY_NAME, COLUMN_NAME);
                System.out.println('\t' + Bytes.toString(valueBytes));
            }

            System.out.println("Delete the table");
            admin.disableTable(table.getName());
            admin.deleteTable(table.getName());

            table.close();
            admin.close();
            connection.close();
        } catch (IOException e) {
            System.err.println("Exception while running HelloTablestore: " + e.toString());
            System.exit(1);
        }
    }
}

```

