# Advanced Active Recored Query with Arel
## contributors: 
1. abdullah al-yazidy    https://github.com/abdullah-alyazidy
2. abdullah bin shamlan  https://github.com/abdalluh-BinShamlan-Jisr

## Index

1. **ActiveRecord**
2. **ActiveRecord not enough**
3. **Examples**
4. **Arel**
5. **More Advanced**
6. **Gifts**

## 1 Active Recored
- **Power full tool**     It provid many tools Like Find ,find_by, where and so...
- **Flexibility**     build queries in the Ruby way
- **Easy Usage**      Make your database access very easy

## 2 Active record not enough
In some scenarios we end up having to write SQL statements in Strings because ActiveRecord just won’t generate the SQL we need for us.

## 3 Examples

```ruby

Employee.where('department_id > ?' , 5)
```

```ruby

Role.where('name ILIKE ?', '%shoe%')
```

```ruby

Role.where(id:2).pluck('name AS non')
```

```ruby

Role.pluck("LOWER(name)")
```


**:heavy_exclamation_mark: PROBLEM**
: Sometimes SQL row is bad choose

> DEPRECATION WARNING: Dangerous query method (method whose arguments are used as raw SQL) called with non-attribute argument(s): "status.status = 'pending'". Non-attribute arguments will be disallowed in Rails 6.1. This method should not be called with user-provided values, such as request parameters or model attributes. Known-safe values can be passed by wrapping them in Arel.sql(). (called from __pry__ at (pry):2)

***Active Recored warns us about this...***

**:heavy_check_mark: SOLUTION**

 It is recommended to use **Arel** !!!
___
 ## 4 Arel
 
**Abbreviation**
 : A Relational Algebra

 **What is ?**
 : Arel is a SQL AST Manager for Ruby

 **Generality**
:  it is adapts to various RDBMSes

**Who use ?**
 : Active Recoreds use Arel to build queries

### Arel components

- Attributes
- Collectors
- Nodes
- Visitors

### Arel  benefits  for starters

- avoid ambiguous matches
Arel took care of quoting the column name for us and also prefixed it with the table name to avoid ambiguous matches
- Work with ruby
now we’re not writing SQL inside a ruby file anymore
- sql syntax
Take out from your mind sql syntax problems

> ###  Notes about Arel
> - Arel just generate SQL
> - Arel Don,t know about database
> - Arel don,t execute the generated SQL

___
### Define Arel table from ActiveRecord model

```ruby

users = Arel::Table.new(:users)
```

```ruby

users = User.arel_table
```


### **Fields**

```ruby

users[:name]
```
```ruby

users[:id]
```

### Where
```ruby

users.where(users[:name].eq('amy'))
```
> SELECT * FROM users WHERE users.name = 'amy'

### Select
```ruby

users.project(users[:id])
```
> SELECT users.id FROM users

### Order
```ruby

users.order(users[:name])
```

```ruby

users.order(users[:name], users[:age].desc)
```

```ruby

users.reorder(users[:age])
```


*- **bad code***
```ruby

Task.order("tasks.status = 'Cancelled' DESC, tasks.employee_id = 2107  DESC, tasks.created_at DESC")`
```

*- **better code***
```ruby

Task.order(Arel.sql("tasks.status = 'Pending' DESC, tasks.employee_id = #{employee.id} DESC, tasks.created_at DESC"))`
```

*- **best code***
```ruby

Task.order(Task[:status].eq('Cancelled').desc, Task[:employee_id].eq(2107).desc, Task[:created_at].desc)`
```


### Order (with custom order (case))

```ruby

order_status = Arel::Nodes::Case.new(PaymentTransfer.arel_table[:status]).
            when(::PaymentTransfer.statuses['cancelled']).then('1').
            when(::PaymentTransfer.statuses['failed']).then('2').
            else(PaymentTransfer.arel_table[:status]).asc
```
___
### Pluck

- **without arel**
```ruby

Role.joins(:employees).pluck('employees.id')
```

- **with arel**
```ruby

