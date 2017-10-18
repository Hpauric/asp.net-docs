---
title: ASP.NET Reference

toc_footers:
  - <a href='https://docs.microsoft.com/en-us/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/'>EF6 Code First</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---

# C\# Basics

## Attributes

Attributes are effectively metadata that describe the property, method or class that they precede. They are encased in square brackets.

```csharp
[Display(Name = "Release Date")] // attribute
public DateTime ReleaseDate { get; set; }
```

## Generics

Generics can be classes or structs. Type-parameters are names used in place of concrete types when defining a new generic. They can be associated with classes or methods by placing the type parameter in angle brackets `< >`.

```csharp
public class List<T> // T stands for type
{
...
}
List<int> list = new List<int>();
List<char> list = new List<char>();
List<Person> list = new List<Person>();
```


## ASP.NET Classes Cheatsheet

### Some Useful Classes

- IEnumerable `System.Collections.Generic`
- DbContext `System.Data.Entity`
- Controller `System.Web.Mvc`
- HTMLHelper `System.Web.Mvc`

# Entity Framework

Why do we need Entity Framework?
You can use Entity Framework for code-first implementations. This allows you to define your models in code, and let Entity Framework generate the database for you.

## Using Entity Framework

The main class that coordinates Entity Framework functionality for a given data model is the database context class. You create this class by deriving from the `System.Data.Entity.DbContext` class. In your code you specify which entities are included in the data model.

## `DbContext`

The `DbContext` class you create will usually be quite simple.

```csharp
    public class ProjectContext : DbContext
    {

        public ProjectContext() : base("ProjectContext")
        {
        }

        public DbSet<Student> Students { get; set; }
        public DbSet<Equipment> Equipments { get; set; }

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
        }
    }
```
### DbContext Methods

## Creating the Schema

If you look at the example application Microsoft use to demonstrate the features, you'll see that in the first excercise they create three entities:
- Student
- Course
- Enrollment

If you were creating your first application you might think you would just need two entities: `Student` and `Course`. Why do we need Enrollment? Why can't our Students have the course they are enrolled in as properties, and our Course have Students as properties? What does the `Enrollment` entity bring to the table?

> The many-to-many relationship involves defining a third table (called a junction or join table), whose primary key is composed of the foreign keys from both related tables. 

Is this true for the Enrollment entity? No. It has it's own EnrollmentID key.

## Navigation Properties

> Navigation properties provide a way to navigate an association between two entity types. Every object can have a navigation property for every relationship in which it participates. Navigation properties allow you to navigate and manage relationships in both directions, returning either a reference object (if the multiplicity is either one or zero-or-one) or a collection (if the multiplicity is many). You may also choose to have one-way navigation, in which case you define the navigation property on only one of the types that participates in the relationship and not on both.

https://msdn.microsoft.com/en-us/library/jj713564(v=vs.113).aspx

```csharp
public virtual OfficeAssignment OfficeAssignment { get; set; }
```

If a navigation property can hold multiple entities, its type must implement the ICollection<T> Interface.
  
```csharp  
public virtual ICollection<Equipment> Equipment { get; set; }  
```

