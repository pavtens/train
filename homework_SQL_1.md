1) SELECT c.CustomerName, c.Country, c.Address FROM [Customers] c
where c.Country='Germany' or c.Country='France' or c.City='Madrid'

2) SELECT new_table.num, new_table.country
  FROM (SELECT COUNT(c.CustomerID) AS num, c.Country AS country FROM [Customers] c GROUP BY c.Country) new_table
  ORDER BY new_table.num DESC LIMIT 3

3) SELECT s.ShipperName, o.OrderDate FROM [Shippers] s, [Orders] o
WHERE s.ShipperID = o.ShipperID
ORDER BY o.OrderDate LIMIT 1 OFFSET 9

4) Последний join в данной задаче был необходим в связи с тем, что "не работает" условие с функцией max, когда пытаешься задать where ProductSum=max(ProductSum). Можно вручную урезать таблицу с помощью oreder by desc и limit, однако мы ведь в теории не знаем сколько позиций в заказе. Возможно ли как-то сделать проще?

select ITOG.ProductName, ITOG.Price from
(select ProductSum, ProductName, Price from   
   (select NT_1.OrderID, OrderDetails.ProductID, NT_1.ProductSum from
      (select NT_0.OrderID, sum(NT_0.ProductCost) as ProductSum from 
        (SELECT od.OrderID, p.Price*od.Quantity as ProductCost FROM OrderDetails od
        join Products p on od.ProductID = p.ProductID) NT_0
      group by NT_0.OrderID) NT_1
   join OrderDetails on NT_1.OrderID = OrderDetails.OrderID) NT_2
join Products on NT_2.ProductID = Products.ProductID) ITOG

join (select max(NT_1.ProductSum) as ProductSum from
     (select NT_0.OrderID, sum(NT_0.ProductCost) as ProductSum from 
     (SELECT od.OrderID, p.Price*od.Quantity as ProductCost FROM OrderDetails od
     join Products p on od.ProductID = p.ProductID) NT_0
     group by NT_0.OrderID) NT_1) NT_sbor
on ITOG.ProductSum = NT_sbor.ProductSum

5)
select NT_2.ProductName, od.OrderID, od.Quantity from
   (select *, max(NT_1.TotalQuantity) from
      (select *, sum(NT_0.Quantity) as TotalQuantity from 
        (SELECT * FROM OrderDetails od
        join Products p on od.ProductID = p.ProductID) NT_0
      group by NT_0.ProductID) NT_1) NT_2
join OrderDetails od on od.ProductID=NT_2.ProductID

6)
select SupplierName, Country, ContactName, Phone from  
  (select *, count(OrderDetailID) as Counts from
    (SELECT s.SupplierID, s.SupplierName, s.Country, s.ContactName, s.Phone, p.ProductID FROM Suppliers s
    join Products p on p.SupplierID=s.SupplierID) NT_0
  join OrderDetails od on od.ProductID=NT_0.ProductID
  group by SupplierID
  order by Counts desc)
limit 5

7)
select Country, ca.CategoryName, Sum from  
  (select NT_1.Country, p.CategoryID, sum(NT_1.Quantity*p.Price) as Sum from   
     (select NT_0.Country, od.ProductID, od.Quantity from   
        (SELECT c.Country, o.OrderID FROM Customers c
        join Orders o on c.CustomerID=o.CustomerID
        where c.Country='Brazil') NT_0
     join OrderDetails od on NT_0.OrderID=od.OrderID) NT_1
  join Products p on p.ProductID=NT_1.ProductID
  group by p.CategoryID
  order by Sum desc limit 1) NT_ITOG
join Categories ca on ca.CategoryID=NT_ITOG.CategoryID

8)
select max(Sum)-min(Sum) from
   (select sum(NT_1.Quantity*p.Price) as Sum from
      (select NT_0.OrderID, od.ProductID, od.Quantity from
        (SELECT o.OrderID FROM Customers c
        join Orders o on c.CustomerID=o.CustomerID
        where c.Country='USA') NT_0
      join OrderDetails od on od.OrderID=NT_0.OrderID) NT_1
    join Products p on p.ProductID=NT_1.ProductID
    group by OrderID)
     
9) Начиная с данного пунтка ДЗ, делал по другой ссылке w3schools, где работает суммирование текстовых строк (и concat), а также substring. Насколько понял, по этой ссылке в качестве интрепретатора вместо MySQL идет SQL Server / MS Access Syntax: https://www.w3schools.com/sql/trysqlserver.asp?filename=trysql_func_sqlserver_substring 

select Sum as Counts, FirstName+' '+LastName as FullName from 
  (select EmployeeID, count(OrderID) as Sum from
    (select NT_0.EmployeeID, o.OrderID from
      (SELECT top 3 BirthDate, EmployeeID FROM Employees e
      order by BirthDate desc) NT_0
    join Orders o on o.EmployeeID=NT_0.EmployeeID) NT_1
  group by EmployeeID) NT_ITOG
join Employees e on e.EmployeeID=NT_ITOG.EmployeeID

10)
select sum(Quantity*NumberOfTins) from
  (SELECT od.Quantity, substring(Unit,1,2) as NumberOfTins FROM Products p
  join OrderDetails od on od.ProductID=p.ProductID
  where ProductName like '%crab%') itog