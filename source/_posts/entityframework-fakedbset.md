title: "Entity Framework FakeDbSet"
date: 2015-03-25 11:56:15
categories:
- 程式開發
tags:
- C#
- EntityFramework
---

為了單元測試，需要作假資料，如果用資料庫來做，就需要考慮測試資料的建立以及 Transaction 的問題，因為建立測試資料後還得刪除。     
  
所以可以使用 Mock 的方式，把 *IDbSet* 改成使用 InMemory   
  
目前 [Nuget.org]() 就有一個套件，可以幫忙你實作 *InMemorySet*   
我目前是安裝 [AnotherFakeDbSet](https://www.nuget.org/packages/AnotherFakeDbSet/)，Source 在 [Github-AnotherFakeDbSet](https://github.com/realistschuckle/FakeDbSet)    

這個 Source 的原始版本是 [FakeDbSet](https://github.com/a-h/FakeDbSet)  
不過我也覺得 [AnotherFakeDbSet](https://www.nuget.org/packages/AnotherFakeDbSet/) 改的不錯，而且因為陸續有推出新版本，所以我自己也是採用這個 Nuget Package  


<!--more-->


使用方式是可以參考他 Source 裡面的 Examples 專案  

不過要使用這個之前，基本上是要替 DbContext 做一個介面，把 *IDbSet<T>* 的屬性都改成介面屬性  


範例如下:   
    
```csharp
using System;
using System.Data.Entity;

namespace Example.Data
{
	/// <summary>
	/// The interface which data access code works against (data access code uses 
	/// IBookStoreEntities rather than BookStoreEntities).
	/// </summary>
	public interface IBookStoreEntities : IDisposable
	{
		IDbSet<Author> Authors { get; set; }
		IDbSet<Book> Books { get; set; }

		int SaveChanges();
	}
}
```


實作類別:     
  
  
```csharp  
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Data.Entity;

namespace Example.Data
{
	/// <summary>
	/// The real database implementation (as opposed to FakeDatabase).  Note that
	/// we implement the IBookStoreEntities interface here as well as inheriting
	/// from DbContext.
	/// </summary>
	public class BookStoreEntities : DbContext, IBookStoreEntities
	{
		public IDbSet<Book> Books { get; set; }
		public IDbSet<Author> Authors { get; set; }
	}
}
```

測試用的 DbContext:  
  
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Example.Data;
using FakeDbSet;
using System.Data.Entity;

namespace Example.BusinessLogicTest
{
	/// <summary>
	/// This is an example of how we'd create a fake database by implementing the 
	/// same interface that the BookeStoreEntities class implements.
	/// </summary>
	public class FakeDatabase : IBookStoreEntities
	{
		/// <summary>
		/// Sets up the fake database.
		/// </summary>
		public FakeDatabase()
		{
			// We're setting our DbSets to be InMemoryDbSets rather than using SQL Server.
			this.Authors = new InMemoryDbSet<Author>();
			this.Books = new InMemoryDbSet<Book>();
		}

		public IDbSet<Author> Authors { get; set; }
		public IDbSet<Book> Books { get; set; }

		public int SaveChanges()
		{
			// Pretend that each entity gets a database id when we hit save.
			int changes = 0;
			changes += DbSetHelper.IncrementPrimaryKey<Author>(x => x.AuthorId, this.Authors);
			changes += DbSetHelper.IncrementPrimaryKey<Book>(x => x.BookId, this.Books);

			return changes;
		}

		public void Dispose()
		{
			// Do nothing!
		}
	}
}
```


不過在自己程式裡面，要使用 *DbContext* 的時候，就不能像是以下的程式一樣直接使用，因為這樣就無法測試了   

```csharp
using(var context = new BookStoreEntities()) {
  // Code....
}
```


我自己的方式，是採用類似 *Provider* 的概念來做。       

```csharp
public class BookServiceImpl : IBookService {

  public Func<IBookStoreEntities> DbContextProvider { get; set; }
  
  public BookServiceImpl() {
    DbContextProvider = () => new BookStoreEntities();
  }

  public void SaveBook() {
  
    // use provider to provide DbContext
    using(var context = GetDbContext()) {
      // Code...
    }  
  }
  
  private IBookStoreEntities GetDbContext() {
    return _dbContextProvider.Invoke(); 
  }
  
}
```


這樣子，你如果測試的時候要抽換就會變得很容易（當然，如果你要使用類似 IoC 的 framework 實作也 OK~）    