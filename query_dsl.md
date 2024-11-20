# QueryDSL in Spring Framework: A Complete Guide

## Introduction

QueryDSL is a powerful framework that enables the construction of type-safe SQL-like queries for multiple backends including JPA, MongoDB, and SQL. In this comprehensive guide, we'll explore how to use QueryDSL with Spring Framework to write type-safe, maintainable, and elegant queries.

## Table of Contents
1. [Setting Up QueryDSL](#setting-up-querydsl)
2. [Basic Concepts](#basic-concepts)
3. [Writing Your First Query](#writing-your-first-query)
4. [Advanced Querying Techniques](#advanced-querying-techniques)
5. [Best Practices](#best-practices)
6. [Real-World Examples](#real-world-examples)

## Setting Up QueryDSL

First, let's add the necessary dependencies to your `pom.xml`:

```xml
<dependencies>
    <!-- QueryDSL JPA -->
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-jpa</artifactId>
        <version>5.0.0</version>
        <classifier>jakarta</classifier>
    </dependency>
    
    <!-- QueryDSL APT -->
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-apt</artifactId>
        <version>5.0.0</version>
        <classifier>jakarta</classifier>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>com.mysema.maven</groupId>
            <artifactId>apt-maven-plugin</artifactId>
            <version>1.1.3</version>
            <executions>
                <execution>
                    <goals>
                        <goal>process</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>target/generated-sources/java</outputDirectory>
                        <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## Basic Concepts

Let's start with a simple entity:

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private BigDecimal price;
    private String category;
    
    // getters and setters
}
```

After compilation, QueryDSL will generate a `QProduct` class that we'll use for querying.

## Writing Your First Query

Here's how to create a basic repository with QueryDSL:

```java
@Repository
public class ProductRepository {
    
    private final JPAQueryFactory queryFactory;
    
    public ProductRepository(EntityManager entityManager) {
        this.queryFactory = new JPAQueryFactory(entityManager);
    }
    
    public List<Product> findByCategory(String category) {
        QProduct product = QProduct.product;
        
        return queryFactory.selectFrom(product)
                          .where(product.category.eq(category))
                          .fetch();
    }
}
```

## Advanced Querying Techniques

### Dynamic Queries

One of QueryDSL's strongest features is its ability to build dynamic queries:

```java
public List<Product> searchProducts(String name, String category, BigDecimal minPrice) {
    QProduct product = QProduct.product;
    
    BooleanBuilder builder = new BooleanBuilder();
    
    if (name != null) {
        builder.and(product.name.containsIgnoreCase(name));
    }
    
    if (category != null) {
        builder.and(product.category.eq(category));
    }
    
    if (minPrice != null) {
        builder.and(product.price.goe(minPrice));
    }
    
    return queryFactory.selectFrom(product)
                      .where(builder)
                      .fetch();
}
```

### Complex Joins

QueryDSL makes it easy to write complex joins:

```java
@Entity
public class Order {
    @Id
    private Long id;
    
    @ManyToOne
    private Customer customer;
    
    @OneToMany
    private List<OrderItem> items;
}

public List<Order> findOrdersWithDetails() {
    QOrder order = QOrder.order;
    QCustomer customer = QCustomer.customer;
    QOrderItem orderItem = QOrderItem.orderItem;
    
    return queryFactory.selectFrom(order)
                      .leftJoin(order.customer, customer)
                      .leftJoin(order.items, orderItem)
                      .fetch();
}
```

### Subqueries

QueryDSL supports subqueries elegantly:

```java
public List<Product> findProductsWithHigherThanAveragePrice() {
    QProduct product = QProduct.product;
    
    return queryFactory.selectFrom(product)
                      .where(product.price.gt(
                          JPAExpressions
                              .select(product.price.avg())
                              .from(product)
                      ))
                      .fetch();
}
```

## Best Practices

1. **Use Projections for DTO**

```java
public List<ProductDTO> findProductDTOs() {
    QProduct product = QProduct.product;
    
    return queryFactory
        .select(Projections.constructor(ProductDTO.class,
            product.id,
            product.name,
            product.price))
        .from(product)
        .fetch();
}
```

2. **Extract Common Predicates**

```java
public class ProductPredicates {
    public static BooleanExpression isActive() {
        return QProduct.product.active.isTrue();
    }
    
    public static BooleanExpression priceGreaterThan(BigDecimal price) {
        return price != null ? QProduct.product.price.gt(price) : null;
    }
}
```

## Real-World Examples

### Advanced Search with Pagination

```java
public Page<Product> searchProducts(ProductSearchCriteria criteria, Pageable pageable) {
    QProduct product = QProduct.product;
    
    JPAQuery<Product> query = queryFactory
        .selectFrom(product)
        .where(
            ProductPredicates.isActive(),
            ProductPredicates.priceGreaterThan(criteria.getMinPrice()),
            Optional.ofNullable(criteria.getCategory())
                .map(product.category::eq)
                .orElse(null)
        )
        .offset(pageable.getOffset())
        .limit(pageable.getPageSize());

    long total = query.fetchCount();
    List<Product> products = query.fetch();
    
    return new PageImpl<>(products, pageable, total);
}
```

### Complex Aggregation

```java
public List<CategoryStats> getCategoryStatistics() {
    QProduct product = QProduct.product;
    
    return queryFactory
        .select(Projections.constructor(CategoryStats.class,
            product.category,
            product.price.avg(),
            product.price.min(),
            product.price.max(),
            product.count()))
        .from(product)
        .groupBy(product.category)
        .fetch();
}
```

## Conclusion

QueryDSL provides a powerful, type-safe way to write complex queries in Spring applications. Its main advantages include:

- Type safety and compile-time error checking
- Clean and readable syntax
- Great integration with Spring Framework
- Support for dynamic queries
- Excellent IDE support with code completion
