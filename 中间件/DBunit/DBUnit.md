# DBUnit
---
## 简介

DbUnit 是一个针对数据库驱动项目的 JUnit 扩展，除其他外，它将你的数据库在测试运行之间放入一个已知的状态。这是一个很好的方法来避免无数的问题，当一个测试用例破坏了数据库并导致后续测试失败或加剧破坏时，就会发生这种情况。

DbUnit 有能力将你的数据库数据导出和导入到 XML 数据集中。从 2.0 版本开始，当以流模式使用时，DbUnit 还可以处理非常大的数据集。DbUnit 还可以帮助你验证你的数据库数据是否符合预期的数值集。

## 快速开始

下面搭建一个简单的测试 demo，只需要执行下面 5 步即可快速使用 DBUnit。

1. 用 DBTestCase 子类设置数据库
2. 用你自己的 TestCase 子类设置数据库
3. 数据库数据验证
4. 数据文件加载器
5. DbUnit Ant 任务和 Canoo WebTest

### Step 1 创建你的数据库文件

你的测试需要一些数据来工作。这意味着你必须创建一个数据集。在大多数情况下，你将与 xml 数据集一起工作。你可以从头开始手动创建一个扁平的 XML 数据集，或者通过从你的数据库中导出一些数据来创建一个。

### Step 2 继承 DBTestCase 这个 Class

现在你需要创建一个测试类。使用 Dbunit 的最简单方法是使你的测试类继承 DBTestCase 类。DBTestCase 扩展了 JUnit TestCase类。需要实现一个模板方法：getDataSet() 来返回你在步骤1中创建的数据集。DBTestCase 依靠一个 IDatabaseTester 来完成它的工作，默认配置使用 PropertiesBasedJdbcDatabaseTester，它在系统属性中为 DriverManager 定位配置。配置它的最简单方法是在你的测试类的构造函数中。你可以通过覆盖 getDatabaseTester() 来修改这一行为。

使用其他 3 个提供的 IDatabaseTester 实现中的一个或你自己的。你也可以使用下表中描述的 DBTestCase 的其他子类。

| Class                     |                                           Description                                           |
| :------------------------ | :---------------------------------------------------------------------------------------------: |
| JdbcBasedDBTestCase       |       uses a DriverManager to create connections (with the aid of a JdbcDatabaseTester).        |
| DataSourceBasedDBTestCase | uses a javax.sql.DataSource to create connections (with the aid of a DataSourceDatabaseTester). |
| JndiBasedDBTestCase       |    uses a javax.sql.DataSource located through JNDI (with the aid of a JndiDatabaseTester).     |

下面是代码示例

```java
public class SampleTest extends DBTestCase {
    public SampleTest(String name) {
        super( name );
        System.setProperty( PropertiesBasedJdbcDatabaseTester.DBUNIT_DRIVER_CLASS, "org.hsqldb.jdbcDriver" );
        System.setProperty( PropertiesBasedJdbcDatabaseTester.DBUNIT_CONNECTION_URL, "jdbc:hsqldb:sample" );
        System.setProperty( PropertiesBasedJdbcDatabaseTester.DBUNIT_USERNAME, "sa" );
        System.setProperty( PropertiesBasedJdbcDatabaseTester.DBUNIT_PASSWORD, "" );
    }

    protected IDataSet getDataSet() throws Exception {
        return new FlatXmlDataSetBuilder().build(new FileInputStream("dataset.xml"));
    }
}
```

### Step 3 （可选）实现 getSetUpOperation() 和 getTearDownOperation() 方法

默认情况下，Dbunit 在执行每个测试之前会执行一个 CLEAN_INSERT 操作，之后不执行任何清理操作。你可以通过覆盖getSetUpOperation() 和 getTearDownOperation() 来修改这一行为。

下面的例子演示了你如何轻松地覆盖测试前或测试后执行的操作。

```java
public class SampleTest extends DBTestCase {
    ...
    protected DatabaseOperation getSetUpOperation() throws Exception {
        return DatabaseOperation.REFRESH;
    }

    protected DatabaseOperation getTearDownOperation() throws Exception {
        return DatabaseOperation.NONE;
    }
    ...
}
```

