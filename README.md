# Lab 4 - Advanced Spring Data JPA (Custom Queries & Dynamic Pagination)

## Course Information

* **Course:** CPAN 228
* **Topic:** Custom Repository Queries & Pagination with Spring Data JPA

---

## Overview

In Lab 3 you implemented basic CRUD operations using Spring Data JPA. Now you'll take it further by implementing **custom repository methods** using both derived queries and **@Query** annotations. You'll then refactor the FighterController to support pagination and multi-criteria sorting, and finally update the Fighters.html template with search, sorting, and pagination controls. You can use the Players.html page as a reference!

---

## Getting Started

### GitHub Setup

1. **Fork the Repository**
   - Go to the repository on GitHub and click the **Fork** button in the top-right corner

2. **Clone Your Fork**
   ```bash
   git clone https://github.com/YOUR-USERNAME/Week-6-Intro-to-Spring-Data-JPA.git
   cd Week-6-Intro-to-Spring-Data-JPA
   ```

3. **Add Upstream Remote**
   ```bash
   git remote add upstream https://github.com/Christin-Classrooms/Week-6-Intro-to-Spring-Data-JPA.git
   ```

4. **Pull Latest Changes**
   ```bash
   git pull upstream main
   ```

5. **Create a Feature Branch**
   ```bash
   git checkout -b feature/lab4-yourname
   ```
   Replace `yourname` with your actual name (e.g., `feature/lab4-john-doe`)

---

## Lab 4 Assignment (Code similar to what we did for the Players page in class!)

### Part 1 — Add Custom Query Methods to FighterRepository

Add the following custom methods to your `FighterRepository` interface. These will allow filtering and searching fighters by different criteria.

#### Derived Query Methods (use Spring Data method naming convention):

1. **`findByNameContainingIgnoreCase(String name, Pageable pageable)`**
   - Returns fighters whose name contains the search term (case-insensitive)
   - Example: searching "Kaz" returns "Kazuya"
   - Returns a `Page<Fighter>`

2. **`findByHealthGreaterThan(int health, Pageable pageable)`**
   - Returns fighters with health greater than the specified value
   - Example: searching health > 1200 returns high-health fighters
   - Returns a `Page<Fighter>`

#### @Query Methods (use JPQL):

3. **`findStrongestFighters(Pageable pageable)`**
   - Use a custom JPQL query: `SELECT f FROM Fighter f ORDER BY f.damage DESC`
   - Returns the strongest fighters ranked by damage (descending)
   - Returns a `Page<Fighter>`

4. **`findBalancedFighters(double minHealth, double maxDamage, Pageable pageable)`**
   - Use a custom JPQL query with two parameters: `SELECT f FROM Fighter f WHERE f.health >= ?1 AND f.damage <= ?2 ORDER BY f.resistance DESC`
   - Finds fighters with high health but controlled damage (good for beginners)
   - Returns a `Page<Fighter>`

---

### Part 2 — Refactor FighterController for Pagination & Sorting

Update the `FighterController` to handle:

- **Pagination**: `@RequestParam` for `page` (default: 0) and `size` (default: 10)
- **Sorting**: `@RequestParam` for `sort` (default: "id") and `direction` (default: "ASC")
  - Allow sorting by: `id`, `name`, `health`, `damage`, `resistance`
- **Search/Filter**: `@RequestParam` for `search` (optional, partial name match)
- **Filter Type**: `@RequestParam` for `filterType` to choose which custom query to use:
  - `name` → Use `findByNameContainingIgnoreCase()`
  - `health` → Use `findByHealthGreaterThan()`
  - `strongest` → Use `findStrongestFighters()`
  - `balanced` → Use `findBalancedFighters()`
  - Default: Show all fighters with `findAll(pageable)`

Pass these attributes to the model:
- `fighters` (List of fighters on current page)
- `totalPages`, `totalElements`, `currentPage`, `pageSize`
- `hasPrevious`, `hasNext`
- `search`, `sort`, `direction`, `filterType`

---

### Part 3 — Update Fighters.html Template

Update the Fighters.html template to add the following features. **You can use Players.html as a reference!**

1. **Search/Filter Form** at the top with:
   - Text input for name search
   - Dropdown to select filter type (All, By Name, By Health, Strongest, Balanced)
   - Submit and Clear buttons

2. **Sortable Table Headers** for Name, Health, Damage, and Resistance
   - Make headers clickable links that sort by that column
   - Toggle between ASC/DESC on click
   - Display arrow indicators (↑ or ↓) to show current sort direction

3. **Pagination Controls** (similar to Players page)
   - First, Previous, Page numbers, Next, Last buttons
   - Display "Page X of Y"

Make sure all Thymeleaf variable names match the attributes you're sending from the controller!

---

## Validation Requirements (unchanged)

| Field | Rule |
|---|---|
| `name` | Required, not blank |
| `health` | Must be > 1000 and < 1500 |
| `damage` | Must be < 100 |
| `resistance` | Must be between 0.0 and 10.0 (double) |

---

## Testing Your Work

After completing **Parts 1, 2 & 3**, verify each of the following manually:

1. **Create** — Submit the form with valid data and confirm the fighter appears in the list
2. **Create (invalid)** — Submit with bad data and confirm errors appear and nothing is saved
3. **Pagination** — Navigate through multiple pages and verify data loads correctly
4. **Sorting** — Click column headers and verify fighters are sorted correctly (ASC/DESC toggle works)
5. **Search by Name** — Enter a partial fighter name in the search box, click Search, and confirm results are filtered
6. **Filter by Health** — Select "Filter by Health" dropdown option, submit, and verify only fighters above health 1200 appear
7. **Strongest Fighters** — Select "Strongest Fighters" option and verify fighters are ranked by damage (descending)
8. **Balanced Fighters** — Select "Balanced Fighters" option and verify fighters with high health and low damage appear
   
---

## Development Workflow

```bash
# Run the app
mvn spring-boot:run

# Commit your changes
git add .
git commit -m "Lab 4: Implement custom repository queries with pagination and sorting"

# Push to your fork
git push origin feature/lab4-yourname
```

Then open a Pull Request and submit the link on BlackBoard.

---

## Resources

* [Custom Queries Cheatsheet](CustomQueries_CHEATSHEET.md)
* [Thymeleaf Cheat Sheet](THYMELEAF_CHEATSHEET.md)
* [Spring Data JPA Docs](https://spring.io/projects/spring-data-jpa)
* [Spring Boot Reference — Data](https://docs.spring.io/spring-boot/docs/current/reference/html/data.html)

