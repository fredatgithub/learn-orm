---
PermaID: 100009
Name: Stored Procedures
---

# Stored Procedures

So far, we have specified all of our SQL in code. But what if you want to use stored procedures for your data access layer? That is perfectly fine, and **Dapper** can work seamlessly with stored procedures. 

A stored procedure is a prepared SQL code that you can save, so the code can be reused over and over again.

Let's consider the following stored procedure.

```csharp
CREATE PROCEDURE [dbo].[GetAuthor]
	@Id int
AS
BEGIN
	SELECT [Id]
		  ,[FirstName]
		  ,[LastName]
	  FROM [dbo].[Authors]
	WHERE Id = @Id;

	SELECT 
		Id,
		Title,
		Category,
		AuthorId
	FROM [dbo].[Books] 
	WHERE AuthorId = @Id;

END
```

Now execute the above stored-procedure in **SQL Query** editor.

<img src="images/stored-procedures-1.png" alt="Create a stored procedure">

Let's write a method to call the above stored-procedure. We will still use the `QueryMultiple` method just like we did before, but instead of specifying inline SQL with two different statements, we will specify the name of the stored procedure and pass in an `Id` as a parameter. 

```csharp
private static void GetAuthorAndTheirBooksSP(int id)
{
    string sql = "GetAuthor";

    using (IDbConnection db = new SqlConnection(ConnectionString))
    {
        using (var results = db.QueryMultiple(sql, new { Id = id }, commandType: CommandType.StoredProcedure ))
        {
            var author = results.Read<Author>().SingleOrDefault();
            var books = results.Read<Book>().ToList();

            if (author != null && books != null)
            {
                author.Books = books;

                Console.WriteLine(author.FirstName + " " + author.LastName);

                foreach (var book in author.Books)
                {
                    Console.WriteLine("\t Title: {0} \t  Category: {1}", book.Title, book.Category);
                }
            }
        }
    }
}
```

You also need to specify `commandType: CommandType.StoredProcedure` as a third parameter. You can see that it is almost identical to how we had it before. 

Let's call the `GetAuthorAndTheirBooks` method in the `Main` method.

```csharp
static void Main(string[] args)
{
    GetAuthorAndTheirBooks(2);
    GetAuthorAndTheirBooks(3);
}
```

Let's execute the above code, and you will see the following output.

```csharp
Meredith Alonso
         Title: Calculus I        Category: Education
         Title: Calculus II       Category: Education
         Title: Trigonometry Basics       Category: Education
Robert T. Kiyosaki
         Title: Rich Dad, Poor Dad        Category: Economics
```
