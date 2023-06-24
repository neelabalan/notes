# jdbc


```java

package org.connectiondemo;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectionDemo
{
	String user_name, password, url_path, driver;
	Connection conn;
	public ConnectionDemo() throws ClassNotFoundException, SQLException
	{
		user_name="system"; // the username of db
		password="12345678";// the pass of db

        // driver in jdbc
		driver="oracle.jdbc.driver.OracleDriver";

        // url is jdbc:oracle:thin@ipadd (localhost):1521 is port number
		url_path="jdbc:oracle:thin:@localhost:1521:XE";
		Class.forName(driver); // requires classnotfoundexception

        // requires sql exception
        // creating an object called conn with getConnection with
        // the above parameters
		conn=DriverManager.getConnection(url_path, user_name, password);

		System.out.println("connection established with return code "+ conn);
	}
}

```

```
>> output

connection established with return code oracle.jdbc.driver.T4CConnection@4629104a
```

```java
package org.connectiondemo;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.*;

public class ConnectionDemo
{

	String datavals[] = new String[4];
	private static Scanner sc =  new Scanner(System.in);
	String user_name, password, url_path, driver;
	Connection conn;
	Statement st;
	public ConnectionDemo() throws ClassNotFoundException, SQLException
	{
		user_name="system";
		password="12345678";
		driver="oracle.jdbc.driver.OracleDriver";
		url_path="jdbc:oracle:thin:@localhost:1521:XE";
		Class.forName(driver);
		conn=DriverManager.getConnection(url_path, user_name, password);
		st=conn.createStatement();
		System.out.println("connection established with return code "+ conn);
	}
	void CreateTable()
	{
		String cmd = "create table emp(empid int, name varchar2(20), salary float, deptid int)";
		try
		{
			st.executeUpdate(cmd);
			System.out.println("successful creation of table");
		}
		catch (SQLException e)
		{
			e.printStackTrace();
		}
	}
	void insertRecord()
	{
		System.out.println("enter employee id - ");
		datavals[0] = sc.next();
		System.out.println("enter name 		  - ");
		datavals[1] = sc.next();
		System.out.println("enter salary 	  - ");
		datavals[2] = sc.next();
		System.out.println("enter depid 	  - ");
		datavals[3] = sc.next();

		String cmd = "insert into emp values("+datavals[0]+",'"+datavals[1]+"',"+datavals[2]+","+datavals[3]+")";
		try
		{
			st.executeUpdate(cmd);
			System.out.println("count val - ");
			System.out.println(st.executeUpdate(cmd));
			System.out.println("data inserted successfuly");
		}
		catch (SQLException e)
		{
			e.printStackTrace();
		}
	}
    public static void main(String[] args) throws ClassNotFoundException, SQLException
	{
		ConnectionDemo demo = new ConnectionDemo();
//		demo.CreateTable();
		demo.insertRecord();
	}
}
```

```
>> output

connection established with return code oracle.jdbc.driver.T4CConnection@619a5dff
enter employee id -
90
enter name 		  -
bal
enter salary 	  -
89
enter depid 	  -
88
count val -
1
data inserted successfuly
```

- for getting the table values

```java
	void getTable() throws SQLException
	{
		String strd = "select * from emp";
		ResultSet rs = st.executeQuery(strd);
		rs.next();
		System.out.println(rs.getInt(1)+"\t"+rs.getString(2));
	}
```

- alternatively

```java
while(rs.next())
{
	System.out.println(rs.getInt(1)+"\t"+rs.getString(2));
}
```

- alternative better code

```java
void getTable() throws SQLException
{
    String strd = "select * from emp";
    // executequery works for select * from emp kind of query
    ResultSet rs = st.executeQuery(strd);

    //getting metadata
    ResultSetMetaData rsmd = rs.getMetaData();

    while(rs.next())
    {
        // making sure that iterator starts from 1 and
        // last column is also included
        for(int i=1; i<=rsmd.getColumnCount(); i++)
        {
            System.out.print(" "+rs.getString(i));
        }
        System.out.println();
    }
}
```

- for getting column name

```java
for(int i=1; i<=rsmd.getColumnCount(); i++)
{
    System.out.print(" "+rsmd.getColumnName(i));
}
```

- aggregation meaning loose coupled code

* connection with data stored in file

