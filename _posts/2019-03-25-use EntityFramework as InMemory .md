---
layout: post
title: "EntityFramework ile InMemory çalışmak"
comments: true
description: ""
keywords: "EntityFramework memory inmemory .net .netcore EF"
---
Unit test yazanlar bilir, veritabanı üzerindeki işlemlerde unit test yazmak baya zahmetlidir. Ya bir test veritabanı ayarlamak gerekir, ya da işlemleri hafızadan (memory)'den yapabilmek için Entity'lerin bulunduğu fake DbSet yapıları kurmak gerekiyor. **.Net Core** ile birlikte geliştiricileri bu zahmetten kurtarmak için EntityFramework (EF)'e InMemory desteği geldi. 

İlk önce bu özelliği kullanabilmek için nuget üzeriden **Microsoft.EntityFrameworkCore.InMemory** kütüphanesini indirmek gerekiyor. Bunun için PM'i açıp

```csharp
 Install-Package Microsoft.EntityFrameworkCore.InMemory
```

yazarak paketi indirebilirsiniz. Şimdi sıra geldi Entity modellerimizi yazmaya,

```csharp
   public class Order
   {
        [Key]
        public Guid Id { get; set; }
        public DateTime CreatedAt { get; set; }
        public virtual ICollection<Product> Products { get; set; }
   }

   public class Product
   {
        [Key] 
        public Guid Id { get; set; }
        public string Name { get; set; }
        public DateTime CreatedAt { get; set; }
   }
```

yazmış olduğumuz entityler veritabanı için kullandığımız ile aynı. Şimdi bir DbContext üzerinden dışarıya açalım.


```csharp
    public class TestContext :DbContext
    {
        public DbSet<Order> Orders { get; set; }
        public DbSet<Product> Products { get; set; }
        public TestContext(DbContextOptions options):base(options)
        {
            
        }
    }
```
**TestContext** sınıfımız üzeriden veritabanına bağlanmak ya da memory üzeriden çalışmak istediğimizde parametre olarak istenilen **DbContextOptions** yani **options'ı** değiştirmemiz yeterli olmaktadır. 

```csharp
     var inMemoryOptions = new DbContextOptionsBuilder<TestContext>()
                .UseInMemoryDatabase(Guid.NewGuid().ToString())
                .Options;
        using (var dbContext = new TestContext(inMemoryOptions))
            {
                dbContext.Orders.Add(new Order()
                {
                    CreatedAt = DateTime.Now,
                    Products = new List<Product>()
                    {
                        new Product()
                        {
                            Name = "Gömlek",
                            CreatedAt = DateTime.Now
                        },
                        new Product()
                        {
                            Name = "Ceket",
                            CreatedAt = DateTime.Now
                        }
                    }
                });
                dbContext.SaveChanges();
            }
```
**DbContextOptionsBuilder** üzerinden kullandığımız **UseInMemoryDatabase** metodu parametre olarak kendi belirlediğimiz bir isimi almakta. Eğer program kapanana kadar tek bir veritabanı üzerinden gitmek istiyorsanız aynı ismi verebilirsiniz ya da test senaryolarınızı buna göre yazabilirsiniz. 
