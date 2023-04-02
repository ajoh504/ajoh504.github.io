---
layout: post
title: A Quick Guide for Setting up SQLite with C#
excerpt_separator: <!--more-->
---

Earlier today, I added a SQLite reference to a C# project. Since this is only my second time implementing a database in a coding project, I decided to put together a quick reference guide.<!--more>

Note: The installation portion of this guide assumes the use of Visual Studio. 

### Install SQLite In Visual Studio

1. Open Visual Studio
2. Select Tools > Nuget Packet Manager > Package Manager Console
3. From the Nuget console, enter the following command: `Install-Package System.Data.SQLite`. To verify that the package was installed correctly, check the Project references and the .csproj file 
4. Press "Ctrl + Alt + L" to open the Solution Explorer
5. Click on your Project
6. Click Dependencies > Packages. The SQLite assembly should be there
7. Double-click your Project to open the .csproj file. This file should show the following, with the appropriate version number:
```xml
  <ItemGroup>
    <PackageReference Include="System.Data.SQLite" Version="x.x.xxx" />
  </ItemGroup>
```

### Create The Database File and Database Tables

If necessary, have the program check for the database file. 

If it doesn't exist, create it.

```cs
static void Main(string[] args)
{
    private readonly string databasePath = "C:\\Path\\To\\Database.sqlite";
    if (!File.Exists(databasePath)
    {
        SQLiteConnection.CreateFile(databasePath);
    }
}  
```

Next, we need a database `SQLiteConnection` object and a connection string.

The connection string is passed as an argument to the database connection object. It's also good practice to wrap the connection object in a `using` statement. This ensures that if the program fails, the connection to the file will be properly closed.

Use the following methods to open a connection:

```cs
SQLiteConnection connection = new SQLiteConnection($"Data Source={databasePath};Version=3;");
connection.Open()
using (connection)
{
}
```

To create any database tables, use the following SQL statement:
```sql
CREATE TABLE table_name (id_column INT, text VARCHAR(100)
```

Since [SQLite will generate](https://www.sqlitetutorial.net/sqlite-create-table/) a `rowid` for every row, you can use the `PRIMARY KEY` constraint to act as an alias for the `rowid`. This might be useful if you want some custom logic to determine how your row ids are generated. So instead of creating the table with  `id_column INT` you can use `id_column INTEGER PRIMARY KEY` instead. 

Finally, to create the table, we'll do three things:
1. Create a `SQLiteCommand` object
2. Pass the string command and the connection object into its constructor
3. Invoke the `ExecuteNonQuery()` method

Like so:

```cs
using(SQLiteConnection connection = new SQLiteConnection($"Data Source={databasePath};Version=3;"))
{
    connection.Open();
    string cmd = "CREATE TABLE table_name (id_column INTEGER PRIMARY KEY, text VARCHAR(100))";
    using (SQLiteCommand command = new SQLiteCommand(cmd, connection))
    {
        command.ExecuteNonQuery();
    }
}
```

### Add Data to the Database

Adding data to the database is nearly identical to creating a new table. The only difference is in the SQL command string. 

Take the following:

```cs

using(SQLiteConnection connection = new SQLiteConnection($"Data Source={databasePath};Version=3;"))
{
    connection.Open();
    string cmd = "INSERT INTO table_name (id_column, text) values (1, 'data')";
    using (SQLiteCommand command = new SQLiteCommand(cmd, connection))
    {
        command.ExecuteNonQuery();
    }
}
```

### Retrieve Data from the Database

Finally, now that we have data stored in our database, we need a way to retrieve it. 

To do this, we need three objects: 
1. A `SQLiteCommmand` object 
2. A `SQLiteConnection` object
3. A `SQLiteDataReader ` object

Once we have a `SQLiteDataReader ` object, we can use the methods `GetInt32()` and `GetString()` to retrieve data from the database. Each of these methods can accept an index as an argument, referring to the location of the row.

As such:

```cs
using (SQLiteConnection connection = new SQLiteConnection($"Data Source={databasePath};Version=3;"))
{
    connection.Open();
    string cmd = "SELECT * FROM table_name";
    using (SQLiteCommand command = new SQLiteCommand(cmd, connection))
    {
        using(SQLiteDataReader reader = command.ExecuteReader())
        {
            while (reader.Read())
            {
                int id = reader.GetInt32(0);
                string data = reader.GetString(1);
                Console.WriteLine($"{id}, {data}");
            }
        }
    }
}
```