### Step 4 （可选）实现 setUpDatabaseConfig(DatabaseConfig config) 方法

使用它来改变 dbunit DatabaseConfig 的一些配置设置。

下面的例子演示了你如何轻松地覆盖这个方法。

```java
public class SampleTest extends DBTestCase {
    ...
    /**
     * Override method to set custom properties/features
     */
    protected void setUpDatabaseConfig(DatabaseConfig config) {
        config.setProperty(DatabaseConfig.PROPERTY_BATCH_SIZE, new Integer(97));
        config.setFeature(DatabaseConfig.FEATURE_BATCHED_STATEMENTS, true);
    }
    ...
}
```

### Step 5 实现你的 testXXX() 方法

像你通常使用 JUnit 那样实现你的测试方法。现在你的数据库在每个测试方法之前被初始化，在每个测试方法之后被清理，根据你在前面步骤中的做法。

## 用你自己的 TestCase 子类设置数据库

为了使用 Dbunit，你不需要扩展 DBTestCase 类。你可以覆盖标准的 JUnit setUp() 方法并在你的数据库上执行所需的操作。如果你需要进行清理，在 teledown() 中做类似的事情。

```java
public class SampleTest extends TestCase {

    public SampleTest(String name) {
        super(name);
    }

    protected void setUp() throws Exception {
        super.setUp();
        // initialize your database connection here
        IDatabaseConnection connection = null;
        // ...
        // initialize your dataset here
        IDataSet dataSet = null;
        // ...
        try1 {
            DatabaseOperation.CLEAN_INSERT.execute(connection, dataSet);
        }
        finally {
            connection.close();
        }
    }
    ...
}
```

从 2.2 版本开始，你可以使用新的 IDatabaseTester 来完成同样的工作。正如在上一个主题中所解释的，DBTestCase 在内部使用一个 IDatabaseTester 来完成它的工作；你的测试类也可以使用这个功能来操作 DataSets。目前有 4 种方便的实现方式。

| Class                             |                                                              Description                                                              |
| :-------------------------------- | :-----------------------------------------------------------------------------------------------------------------------------------: |
| JdbcDatabaseTester                |                                              uses a DriverManager to create connections.                                              |
| PropertiesBasedJdbcDatabaseTester | also uses DriverManager, but the configuration is taken from system properties.This is the default implementation used by DBTestCase. |
| DataSourceDatabaseTester          |                                          uses a javax.sql.DataSource to create connections.                                           |
| JndiDatabaseTester                |                                           uses a javax.sql.DataSource located through JNDI.                                           |

你也可以提供你自己的 IDatabaseTester 实现。我们推荐使用 AbstractDatabaseTester 作为起点。

```java
public class SampleTest extends TestCase {
    private IDatabaseTester databaseTester;

    public SampleTest(String name) {
        super(name);
    }

    protected void setUp() throws Exception {
        databaseTester = new JdbcDatabaseTester("org.hsqldb.jdbcDriver",
            "jdbc:hsqldb:sample", "sa", "");

        // initialize your dataset here
        IDataSet dataSet = null;
        // ...

        databaseTester.setDataSet(dataSet);
        // will call default setUpOperation
        databaseTester.onSetup();
    }

    protected void tearDown() throws Exception {
    // will call default tearDownOperation
        databaseTester.onTearDown();
    }
    ...
}
```

## 数据库数据校验

Dbunit 提供了对验证两个表或数据集是否包含相同数据的支持。以下两种方法可以用来验证你的数据库在测试用例执行期间是否包含预期的数据。

```java
public class Assertion {
    public static void assertEquals(ITable expected, ITable actual)
    public static void assertEquals(IDataSet expected, IDataSet actual)
}
```

### 示例

下面的例子，显示了如何将数据库表快照与 XML 表进行比较。

