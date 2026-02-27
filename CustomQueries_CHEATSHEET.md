# Spring Data JPA Custom Queries Cheatsheet

This cheatsheet provides examples of custom repository methods using both **Derived Query Methods** and **@Query annotations**.

---

## Part 1: Derived Query Methods (Spring Data Magic)

Derived query methods use a special naming convention. Spring Data generates the SQL automatically!

### Basic Queries

```java
// Find all fighters (already included in JpaRepository)
Page<Fighter> findAll(Pageable pageable);

// Find by single field
Fighter findById(int id);
List<Fighter> findByName(String name);
Page<Fighter> findByHealth(int health, Pageable pageable);

// Find with multiple conditions (AND)
List<Fighter> findByNameAndHealth(String name, int health);
Page<Fighter> findByHealthAndDamage(int health, double damage, Pageable pageable);
```

### Comparison Operators

```java
// Greater Than (>)
Page<Fighter> findByHealthGreaterThan(int health, Pageable pageable);
List<Fighter> findByDamageGreaterThan(double damage);

// Greater Than or Equal (>=)
Page<Fighter> findByHealthGreaterThanEqual(int health, Pageable pageable);
List<Fighter> findByDamageGreaterThanEqual(double damage);

// Less Than (<)
Page<Fighter> findByHealthLessThan(int health, Pageable pageable);
List<Fighter> findByResistanceLessThan(double resistance);

// Less Than or Equal (<=)
Page<Fighter> findByHealthLessThanEqual(int health, Pageable pageable);

// Between
Page<Fighter> findByHealthBetween(int minHealth, int maxHealth, Pageable pageable);
List<Fighter> findByDamageBetween(double minDamage, double maxDamage);
List<Fighter> findByResistanceBetween(double minResistance, double maxResistance);
```

### String Matching

```java
// Partial match (case-sensitive)
List<Fighter> findByNameContaining(String name);
Page<Fighter> findByNameContaining(String name, Pageable pageable);

// Case-insensitive partial match
Page<Fighter> findByNameContainingIgnoreCase(String name, Pageable pageable);
List<Fighter> findByNameContainingIgnoreCase(String name);

// Starts with
List<Fighter> findByNameStartingWith(String prefix);
Page<Fighter> findByNameStartingWith(String prefix, Pageable pageable);

// Ends with
List<Fighter> findByNameEndingWith(String suffix);

// Like (SQL LIKE operator)
List<Fighter> findByNameLike(String pattern);
```

### Ordering Results

```java
// Order by (Ascending - default)
List<Fighter> findByHealthGreaterThanOrderByDamageAsc(int health);
List<Fighter> findAllByOrderByNameAsc();

// Order by (Descending)
List<Fighter> findByHealthGreaterThanOrderByDamageDesc(int health);
List<Fighter> findAllByOrderByDamageDesc();
Page<Fighter> findAllByOrderByDamageDesc(Pageable pageable);

// Multiple order by
List<Fighter> findAllByOrderByHealthDescDamageAsc();
List<Fighter> findByHealthGreaterThanOrderByDamageDescResistanceAsc(int health);
```

### Negation

```java
// NOT
List<Fighter> findByNameNot(String name);
Page<Fighter> findByHealthNot(int health, Pageable pageable);

// Is Null / Is Not Null
List<Fighter> findByNameIsNull();
List<Fighter> findByNameIsNotNull();
```

### Boolean Logic

```java
// OR condition
List<Fighter> findByNameOrHealth(String name, int health);

// AND condition (implicit with multiple params)
List<Fighter> findByNameAndHealth(String name, int health);

// Combining: Name is "Kazuya" OR Health > 1300
// This requires @Query - see Part 2
```

---

## Part 2: @Query Annotations (Custom JPQL)

Use `@Query` for complex queries that can't be expressed with derived methods.

### Basic JPQL Queries

```java
// Simple SELECT all
@Query("SELECT f FROM Fighter f")
List<Fighter> getAllFighters();

// SELECT all with pagination
@Query("SELECT f FROM Fighter f")
Page<Fighter> getAllFighters(Pageable pageable);

// SELECT by single field
@Query("SELECT f FROM Fighter f WHERE f.name = ?1")
Fighter findByExactName(String name);

// SELECT by single field with pagination
@Query("SELECT f FROM Fighter f WHERE f.health > ?1")
Page<Fighter> findStrongFighters(int health, Pageable pageable);
```

### Comparison Operators in JPQL

