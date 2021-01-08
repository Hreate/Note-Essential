# JDBC

完整示例，Statement：

```java
public class DbUtil {
	// 标准 url
    public static final String URL = "jdbc:mysql://localhost:3306/db1?useSSL=true&useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC";
    public static final String USER = "root";
    public static final String PASSWORD = "greenday";
    
    public static void main(String[] args) throws Exception {
    	Connection conn = null;
		Statement stmt = null;
		try {
            // 1. 加载驱动程序
			Class.forName("com.mysql.cj.jdbc.Driver");
            // 2. 获得数据库连接
			conn = DriverManager.getConnection(URL, USER, PASSWORD);
            // 3. 操作数据库，实现增删改查
			stmt = conn.createStatement();
			ResultSet rs = stmt.executeQuery("select * from user");
            // 如果有数据，rs.next() 返回 true
			while (rs.next()) {
				System.out.println(rs.getString("name") + rs.getString("age"));
			}
		} catch (ClassNotFoundException | SQLException e) {
			e.printStackTrace();
		} finally {
			try {
				if (stmt != null) {
					stmt.close();
				}
			} catch (SQLException throwables) {
				throwables.printStackTrace();
			}
			try {
				if (conn != null) {
					conn.close();
				}
			} catch (SQLException throwables) {
				throwables.printStackTrace();
			}
		}
    }
}
```



PreparedStatement：

```java
public class DbUtil {
	
    public static final String URL = "jdbc:mysql://localhost:3306/db1";
    public static final String USER = "root";
    public static final String PASSWORD = "greenday";
    
    public static void main(String[] args) throws Exception {
    	Connection conn = null;
		PreparedStatement ps = null;
		try {
			Class.forName("com.mysql.cj.jdbc.Driver");
			conn = DriverManager.getConnection(URL, USER, PASSWORD);
			String sql = "select * from user where id = ?";
			ps = conn.prepareStatement(sql);
//			ps.setInt(1, 2);
			ps.setObject(1, 2);
			ResultSet rs = ps.executeQuery();
			while (rs.next()) {
				System.out.println(rs.getString("name") + rs.getString("age"));
			}
		} catch (ClassNotFoundException | SQLException e) {
			e.printStackTrace();
		} finally {
			try {
				if (ps != null) {
					ps.close();
				}
			} catch (SQLException throwables) {
				throwables.printStackTrace();
			}
			try {
				if (conn != null) {
					conn.close();
				}
			} catch (SQLException throwables) {
				throwables.printStackTrace();
			}
		}
    }
}
```



# Druid

1. 直接导入 druid 依赖：

   ```java
   Properties props = new Properties();
   InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("durid.properties");
   props.load(is);
   DataSource ds = DruidDataSourceFactory.createDataSource(props);
   Connection conn = ds.getConnection();
   System.out.println(conn);
   conn.close();
   ```

2. 导入 druid-spring-boot-starter 依赖：

   在 application.yml 中配置 DataSource。



# 事务

1. 取消自动提交
2. 操作执行完毕，手动提交
3. 操作出现错误，进行回滚

完整示例：

```java
Connection conn = null;
PreparedStatement ps = null;
try {
    Properties props = new Properties();
    InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("durid.properties");
    props.load(is);
    DataSource ds = DruidDataSourceFactory.createDataSource(props);
    conn = ds.getConnection();
    // 取消自动提交
    conn.setAutoCommit(false);
    String sql = "INSERT INTO user (name, age) VALUES (?,?)";
    ps = conn.prepareStatement(sql);
    ps.setString(1, "Acho");
    ps.setInt(2, 19);
    ps.execute();
    // 操作执行完毕，手动提交
    conn.commit();
} catch (Exception e) {
    e.printStackTrace();
    try {
        if (!ObjectUtils.isEmpty(conn)) {
            conn.rollback();
        }
    } catch (SQLException throwables) {
        throwables.printStackTrace();
    }
} finally {
    try {
        if (ps != null) {
            ps.close();
        }
    } catch (SQLException throwables) {
        throwables.printStackTrace();
    }
    try {
        if (conn != null) {
            conn.close();
        }
    } catch (SQLException throwables) {
        throwables.printStackTrace();
    }
}
```