```java
public class SampleTest extends DBTestCase {
    public SampleTest(String name) {
        super(name);
    }

    // Implements required setup methods here
    ...

    public void testMe() throws Exception {
        // Execute the tested code that modify the database here
        ...
        // Fetch database data after executing your code
        IDataSet databaseDataSet = getConnection().createDataSet();
        ITable actualTable = databaseDataSet.getTable("TABLE_NAME");

        // Load expected data from an XML dataset
        IDataSet expectedDataSet = new FlatXmlDataSetBuilder().build(new File("expectedDataSet.xml"));
        ITable expectedTable = expectedDataSet.getTable("TABLE_NAME");

        // Assert actual database table match expected table
        Assertion.assertEquals(expectedTable, actualTable);
    }
}
```

实际数据集是一个数据库快照，你要对照预期数据集来验证。顾名思义，预期数据集包含预期值。
预期数据集必须与你用来设置数据库的数据集不同。因此，你需要两个数据集，一个在测试前设置数据库，另一个在测试中提供预期数据。

## 使用查询来获取数据库快照

你也可以验证一个查询的结果是否与预期的数据集相匹配。查询可以用来只选择一个表的子集，甚至把多个表连接起来。

```java
ITable actualJoinData = getConnection().createQueryTable("RESULT_NAME",
                "SELECT * FROM TABLE1, TABLE2 WHERE ..."); 
```

## 忽略对比的列

有时，忽略一些列来进行比较是可取的；特别是对于主键、日期或时间列，它们的值是由被测代码生成的。做到这一点的一个方法是在你的预期表中省略不需要的列。然后你可以过滤实际的数据库表，只显示预期表的列。

下面的代码片段向你展示了如何过滤实际表。要发挥作用，实际表必须至少包含预期表的所有列。额外的列可以存在于实际表中，但不能存在于预期表中。

```java
ITable filteredTable = DefaultColumnFilter.includedColumnsTable(actual,
            expected.getTableMetaData().getColumns());
Assertion.assertEquals(expected, filteredTable); 
```

这种技术的一个主要限制是，你不能使用 DTD 与你预期的 XML 数据集。使用 DTD，你需要从预期的和实际的表中过滤列。

## 行的排列

默认情况下，DbUnit 排列的数据库表快照是按主键排序的。如果一个表没有主键或者主键是由你的数据库自动生成的，那么行的排序是不可预测的，assertEquals 将失败。

你必须通过使用带有 "ORDER BY "子句的 IDatabaseConnection.createQueryTable 手动排序你的数据库快照。或者你可以像这样使用 SortedTable 装饰器类。

```java
Assertion.assertEquals(new SortedTable(expected),
                new SortedTable(actual, expected.getTableMetaData()));
```

注意，SortedTable 默认使用每一列的字符串值来进行排序。因此，如果你对一个数字列进行排序，你会发现排序顺序是 1、10、11、12、2、3、4。如果你想使用列的数据类型进行排序（以获得像 1、2、3、4、10、11、12 这样的列），你可以按以下方式进行。

```java
SortedTable sortedTable1 = new SortedTable(table1, new String[]{"COLUMN1"});
sortedTable1.setUseComparable(true); // must be invoked immediately after the constructor
SortedTable sortedTable2 = new SortedTable(table2, new String[]{"COLUMN1"});
sortedTable2.setUseComparable(true); // must be invoked immediately after the constructor
Assertion.assertEquals(sortedTable1, sortedTable2);
```

目前不在构造函数中使用该参数的原因是，SortedTable 需要的构造函数数量将从 4 个增加到 8 个，这是一个很大的数目。我们应该继续讨论如何在未来以最佳方式实现这一功能。

## 断言并收集差异

默认情况下，dbunit 在发现第一个数据差异时立即失败。从 dbunit 2.4 开始，可以注册一个自定义的 FailureHandler，让用户指定哪些类型的异常被抛出以及如何处理数据差异的发生。使用 DiffCollectingFailureHandler，你可以避免在数据不匹配时抛出一个异常，这样你就可以在事后评估所有的数据比较结果。