```java
// Greater than
@Query("SELECT f FROM Fighter f WHERE f.damage > ?1")
List<Fighter> findHighDamageFighters(double damage);

// Greater than or equal
@Query("SELECT f FROM Fighter f WHERE f.health >= ?1")
Page<Fighter> findTankFighters(int minHealth, Pageable pageable);

// Less than
@Query("SELECT f FROM Fighter f WHERE f.resistance < ?1")
List<Fighter> findSquishy(double maxResistance);

// Between
@Query("SELECT f FROM Fighter f WHERE f.health BETWEEN ?1 AND ?2")
List<Fighter> findFightersByHealthRange(int minHealth, int maxHealth);

// Not equal
@Query("SELECT f FROM Fighter f WHERE f.name != ?1")
List<Fighter> findAllExcept(String name);
```

### String Operations

```java
// LIKE (partial match)
@Query("SELECT f FROM Fighter f WHERE f.name LIKE %?1%")
List<Fighter> findByNameContaining(String name);

// Case-insensitive LIKE
@Query("SELECT f FROM Fighter f WHERE LOWER(f.name) LIKE LOWER(CONCAT('%', ?1, '%'))")
Page<Fighter> findByNameContainingIgnoreCase(String name, Pageable pageable);

// LIKE with wildcard positions
@Query("SELECT f FROM Fighter f WHERE f.name LIKE ?1%")
List<Fighter> findByNameStartingWith(String prefix);

@Query("SELECT f FROM Fighter f WHERE f.name LIKE %?1")
List<Fighter> findByNameEndingWith(String suffix);
```

### AND / OR Logic

```java
// AND (multiple conditions)
@Query("SELECT f FROM Fighter f WHERE f.health > ?1 AND f.damage > ?2")
Page<Fighter> findStrongAndDangerous(int minHealth, double minDamage, Pageable pageable);

// OR
@Query("SELECT f FROM Fighter f WHERE f.name = ?1 OR f.name = ?2")
List<Fighter> findTwoFighters(String name1, String name2);

// Complex: (Health > 1200) AND (Damage < 90) OR (Resistance > 7)
@Query("SELECT f FROM Fighter f WHERE (f.health > ?1 AND f.damage < ?2) OR f.resistance > ?3")
List<Fighter> findBalancedOrTankFighters(int health, double damage, double resistance);
```

### ORDER BY

```java
// Order ascending
@Query("SELECT f FROM Fighter f ORDER BY f.damage ASC")
List<Fighter> findAllOrderByDamageAsc();

// Order descending
@Query("SELECT f FROM Fighter f ORDER BY f.damage DESC")
Page<Fighter> findAllOrderByDamageDesc(Pageable pageable);

// Order by multiple fields
@Query("SELECT f FROM Fighter f ORDER BY f.health DESC, f.damage ASC")
List<Fighter> findAllOrderedByHealthThenDamage();

// Order with WHERE clause
@Query("SELECT f FROM Fighter f WHERE f.health > ?1 ORDER BY f.damage DESC")
Page<Fighter> findTanksByDamage(int minHealth, Pageable pageable);
```

### Named Parameters (Better Practice!)

Instead of `?1`, `?2`, use `@Param` for clarity:

```java
// Using named parameters
@Query("SELECT f FROM Fighter f WHERE f.name LIKE %:name% AND f.health > :health")
Page<Fighter> findByNameAndMinHealth(@Param("name") String name, 
                                     @Param("health") int health, 
                                     Pageable pageable);

// Another example
@Query("SELECT f FROM Fighter f WHERE f.damage BETWEEN :minDamage AND :maxDamage")
List<Fighter> findByDamageRange(@Param("minDamage") double minDamage, 
                                @Param("maxDamage") double maxDamage);
```

### Aggregations

```java
// COUNT
@Query("SELECT COUNT(f) FROM Fighter f")
long countAllFighters();

@Query("SELECT COUNT(f) FROM Fighter f WHERE f.health > ?1")
long countStrongFighters(int minHealth);

// MAX / MIN
@Query("SELECT MAX(f.damage) FROM Fighter f")
Double findMaxDamage();

@Query("SELECT MIN(f.health) FROM Fighter f")
Integer findMinHealth();

// AVG
@Query("SELECT AVG(f.damage) FROM Fighter f")
Double findAverageDamage();
```

### DISTINCT

```java
// Remove duplicates
@Query("SELECT DISTINCT f FROM Fighter f ORDER BY f.name")
List<Fighter> findDistinctFighters();

@Query("SELECT DISTINCT f.name FROM Fighter f")
List<String> findDistinctFighterNames();
```

### IN Clause

