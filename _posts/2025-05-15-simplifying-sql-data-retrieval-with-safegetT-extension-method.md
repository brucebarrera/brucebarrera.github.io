---
layout: post
title: Simplifying SQL Data Retrieval with SafeGetT Extension Method in C#
date: 2025-05-15 18:58:00 -0500
categories: [C#]
tags: [C#, SQL, Database]
published: true
---

When working with SQL data in C#, developers often need to handle data retrieval from an `IDataReader` safely, accounting for potential null values or missing columns. In this article we will implement a `SqlDataExtensions` static class that provides a useful extension method called `SafeGetT` that simplifies this process by safely retrieving typed data from an `IDataReader` with proper error handling.

## Overview of `SafeGetT`

The `SafeGetT` extension method extends `IDataReader` to retrieve a value from a specified column and cast it to the desired type `T`. It handles common issues such as:
- Invalid or empty column names
- Missing columns
- Null database values
- Type conversion errors

If any error occurs, it returns `default(T)` (e.g., `null` for reference types, `0` for integers, etc.), ensuring the application remains stable.

## Code Implementation

```csharp
public static class SqlDataExtensions
{
    public static T SafeGetT<T>(this IDataReader reader, string columnName)
    {
        if (!string.IsNullOrEmpty(columnName))
        {
            try
            {
                var columnIndex = reader.GetOrdinal(columnName);
                if (!reader.IsDBNull(columnIndex))
                {
                    return (T)reader[columnName];
                }
            }
            catch
            {
                return default(T);
            }
        }
        return default(T);
    }
}
```

## Example Usage

Below is an example of how to use `SafeGetT` to retrieve data from a SQL query result:

```csharp
using System.Data;
using System.Data.SqlClient;

class Program
{
    static void Main()
    {
        using (var connection = new SqlConnection("your_connection_string"))
        {
            connection.Open();
            var command = new SqlCommand("SELECT Id, Name, Price FROM Products", connection);
            using (var reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    int id = reader.SafeGetT<int>("Id");
                    string name = reader.SafeGetT<string>("Name");
                    decimal price = reader.SafeGetT<decimal>("Price");
                    Console.WriteLine($"Product: {id}, {name}, ${price}");
                }
            }
        }
    }
}
```

In this example, `SafeGetT` retrieves the `Id` (int), `Name` (string), and `Price` (decimal) from the `Products` table. If the `Price` column is missing or contains a null value, `SafeGetT<decimal>` returns `0.0m` instead of throwing an exception.

## Unit Test

To ensure the `SafeGetT` method works as expected, below is a unit test using MSTest and Moq to mock an `IDataReader`:

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Moq;
using System.Data;

[TestClass]
public class SqlDataExtensionsTests
{
    [TestMethod]
    public void SafeGetT_ValidColumnWithValue_ReturnsCorrectValue()
    {
        // Arrange
        var readerMock = new Mock<IDataReader>();
        readerMock.Setup(r => r.GetOrdinal("Name")).Returns(0);
        readerMock.Setup(r => r.IsDBNull(0)).Returns(false);
        readerMock.Setup(r => r["Name"]).Returns("TestProduct");

        // Act
        string result = readerMock.Object.SafeGetT<string>("Name");

        // Assert
        Assert.AreEqual("TestProduct", result);
    }

    [TestMethod]
    public void SafeGetT_NullColumnName_ReturnsDefault()
    {
        // Arrange
        var readerMock = new Mock<IDataReader>();

        // Act
        string result = readerMock.Object.SafeGetT<string>(null);

        // Assert
        Assert.IsNull(result);
    }

    [TestMethod]
    public void SafeGetT_ColumnNotFound_ReturnsDefault()
    {
        // Arrange
        var readerMock = new Mock<IDataReader>();
        readerMock.Setup(r => r.GetOrdinal("NonExistent")).Throws(new IndexOutOfRangeException());

        // Act
        int result = readerMock.Object.SafeGetT<int>("NonExistent");

        // Assert
        Assert.AreEqual(0, result);
    }
}
```

This test suite verifies:
1. Correct value retrieval for a valid column with data.
2. Proper handling of a null column name.
3. Graceful handling of a non-existent column.

## Benefits

- **Safety**: Prevents exceptions from null values or missing columns.
- **Simplicity**: Reduces boilerplate code for data retrieval.
- **Flexibility**: Works with any type `T` that can be cast from the reader's data.

## Conclusion

The `SafeGetT` extension method is a valuable tool for C# developers working with SQL data. By incorporating it into your data access layer, you can write cleaner, more robust code that gracefully handles edge cases. The provided example and unit tests demonstrate its practical application and reliability.