```java
package org.connectionfile;

import java.io.FileReader;
import java.sql.Connection;
import java.sql.DriverManager;
import java.util.Properties;
import java.util.*;

public class ConnectionFile
{

	static public Connection getConnectionToDB()
	{

		String username, password, url;
		Connection conn=null;
		try
		{
			FileReader fr = new FileReader("datafile");

			Properties prop = new Properties();
			prop.load(fr);
			Class.forName(prop.getProperty("driver"));

			System.out.println();
			username =  prop.getProperty("user");
			System.out.println(username);
			password = prop.getProperty("password");
			System.out.println(password);
			url = prop.getProperty("url");
			System.out.println(url);

			conn = DriverManager.getConnection(url,username,password);
		}
		catch(Exception e)
		{
			e.printStackTrace();
		}
		finally
		{
			if(conn != null)
			{
				try
				{
					conn.close();
				}
				catch(Exception e)
				{
					e.printStackTrace();
				}
			}
		}
		return conn;
	}
}

public class TestConnection
{
	public static void main(String[] args)
	{
		System.out.println(ConnectionFile.getConnectionToDB());
		System.out.println("successful data reading");
	}
}

```

- getting dbms info

```java
try
{
    DatabaseMetaData dbms = new getMetaData();
    System.out.println(dbms.getDatabaseMajorVersion());
    System.out.println(dbms.getDriverName());
    System.out.println(dbms.getDatabaseProductName());
}
catch(SQLException e)
{
    e.printStackTrace();
}

```

```sql
SQL> r 1
  1  create or replace procedure getempname(eid IN number, ename OUT varchar2)
  2   is begin
  3   select name into ename
  4   from emp
  5   where empid=eid;
  6*  end getempname;

Procedure created.
```

> in the above code the procedure getempname is created
> that procedure is now called from the java program

```java
package org.callablestatement;
import org.connectionfile.ConnectionFile;

import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Types;
import java.util.Scanner;

public class CallableStatementDemo
{
	Connection conn;

    // callable statement - procedure cst object
	CallableStatement cst;
	Scanner sc = new Scanner(System.in);
	public CallableStatementDemo()
	{
		conn = ConnectionFile.getConnectionToDB();
		try
		{
            //prepare call with argument string
			cst=conn.prepareCall("{call getempname(?, ?)}");
		}
		catch (SQLException e)
		{
			e.printStackTrace();
		}
	}
	public void getDataFromDB()
	{
		System.out.println("enter emp id : ");
		 int id = sc.nextInt();

		try
		{
            // setting input for first parameter
			cst.setInt(1, id);
            // setting output for second parameter
			cst.registerOutParameter(2, Types.VARCHAR);
			cst.execute();

            // storing the output in string and printing it
			String empname = cst.getString(2);
			System.out.println(empname);
		}
		catch (Exception e)
		{
			e.printStackTrace();
		}
	}
	public static void main(String[] args)
	{
        // calling it like this in the main function
		new CallableStatementDemo().getDataFromDB();
	}
}
```

```
> output

system
12345678
jdbc:oracle:thin:@localhost:1521:XE
enter emp id :
123
prateek
```

- jdbc procedure

1. import the packages
   `import java.sql.*;`

2. register the jdbc driver
   `Class.forName("com.mysql.jdbc.Driver");`

3. open a connection
   `conn = DriverManager.getConnection(DB_URL,USER,PASS);`

4. execute a query

```java
stmt = conn.createStatement();
      String sql;
      sql = "SELECT id, first, last, age FROM Employees";
      ResultSet rs = stmt.executeQuery(sql);
```

5. extract data from result

```java
while(rs.next())
{
	//Retrieve by column name
	int id  = rs.getInt("id");
	int age = rs.getInt("age");
	String first = rs.getString("first");
	String last = rs.getString("last");

	//Display values
	System.out.print("ID: " + id);
	System.out.print(", Age: " + age);
	System.out.print(", First: " + first);
	System.out.println(", Last: " + last);
}
```

6. clean up env

```
rs.close();
stmt.close();
conn.close();
```

- insert data into table

```java
package org.preparedstatement;

import org.connectionfile.ConnectionFile;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.Scanner;

public class PreparedStatementDemo
{
	PreparedStatement pst;
	Connection conn;
	Scanner sc = new Scanner(System.in);
	String name;
	float salary;
	int empid, deptid;
	public PreparedStatementDemo()
	{
		conn=ConnectionFile.getConnectionToDB();
		try
		{
			pst=conn.prepareStatement("insert into emp values(?, ?, ?, ?)");
		}
		catch(SQLException e)
		{
			e.printStackTrace();
		}
	}
	public void acceptEmpDetails()
	{
		System.out.println("enter your name ");
		name = sc.next();

		System.out.println("enter your salary");
		salary = sc.nextFloat();

		System.out.println("enter your depid");
		deptid = sc.nextInt();

		System.out.println("enter your empid");
		empid = sc.nextInt();

		try
		{
			pst.setInt(1,  empid);
			pst.setString(2, name);
			pst.setFloat(3, salary);
			pst.setInt(4, deptid);
			System.out.println("record inserted successfully");

			// send sql query from java to backend
			pst.executeUpdate();
		}
		catch(SQLException e)
		{
			e.printStackTrace();
		}
	}
	public static void main(String[] args)
	{
		new PreparedStatementDemo().acceptEmpDetails();

	}
}
```