```java
// Find fighters with IDs in a list
@Query("SELECT f FROM Fighter f WHERE f.id IN ?1")
List<Fighter> findFightersByIds(List<Integer> ids);

// Find fighters with names in a list
@Query("SELECT f FROM Fighter f WHERE f.name IN (?1)")
List<Fighter> findByNames(List<String> names);
```

### UPDATE & DELETE (Advanced)

```java
// UPDATE query
@Modifying
@Transactional
@Query("UPDATE Fighter f SET f.health = f.health + ?1 WHERE f.id = ?2")
void increaseHealth(int amount, int fighterId);

// DELETE query
@Modifying
@Transactional
@Query("DELETE FROM Fighter f WHERE f.damage < ?1")
void deleteLowDamageFighters(double maxDamage);
```

---

## Lab 4 Examples - For Your Assignment

### Example 1: Derived Query (Name Search)
```java
@Repository
public interface FighterRepository extends JpaRepository<Fighter, Integer> {
    
    // Search fighters by name containing (case-insensitive)
    Page<Fighter> findByNameContainingIgnoreCase(String name, Pageable pageable);
}
```

### Example 2: Derived Query (Health Filter)
```java
@Repository
public interface FighterRepository extends JpaRepository<Fighter, Integer> {
    
    // Find fighters with health greater than a value
    Page<Fighter> findByHealthGreaterThan(int health, Pageable pageable);
}
```

### Example 3: @Query (Strongest Fighters)
```java
@Repository
public interface FighterRepository extends JpaRepository<Fighter, Integer> {
    
    // Get strongest fighters ordered by damage (descending)
    @Query("SELECT f FROM Fighter f ORDER BY f.damage DESC")
    Page<Fighter> findStrongestFighters(Pageable pageable);
}
```

### Example 4: @Query (Balanced Fighters)
```java
@Repository
public interface FighterRepository extends JpaRepository<Fighter, Integer> {
    
    // Find fighters with high health but controlled damage (good for beginners)
    @Query("SELECT f FROM Fighter f WHERE f.health >= ?1 AND f.damage <= ?2 ORDER BY f.resistance DESC")
    Page<Fighter> findBalancedFighters(int minHealth, double maxDamage, Pageable pageable);
}
```

---

## Common Naming Conventions for Derived Queries

| Keyword | SQL | Example |
|---------|-----|---------|
| `And` | AND | `findByNameAndHealth` |
| `Or` | OR | `findByNameOrHealth` |
| `Is, Equals` | = | `findByName` |
| `Between` | BETWEEN | `findByHealthBetween` |
| `LessThan` | < | `findByHealthLessThan` |
| `LessThanEqual` | <= | `findByHealthLessThanEqual` |
| `GreaterThan` | > | `findByHealthGreaterThan` |
| `GreaterThanEqual` | >= | `findByHealthGreaterThanEqual` |
| `After` | > (dates) | `findByCreatedAfter` |
| `Before` | < (dates) | `findByCreatedBefore` |
| `IsNull` | IS NULL | `findByNameIsNull` |
| `IsNotNull` | IS NOT NULL | `findByNameIsNotNull` |
| `Like` | LIKE | `findByNameLike` |
| `NotLike` | NOT LIKE | `findByNameNotLike` |
| `StartingWith` | LIKE prefix% | `findByNameStartingWith` |
| `EndingWith` | LIKE %suffix | `findByNameEndingWith` |
| `Containing` | LIKE %substring% | `findByNameContaining` |
| `Not` | != | `findByNameNot` |
| `In` | IN (...) | `findByIdIn` |
| `NotIn` | NOT IN (...) | `findByIdNotIn` |
| `True` | IS TRUE | `findByActivatedTrue` |
| `False` | IS FALSE | `findByActivatedFalse` |
| `IgnoreCase` | UPPER() | `findByNameIgnoreCase` |
| `OrderBy` | ORDER BY | `findAllOrderByNameAsc` |

---

## Tips & Best Practices

1. **Use Derived Queries** for simple, straightforward queries (faster to write, less error-prone)
2. **Use @Query** for complex queries with multiple conditions, aggregations, or custom logic
3. **Use Named Parameters** (@Param) instead of positional parameters (?1, ?2) for readability
4. **Always add Pageable** for queries that return large datasets
5. **Add @Transactional** when using @Modifying (UPDATE/DELETE)
6. **Test your queries** with actual data to ensure they work as expected

---

## Resources

- [Spring Data JPA Query Methods](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods)
- [Spring Data JPA @Query](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.at-query)
- [JPQL Guide](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#hql)
