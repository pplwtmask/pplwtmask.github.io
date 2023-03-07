---
title: JdbcTemplate and TransactionTemplate
date: 2022-08-27 15:38:06
---

# JdbcTemplate和TransactionTemplate是如何配合来控制事务的
> JdbcTemplate和TransactionTemplate是spring提供给我们用于操作数据库和数据库事务的非常重要的两个类，我们知道想要手动控制数据库事务是依赖于同一个数据库连接的，那么这两个类之间是如何相互配合做到精准控制的呢。
注：将访问数据库和事务控制分离出来符合面向对象设计原则的单一职责原则，不得不说spring的设计思想一直都是可以的：）

## 1.原生JDBC控制事务

[官方文档](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-usagenotes-connect-drivermanager.html)
```
		/**
         * 加载数据库驱动
         * 可省略详见{@link javax.sql.DataSource}
         */
        Class.forName("com.mysql.cj.jdbc.Driver").newInstance();

        Statement stmt = null;
        ResultSet rs = null;
        Connection conn = null;
        boolean preAutoCommit = false;

        try {

            conn = DriverManager.getConnection("jdbc:mysql://localhost/test?" +
                    "user=minty&password=greatsqldb");

            stmt = conn.createStatement();


            preAutoCommit = conn.getAutoCommit();
            
            conn.setAutoCommit(false);

            rs = stmt.executeQuery("SELECT foo FROM bar");

            // or alternatively, if you don't know ahead of time that
            // the query will be a SELECT...

            if (stmt.execute("SELECT foo FROM bar")) {
                rs = stmt.getResultSet();
            }
            conn.commit();
            // Now do something with the ResultSet ....
        }
        catch (SQLException ex){
            conn.rollback();
            // handle any errors
            System.out.println("SQLException: " + ex.getMessage());
            System.out.println("SQLState: " + ex.getSQLState());
            System.out.println("VendorError: " + ex.getErrorCode());
        }
        finally {
            // it is a good idea to release
            // resources in a finally{} block
            // in reverse-order of their creation
            // if they are no-longer needed

            if (rs != null) {
                try {
                    rs.close();
                } catch (SQLException sqlEx) { } // ignore

                rs = null;
            }

            if (stmt != null) {
                try {
                    stmt.close();
                } catch (SQLException sqlEx) { } // ignore

                stmt = null;
            }


            if (conn != null) {
                try {
                    conn.setAutoCommit(preAutoCommit);
                    conn.close();
                } catch (SQLException sqlEx) { } // ignore

                conn = null;
            }
        }
```

## 2.使用JdbcTemplate和TransactionTemplate控制事务

假定你已经做好了配置
可见相对于原生JDBC，代码非常的简单明了
```
transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                jdbcTemplate.execute("INSERT INTO test.pet\n" +
                        "(name, owner, species, sex, birth, death)\n" +
                        "VALUES('汪汪', '小红', '3232', '男', '1992-01-01 00:00:00', '1992-01-01 00:00:00');");
                jdbcTemplate.execute("INSERT INTO test.pet\n" +
                        "(name, owner, species, sex, birth, death)\n" +
                        "VALUES('汪汪', '小红', '3232', '男', '1992-01-01 00:00:00', '1992-01-01 00:00:00');");
            }
        });
```
## 原理

### 主要用到的类
* `DataSourceTransactionManager`:事务管理器，主要用于管理事务
* `TransactionDefinition`:定义了当前事务的基本信息，隔离性、传播性、超时时间、只读
* `DataSourceTransactionObject`:`DataSourceTransactionManager`中的内部类，内部属性ConnectionHolder保存了当前线程的数据库连接，被用于类`DefaultTransactionStatus`中的属性transaction。
* `TransactionSynchronizationManager`:管理每个线程的资源，比如当前线程的数据库连接

### 执行过程
这里只给出开启事务的核心代码
**开启事务之前会现在DataSourceTransactionObject中获取连接，不存在会直接在数据库连接池获取一个新的连接，然后放到DataSourceTransactionObject中；
最后会将持有连接ConnectionHolder以数据库为键存到TransactionSynchronizationManager中的ThreadLoacle中，为JdbcTemplate的执行做准备；**
org.springframework.jdbc.datasource.DataSourceTransactionManager#doBegin
```java
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;
		Connection con = null;

		try {
			if (!txObject.hasConnectionHolder() ||
					txObject.getConnectionHolder().isSynchronizedWithTransaction()) {
				Connection newCon = obtainDataSource().getConnection();
				if (logger.isDebugEnabled()) {
					logger.debug("Acquired Connection [" + newCon + "] for JDBC transaction");
				}
				txObject.setConnectionHolder(new ConnectionHolder(newCon), true);
			}
			.
			.
			.
// Bind the connection holder to the thread.
			if (txObject.isNewConnectionHolder()) {
				TransactionSynchronizationManager.bindResource(obtainDataSource(), txObject.getConnectionHolder());
			}
```

**JdbcTemplate执行时，会先在TransactionSynchronizationManager中获取当前线程的ConnectionHolder，它里面存放这当前线程的连接，这样控制事务的连接和执行sql的连接保证了是同一个，这样就为我们成功的控制了事务**

org.springframework.jdbc.datasource.DataSourceUtils#doGetConnection
```
		Assert.notNull(dataSource, "No DataSource specified");

		ConnectionHolder conHolder = (ConnectionHolder) TransactionSynchronizationManager.getResource(dataSource);
		if (conHolder != null && (conHolder.hasConnection() || conHolder.isSynchronizedWithTransaction())) {
			conHolder.requested();
			if (!conHolder.hasConnection()) {
				logger.debug("Fetching resumed JDBC Connection from DataSource");
				conHolder.setConnection(fetchConnection(dataSource));
			}
			return conHolder.getConnection();
		}
		// Else we either got no holder or an empty thread-bound holder here.
```


