- executequery is kind of universal for execution
- executequery done only when select is used in query
- executeupdate insert into operations
  `pst.executeUpdate`

- rs.absolute

```java
// selecting the 2nd row here
// negative value passing results in starting from last row
// similarly relative can be used to get from absolute path
rs.absolute(2);
System.out.println(rs.getString(1)+ " " + rs.getString(2));
```

- automatically incrementing emp id with jdbc

```java
package org.preparedstatement;

import org.connectionfile.ConnectionFile;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Scanner;

public class PreparedStatementDemo
{
	PreparedStatement pst;
	Statement st;
	Connection conn;
	Scanner sc = new Scanner(System.in);
	String name;
	float salary;
	int empid, deptid;
	public PreparedStatementDemo()
	{
		conn=ConnectionFile.getConnectionToDB();
		try
		{
			// st need create statement here
			// passing values 1005 and 1008 is here
			// only upon passing the values 1005 and 1008 the absolute
			// with -1 parameter works from the last of the table
			st=conn.createStatement(1005, 1008);

			// alternatively creating a sequence in sql
			/* create sequence empidgen (to be done in sql)
			*  insert into emp values(empidgen.nextVal, ?, ?, ?);
			*  in the try block remove
			*  pst.setInt(1, empid);
			*  next sequence of pst.setString(1, name)
			*  starts with 1
			*/
			pst=conn.prepareStatement("insert into emp values(?, ?, ?, ?)");
		}
		catch(SQLException e)
		{
			e.printStackTrace();
		}
		// the empid needs to be set primary key (not necessary)
	}
	public void acceptEmpDetails() throws SQLException
	{
		empid=0;
		String strd = "select * from emp";
		ResultSet rs = st.executeQuery(strd);
		rs.absolute(-1);
		empid=rs.getInt(1);
		System.out.println(empid+ "is emp id");

		empid++;
		System.out.println("enter your name ");
		name = sc.next();
		System.out.println("enter your salary");
		salary = sc.nextFloat();
		System.out.println("enter your depid");
		deptid = sc.nextInt();
//		System.out.println("enter your empid");
//		empid = sc.nextInt();



		try
		{
			pst.setInt(1,  empid);
			pst.setString(2, name);
			pst.setFloat(3, salary);
			pst.setInt(4, deptid);
			System.out.println("record inserted successfully");
			pst.executeUpdate();
		}
		catch(SQLException e)
		{
			e.printStackTrace();
		}
	}
	public static void main(String[] args) throws SQLException
	{
		new PreparedStatementDemo().acceptEmpDetails();

	}
}



```

```java
package org.loadimagedb;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;

import org.connectionfile.ConnectionFile;

public class StoreImageDb
{
	Statement st;
	PreparedStatement pst;
	Connection conn;

	public StoreImageDb()
	{

		conn = ConnectionFile.getConnectionToDB();
		try
		{
			st = conn.createStatement();
			pst = conn.prepareStatement("insert into imgTable values(?, ?)");
		}
		catch(SQLException e)
		{
			e.printStackTrace();
		}
	}
	public void createTable()
	{
		try
		{
			st.executeUpdate("create table imgTable(id int, photo blob)");
			System.out.println("table created");
		}
		catch(SQLException e)
		{
			e.printStackTrace();
		}
	}
	public void insertImage() throws FileNotFoundException
	{
		File file = new File("C:\\users\\neelabalan.n\\image.png");
		try
		{
			FileInputStream fls = new FileInputStream(file);
			pst.setInt(1,  121);
			pst.setBinaryStream(2, fls, file.length());
			pst.executeUpdate();
			System.out.println("successful store of image");
		}
		catch(SQLException e)
		{
			e.printStackTrace();
		}
	}
	public static void main(String[] args) throws FileNotFoundException
	{
		StoreImageDb img = new StoreImageDb();
		img.createTable();
		img.insertImage();
	}
}

```

> output
> binary data stored in the database

```
SQL> select * from imgtable;

        ID
----------
PHOTO
--------------------------------------------------------------------------------
       121
89504E470D0A1A0A0000000D4948445200000356000002CB0803000000CD0DE370000000AB504C54
45000000000000000000000000000000000000000000000000000000000000000000000000000000
```

```
- connection
> it is a pointer to database

- java.sql.ResultSet

> it is a pointer to database table.
> result set is nothing but an object oriented representation of
> database table. By default result set will point to "no record area".
> hence to point to record area we need to call nextMethod on result set object.
> nextMethod shift to next are and also return boolean value
```