```java
IDataSet dataSet = getDataSet();
DiffCollectingFailureHandler myHandler = new DiffCollectingFailureHandler();
//invoke the assertion with the custom handler
assertion.assertEquals(dataSet.getTable("TEST_TABLE"),
                       dataSet.getTable("TEST_TABLE_WITH_WRONG_VALUE"),
                       myHandler);
// Evaluate the results and throw an failure if you wish
List diffList = myHandler.getDiffList();
Difference diff = (Difference)diffList.get(0);
...
```

## 数据文件的加载

有了 Dbunit Ant 任务，Dbunit 使运行以数据库为中心的应用程序的 Canoo WebTest 脚本变得更加容易。WebTest 是一个模拟用户的浏览器在网站上点击页面的工具。它允许你为你的网站创建一系列基于 Ant 的测试。事实上，这可以用来对使用非 Java 技术（如ColdFusion 或 ASP）建立的网站进行用户验收测试！这也是一个很好的例子。本文档将引导你了解存储测试的建议格式。

### Step 1 创建 dataset file

你的第一步是创建你的数据集文件，在运行你的 WebTest 脚本之前将其加载到数据库中。使用上面描述的各种方法中的一种。把你需要的各种数据集放在一个 /data 目录中。

### Step 2 创建你的 Ant build.xml

一个建议的设置是有一个单一的build.xml文件，作为所有测试的入口。这将包括几个目标，如。

1. test: 运行你已经创建的所有测试套件
2. test: single: 在一个特定的测试套件中运行一个单一的测试
3. test: suite: 运行一个特定测试套件的所有测试

### Step 3 创建你的各种测试套件

一旦你建立了 build.xml 文件，你现在可以调用各种 TestSuite。为你想测试的各种模块创建一个单独的 TestSuiteXXX.xml。在TestSuiteXXX.xml 中，你应该让你的默认目标 TestSuite 调用你定义的所有测试用例。

```xml
<target name="testSuite">
    <antcall target="unsubscribeEmailAddressWithEmail"/>
    <antcall target="unsubscribeEmailAddressWithEmailID"/>
    <antcall target="unsubscribeEmailAddressWithNewEmailAddress"/>
    <antcall target="subscribeEmailAddressWithOptedOutEmail"/>
    <antcall target="subscribeEmailAddressWithNewEmailAddress"/>
    <antcall target="subscribeEmailAddressWithInvalidEmailAddress"/>
</target>
```

这样，你既可以运行你的测试套件中的所有测试，也可以只运行一个特定的测试，所有这些都来自 build.xml!

### Step 4 创建你的各种测试

现在你需要编写你的各种测试用例。关于 WebTest 的更多信息，请参考 WebTest 主页。如果你发现你在重复 XML 的片段，那么把它们放在 /includes 目录下。如果你有一套单一的属性，那么通过在 build.properties 文件中指定它们，将它们作为 build.xml 的一部分加载。如果你有多个数据库需要连接，那么在 TestSuiteXXX.properties 文件中声明你的sql连接属性，你在每个套件的基础上加载。在这个例子中，我们使用清洁插入数据库，并使用 MSSQL_CLEAN_INSERT 而不是 CLEAN_INSERT，因为需要做身份列插入。

```xml
<target name="subscribeEmailAddressWithOptedOutEmail">
    <dbunit
        driver="${sql.jdbcdriver}"
        url="${sql.url}"
        userid="${sql.username}"
        password="${sql.password}">
            <operation type="MSSQL_CLEAN_INSERT"
                    src="data/subscribeEmailAddressWithOptedOutEmail.xml"
            format="flat"/>
    </dbunit>
    <testSpec name="subscribeEmailAddressWithOptedOutEmail">
        &amp;sharedConfiguration;
        <steps>
        <invoke stepid="main page"
            url="/edm/subscribe.asp?e=subscribeEmailAddressWithOptedOutEmail@test.com"
            save="subscribeEmailAddressWithNewEmailAddress"/>
        <verifytext stepid="Make sure we received the success message"
            text="You have been subscribed to the mailing list"/>
        </steps>
    </testSpec>
</target>
```