employee = Employee.arel_table
Role.joins(:employees).pluck(employee[:id])
```
___
### Extracting

- **Without Arel**

```ruby

Employee.distinct.pluck("EXTRACT(year FROM employees.created_at)")
```

- **With Arel**
```ruby

Employee.distinct.pluck(Employee.arel_table[:created_at].extract("year"))
```
___
### Aggregates
- **Module Arel::Expressions**

```ruby
users.project(users[:age].sum) # .average .minimum .maximum
```

```ruby

users.project(users[:id].count)
```

```ruby
users.project(users[:id].count.as('user_count'))
```

**Other Aggregates**
- minimum
- average
- sum
- count
- maximum
- extract(field)

___
### Predications
- **Module Arel::Predications**
 Moving on. You can build several different conditions with Arel. We have methods for    **=,   IN,   >,   >=,   <,   <=**

> **First: we should defin this to use in following examples :small_red_triangle_down:**
```ruby
product = Product.arel_table
```

```ruby

Product.where(product[:name].eq('shoe'))

> SELECT "products".* FROM "products" WHERE "products"."name" = 'shoe'
```

```ruby

Product.where(product[:name].in(['shoe', 'sneakers']))

> SELECT "products".* FROM "products" WHERE "products"."name" IN ('shoe', 'sneakers')
```

```ruby

Product.where(product[:price].gt(100))

> SELECT "products".* FROM "products" WHERE "products"."price" > 100.0
```

```ruby

Product.where(product[:price].gteq(100))

> SELECT "products".* FROM "products" WHERE "products"."price" >= 100.0
```

```ruby

Product.where(product[:price].lt(100))

> SELECT "products".* FROM "products" WHERE "products"."price" < 100.0
```

```ruby

Product.where(product[:price].lteq(100))

> SELECT "products".* FROM "products" WHERE "products"."price" <= 100.0
```

```ruby

Product.where(product[:name].not_eq('shoe'))

> SELECT "products".* FROM "products" WHERE "products"."name" != 'shoe'
```

```ruby

Product.where(product[:name].not_in(['shoe', 'sneakers']))

> SELECT "products".* FROM "products" WHERE "products"."name" NOT IN ('shoe', 'sneakers')
```

### AND / OR

- **Module Arel::Nodes::Node**
We can also combine conditions together. **If you use where clauses from ActiveRecord you know they are gonna generate you AND conditions.**

```ruby

Product.where(product[:name].eq('shoe')).where(product[:id].eq(1))

> SELECT "products".* FROM "products" WHERE "products"."name" = 'shoe' AND "products"."id" = 1
```

```ruby

Product.where(product[:name].eq('shoe').or(product[:id].eq(1)))

> SELECT "products".* FROM "products" WHERE ("products"."name" = 'shoe' OR "products"."id" = 1)

```

```ruby

Product.where(product[:name].eq('shoe').or(product[:id].eq(1).and(product[:name].matches('%sneakers%'))))

> SELECT "products".* FROM "products" WHERE ("products"."name" = 'shoe' OR "products"."id" = 1 AND "products"."name" ILIKE '%sneakers%')
```
___
### Grouping
```ruby

role = Role.arel_table
```
```ruby
Role.where(role.grouping(role[:id].gt(5).and(role[:name].eq('admin'))).or(role[:name].not_eq('employee')))
```

### Alias Table

```ruby

creators = User.arel_table.alias('creators')

updaters = User.arel_table.alias('updaters')

photos = Photo.arel_table

photos_with_credits =\
  photos.join(photos.join(creators, Arel::Nodes::OuterJoin).on(photos[:created_by_id].eq(creators[:id])))
        .join(photos.join(updaters, Arel::Nodes::OuterJoin).on(photos[:assigned_id].eq(updaters[:id])))
        .project(photos[:name], photos[:created_at], creators[:name].as('creator'), updaters[:name].as('editor'))
```

```ruby

photos_with_credits.to_sql
```
___

## 5 More Advanced

- ### Arel::Nodes::SqlLiteral

```ruby