The `Equipment` property is a navigation property. Navigation properties hold other entities that are related to this entity. In this case, the `Equipment` property of a `Student` entity will hold all of the `Equipment` entities that are related to that Student entity. In other words, if a given `Student` row in the database has two related `Equipment` rows (rows that contain that student's primary key value in their StudentID foreign key column), that `Student` entity's `Equipment` navigation property will contain those two `Equipment` entities.

Navigation properties are typically defined as virtual so that they can take advantage of an Entity Framework feature called lazy loading.

## Code First

Entity Framework allows you to develop a data model with a Code First approach.

This means that you can define your models and Entity Framework will create the database for you.

## The Connection String

A connection string provides the information that a provider needs to communicate with a particular database. The Connection String includes parameters such as the name of the driver, Server name and Database name , as well as security information such as user name and password.

Enter this text in the `Web.config` file in your project.

```html
<connectionStrings>
    <add name="[YOURMODELNAMEHERE]Context"    
         connectionString="Data Source=(LocalDB)\MSSQLLocalDB;
  AttachDbFilename=|DataDirectory|\[YOURDATABASENAMEHERE].mdf;
  Integrated Security=True" 
         providerName="System.Data.SqlClient"/>
  </connectionStrings>
```

You don't actually have to have a connection string in the `Web.config` file. If you don't supply a connection string, Entity Framework will use a default one based on your context class. However in that case the data won't save to a database.

Note that In Visual Studio 2012 the data source is `(LocalDB)\v11.0`. For Visual Studio 2015 and Visual Studio 2017 it is `(LocalDB)\MSSQLLocalDB`.
`LocalDB` is a lightweight version of the SQL Server Express Database Engine that is targeted for program development. `LocalDB` starts on demand and runs in user mode, so there is no complex configuration. 

### Deploying to Azure

New connection strings are generated when you deploy to Azure.

```html
<connectionStrings>
  <add name="ProjectContext"
    connectionString="$(ReplacableToken_ProjectContext-Web.config Connection String_0)"
    providerName="System.Data.SqlClient" />
  <add name="ProjectContext_DatabasePublish"
    connectionString="$(ReplacableToken_ProjectContext_DatabasePublish-Web.config Connection String_0)"
    providerName="System.Data.SqlClient"/>
</connectionStrings>
```

You can find them in `[ProjectName]\obj\Release\Package\PackageTmp\Web.config`.

## Migrations

Code First Migrations allow you to update your data model and update the database schema without having to drop and re-create the database.

You don't _have_ to use Migrations. However, if you don't, all database content will be lost when you update your models.

[Great introduction to Code First Migrations](https://stackoverflow.com/questions/40606167/error-when-update-database-using-code-first-there-is-already-an-object-named)

## NuGet Commands

There are three main NuGet commands used in migrations.

| Command | Description |
| ------------- | ------------- |
|  `Enable-Migrations` | Install migration modules - only needs to be run initially | 
|  `Add-Migration [MigrationName]` | Generate a new migration code file based on changes in your project models | 
|  `Update-Database` | Implement any outstanding migrations | 

Migrations are code files. Every time you run `Add-Migration` a code file is generated/updated, usually in a folder called `Migrations` inside your project. The name of a migration file is composed with a timestamp of its generation concatenated with the name used when running `Add-Migration`. For example
```bash
Add-Migration Initial
```
generates
```bash
201710021217244_Initial.cs
```
You can check the contents of those files and see the effects of running `Add-Migration`. You can also modify them once generated, and add your own code. From my experience it is often necessary to do this.

## Up and Down Methods

Every single migration has a method `Up` and a method `Down`. 

The `Up` method updates the database. The `Down` method reverts them.

Methods:
- `CreateTable`&nbsp;
- `CreateIndex`&nbsp;
- `AlterColumn`&nbsp;
- `AddForeignKey`&nbsp;
- `DropTable`&nbsp;
- `DropIndex`&nbsp;
- `DropForeignKey`&nbsp;

## Migration History

Migrations are intended to be incremental. You start with an `Initial` migration, and every time you change your model code you generate a new migration file. The database contains a table named `__MigrationsHistory` that keeps trace of which migrations have been run in your database.

When you run `Update-Database` there are always two implicit parameters: `SourceMigration` and `TargetMigration`. EF incrementally applies the `Up` methods of all the migrations between `SourceMigration` and `TargetMigration` (or the `Down` methods if you are downgrading your database). 

The default scenario when you don't specify the `SourceMigration` and `TargetMigration` parameters is that `SourceMigration` is the last migration applied to the database and `TargetMigration` is the last of the pending ones. EF determines those parameters by querying the `__MigrationsHistory` table of the default database of your project, so if that database is not in a consistent state, your migrations can be generated incorrectly.

## Setting the Seed method to update from a csv file.

Rather than painstakingly typing values into the Seed method, you can set it to import from a CSV file.
[There are great instructions on how to do this here.](https://www.davepaquette.com/archive/2014/03/18/seeding-entity-framework-database-from-csv.aspx)
One mistake I made at first was that I referenced the wrong assembly. Stack Overflow recommended "to make sure you're in the right assembly and with right name: dump and evaluate all the resources available in your target assembly." Which I did:

```csharp
string[] names = assembly.GetManifestResourceNames();
string fullString = "";
names.ToList().ForEach(i => fullString += (i.ToString()) + '\n');
```

## When Things Go Wrong

Entity Framework is not perfect.

> Essentially the ORM can handle about 80-90% of the mapping problems, but that last chunk always needs careful work by somebody who really understands how a relational database works.

> Mapping to a relational database involves lots of repetitive, boiler-plate code. A framework that allows me to avoid 80% of that is worthwhile even if it is only 80%. The problem is in me for pretending it's 100% when it isn't.

You can't just run `Add-Migration MyNewMigration` and assume everything will work out fine. You need to check to make sure that the generated code works correctly.

### Debugging the Update-Database command

Running the Update-Database command meant that debugging statments like `Debug.WriteLine()` didn't work. Instead I threw an error:

```csharp
throw new Exception(fullString);
```
Then I could see I couldn't access the resource since:

1. It was saved in the wrong namespace.&nbsp;
2. It's build action was set to `Resource` instead of `Embedded Resource`.&nbsp;

After I fixed these mistakes I could easily Seed from a CSV.

### Erasing Everything - The Easiest Way to Fix a Problem

You can simply delete the database and migrations history. Obviously this is not a viable solution for production code, but it's often the easiest solution for development code.

1. Delete the `Migrations` folder in your solution.
2. Delete the tables in the database.
3. Delete the `_MigrationHistory` table in the database.
4. Close the database connection.
5. In the NuGet console, run `Enable-Migrations`.
6. Run `Add-Migration Initial`.
7. Run `Update-Database`.

## Errors

### Error: `There is already an object named 'ModelName' in the database.`

### Error: `Sequence contains more than one element.`

https://docs.microsoft.com/en-us/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/migrations-and-deployment-with-the-entity-framework-in-an-asp-net-mvc-application#enable-code-first-migrations

The first parameter passed to the AddOrUpdate method specifies the property to use to check if a row already exists. For the test student data that you are providing, the LastName property can be used for this purpose since each last name in the list is unique:
C#Copy
context.Students.AddOrUpdate(p => p.LastName, s)


This code assumes that last names are unique. If you manually add a student with a duplicate last name, you'll get the following exception the next time you perform a migration.
Sequence contains more than one element

https://blogs.msdn.microsoft.com/rickandy/2013/02/12/seeding-and-debugging-entity-framework-ef-dbs/

So it was a Seed error. But it doesn’t cause any big problems. It just means I can’t add the extra data. That’s not a problem, is it?





# Views


## Using Code

> The key transition character in Razor is the “at” sign (@). This single character is used to transition from markup to code.


The `@` transition symbol effectively works as a `console.log` or `println()` mechanism.

Code that is encased in curly brackets will be executed. Code that's simply preceded by `@` will be outputted.

```liquid
@{var test = 10; }
@{test = test -1;}
@test // 9
```



By including a @model statement at the top of the view template file, you can specify the type of object that the view expects.

`@model IEnumerable<Your.Model.Namespace>`

This `@model` directive allows you to access the list of movies that the controller passed to the view

##  Layout

By convention, the default layout for an ASP.NET app is named `_Layout.cshtml`.

The default layout is defined in `_ViewStart.cshtml` file in `Views`.

 
## HTML Helper Methods

The `Html` object is a helper that's exposed using a property on the `System.Web.Mvc.WebViewPage base` class. 

### `Html.BeginForm`

if you leave it empty it will look for a post action with the same name that on the page you are now

- `Html.DisplayNameFor`
- `Html.DisplayFor`
- `Html.ValidationSummary`
- `Html.EditorFor`
- `Html.LabelFor`
- `Html.DropDownList`

### `Html.ActionLink`

The `ActionLink` method of the helper makes it easy to dynamically generate HTML hyperlinks that link to action methods on controllers. 

```csharp
@Html.ActionLink("Edit", "Edit", new { id=item.ID }) 
```
arguments
- string linkText
- string actionName (this is the method name unless otherwise specified)
- string controllerName OR object routeValues

| Type | Name |
| ------------- | ------------- |
| string | linkText | 
| string | actionName (this is the method name unless otherwise specified) | 
| string OR object | controllerName OR routeValues | 

The default route (established in `App_Start\RouteConfig.cs)` takes the URL pattern `{controller}/{action}/{id}`

So for example,the generated route in this case is `http://localhost:1234/Movies/Edit/4`

### `Html.DisplayFor`

# Controllers

Rather than having a direct relationship between the URL and a file living on the web server’s hard
drive, a relationship exists between the URL and a method on a controller class

AccountController: Responsible for account-related requests, such as login and account
registration

A controller can pass data or objects to a view template using the `ViewBag` object. The `ViewBag` is a dynamic object that provides a convenient late-bound way to pass information to a view.

## Creating a `BulkCreate` method

https://docs.microsoft.com/en-us/aspnet/mvc/overview/getting-started/introduction/adding-a-view

> Controller methods (also known as action methods), such as the Index method above, generally return an ActionResult (or a class derived from ActionResult), not primitive types like string.

> simply ran the statement return View(), which specified that the method should use a view template file to render a response to the browser. Because you didn't explicitly specify the name of the view template file to use, ASP.NET MVC defaulted to using the Index.cshtml view file in the \Views\HelloWorld folder. The image below shows the string "Hello from our View Template!" hard-coded in the view.



ASP.NET controller scaffolding will create `Create` action methods for both GET and POST requests.

But what about if you want to bulk create items?


