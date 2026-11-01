# Ch22 – Maven, JPA / Hibernate, JPQL, Criteria Queries & Lombok

Educational project (Coding Factory, AUEB): a first hands-on look at ORM with JPA / Hibernate,
using a simple **Teacher – Course** model (one-to-many) on MySQL.
Covers Maven, `persistence.xml`, entities, `EntityManager`, JPQL, Criteria Queries, fetch strategies and Lombok.

**Technologies:** Java 21 · Maven · JPA / Hibernate · JPQL · Criteria Queries · MySQL · Lombok

## Getting started

1. Create an empty schema in MySQL (the name must match the one in `persistence.xml`).
2. Adjust the connection settings in `src/main/resources/META-INF/persistence.xml`.
3. Run `Main` once: with `hbm2ddl.auto` set to `update`, Hibernate creates the `teachers` and `courses` tables from the entities.
4. Run the scripts in `src/main/resources/db/` (teachers first, then courses) to populate the tables.
5. Run `Main` again. It is a scratchpad: it holds many JPQL, Criteria, `persist`/`merge`/`remove` examples as comments. Uncomment the one you want to try.

> The password in `persistence.xml` belongs to my local database (educational use only).

Theory notes (JDBC, JPA, Hibernate, JPQL, Criteria, etc.): [NOTES.md](NOTES.md)