Arel.sql("tasks.status = 'Pending' DESC, tasks.employee_id = #{employee.id} DESC, tasks.created_at DESC"))
```
- ### JOIN

| join type        | join type in arel    |
| :----            | :----               |
| Inner Join       | InnerJoin / defualt  |
| Full Outer Join  | FullOuterJoin        |
| Left Outer Joi n | OuterJoin            |
| Right Outer Join | RightOuterJoin       |

**Examples**

- **Without Arel**
```ruby

Role.joins('LEFT OUTER JOIN employees ON roles.id = employees.role_id')

> SELECT "roles".* FROM "roles" LEFT OUTER JOIN employees ON roles.id = employees.role_id
```
- **With Arel**

```ruby

role = Role.arel_table
employee = Employee.arel_table
```

```ruby

Role.joins(role.join(employee, Arel::Nodes::OuterJoin).on(role[:id].eq(employee[:role_id])).join_sources)

> SELECT "roles".* FROM "roles" LEFT OUTER JOIN "employees" ON "roles"."id" = "employees"."role_id"
```

- **Other type (FullOuterJoin)**

```ruby

Role.joins(role.join(employee, Arel::Nodes::FullOuterJoin).on(role[:id].eq(employee[:role_id])).join_sources)
```

- **Join with conditions**

```ruby

Role.select(Arel.star).joins(role.join(employee,Arel::Nodes::RightOuterJoin).on(employee[:role_id].eq(role[:id])).join_sources).where(employee[:id].lt(10))
```
___

### Join Filtering

Suppos we have Role and Employee models, and we want to **get all Role with all Employees just that have specific condition**
**such as:** 
 we want to retrive all Roles with active Employees  Otherwise return Role with null value
> note ... now we need to applay the condition on just employees **with keep retivving all Roles**

- **Without Arel**

```ruby
Employee.left_joins(:role).count
>> 3656
```

```ruby
Employee.left_joins(:role).where("roles.role_type = 1").count
>> 1004
```
**:heavy_exclamation_mark:**
note it filters Roles and Employees, And that's not what we want

just you can doing it by **sql pure**

```ruby

Employee.joins("LEFT OUTER JOIN roles ON employees.role_id = roles.id  AND roles.role_type = 1").count

>> 3656
```

- **With Arel**

```ruby

employee = Employee.arel_table
role = Role.arel_table
```
```ruby

Employee.joins(employee.join(role, Arel::Nodes::OuterJoin).on(employee[:role_id].eq(role[:id]).and(employee[:status].eq('active'))).join_sources).count

>> 3656
```

## 6 Gifts :gift:

- ### Shorthand for arel_table
When writing Arel in Rails you access columns through the arel_table method on your models like so:
Employee.arel_table[:email]. However, there is a lovely little shortcut that is often added to shorten this:

```ruby

class ApplicationRecord < ActiveRecord::Base
  def self.[](attribute)
    arel_table[attribute]
  end
end
```
This way you can write **Employee[:email]** instead of **Employee.arel_table[:email]**
___

- ### Customize
Create Custome Function
***config > initializers > arel_date_extension.rb***

you can define methods to use anywhere in your project

```ruby

module Arel
  module Attributes
    module AttributeExtension

      def distinct
        Arel::Nodes::NamedFunction.new('DISTINCT', [self])
      end

      def contain(child)
        Arel::Nodes::InfixOperation.new('@>',
          [self],
          child)
      end

      def generate_series(start_date, end_date)
        Arel::Nodes::NamedFunction.new('GENERATE_SERIES',
               [start_date, end_date, Arel::Nodes::SqlLiteral.new('\'1 day\'')]
        )
      end
    end
  end
end

Arel::Attributes::Attribute.send :include, Arel::Attributes::AttributeExtension
```
then use it simple and easy
```ruby

Model.where(Model[:field].contain(Arel::Nodes.build_quoted('{1}')))
```
___

- ### Converter :arrows_counterclockwise:
Sometimes you have a text sql and you don't know how to write it in Arel
Here is this site... it converts any text sql to Arel

:rocket: ***Have fun with Arel***

http://www.scuttle.io/


