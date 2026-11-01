# Σημειώσεις – JPA / Hibernate

## 1. Η εικόνα συνολικά

```
Main (ο κώδικάς μου)
   │  χρησιμοποιεί JPA API (EntityManager, JPQL, Criteria)
   ▼
Hibernate            ← υλοποίηση της JPA, παράγει SQL
   │  καλεί JDBC API (Connection, PreparedStatement, ResultSet)
   ▼
mysql-connector-j    ← JDBC driver, μιλάει το πρωτόκολλο της MySQL
   ▼
MySQL
```

## 2. JDBC, driver, JPA, Hibernate

| Έννοια | Τι είναι |
|--------|----------|
| **JDBC** | Το API της Java για βάσεις δεδομένων (`java.sql`). Είναι μόνο interfaces: `Connection`, `Statement`, `ResultSet`. Έρχεται με το JDK. |
| **JDBC driver** | Η υλοποίηση του JDBC για μία συγκεκριμένη βάση. Εδώ: `mysql-connector-j`. Χωρίς driver δεν υπάρχει σύνδεση. |
| **JPA** | Προδιαγραφή ORM (`jakarta.persistence`): annotations, `EntityManager`, JPQL, Criteria. Είναι μόνο interfaces και κανόνες. |
| **Hibernate** | Η υλοποίηση (provider) της JPA. Μετατρέπει entities και JPQL / Criteria σε SQL και το εκτελεί μέσω JDBC. |

Με λίγα λόγια:
- JPA : Hibernate = JDBC : mysql-connector-j (προδιαγραφή : υλοποίηση).
- Το Hibernate **χρησιμοποιεί** JDBC από κάτω, δεν το αντικαθιστά.
- Στο ch19 έγραφα εγώ το SQL και το JDBC. Εδώ το Hibernate τα κάνει όλα αυτά για μένα.

### Connection pool
- Το άνοιγμα σύνδεσης στη βάση είναι ακριβό, γι' αυτό χρησιμοποιείται pool (έτοιμες συνδέσεις που ξαναχρησιμοποιούνται).
- Το Hibernate έχει **απλό ενσωματωμένο pool**, μόνο για εκπαίδευση.
- Το **HikariCP** είναι το συνηθισμένο pool για πραγματικές εφαρμογές. Είναι ξεχωριστό dependency και το χρησιμοποιεί το Spring Boot by default.

## 3. Maven
- Εργαλείο **διαχείρισης dependencies** και **build**.
- Το `pom.xml` περιέχει: `groupId`, `artifactId`, `version`, dependencies, plugins, έκδοση Java.
- Οι βιβλιοθήκες κατεβαίνουν αυτόματα και φαίνονται στα **External Libraries**.

## 4. persistence.xml
- Βρίσκεται στο `META-INF/`.
- Ορίζει το **persistence unit** (εδώ `schoolPU`): driver, URL, user, password, dialect, `hbm2ddl.auto`, `show_sql`.
- Το όνομα του unit δίνεται στο `Persistence.createEntityManagerFactory("schoolPU")`.
- `hbm2ddl.auto = update`: δημιουργεί τους πίνακες αν λείπουν και δεν σβήνει δεδομένα. (`create` / `create-drop` θα τους ξαναέφτιαχναν σε κάθε εκτέλεση και θα χάνονταν τα δεδομένα.)
- Οι πίνακες δημιουργούνται από τα entities. Τα scripts στο `db/` τρέχουν μόνο για να τους γεμίσουν.

## 5. Entities
- `@Entity`, `@Table`, `@Id`, `@GeneratedValue(strategy = IDENTITY)`, `@Column`.
- Σχέση ένα-προς-πολλά:
  - `Teacher`: `@OneToMany(mappedBy = "teacher", fetch = LAZY)`
  - `Course`: `@ManyToOne(fetch = LAZY)` + `@JoinColumn(name = "teacher_id")`
- **Owning side** είναι η πλευρά με το foreign key (`Course`). Το `mappedBy` δηλώνει ότι το `Teacher` είναι η αντίστροφη πλευρά.
- Helper methods (`addCourse`, `removeCourse`) για να συγχρονίζονται και οι δύο πλευρές της σχέσης.

## 6. EntityManager
- `EntityManagerFactory`: ένα ανά εφαρμογή (ακριβό στη δημιουργία). Στη `Main` είναι `static final`.
- `EntityManager`: ένα ανά «μονάδα εργασίας».
- Βασικές μέθοδοι: `persist`, `find`, `merge`, `remove`, `createQuery`, `createNativeQuery`.
- Transactions: `tx.begin()` / `tx.commit()` / `tx.rollback()`.
- Μοτίβο: `try { begin ... commit } catch (RuntimeException) { rollback; throw } finally { em.close(); emf.close(); }`.

## 7. JPQL
- Ερωτήματα πάνω σε **entities και πεδία**, όχι σε πίνακες και στήλες.
- Named parameters (`:name`), `JOIN`, `ORDER BY`, aggregate functions.


## 8. Criteria Queries
- **Type-safe, προγραμματιστικός** τρόπος δημιουργίας queries.
- Βασικά αντικείμενα: `CriteriaBuilder`, `CriteriaQuery<T>`, `Root<T>` (αντίστοιχο του FROM), `Predicate`.
- Το `T` στο `CriteriaQuery<T>` είναι ό,τι επιστρέφει το query.
- Χρήσιμο όταν τα φίλτρα είναι **δυναμικά**.


## 9. Lombok
- Μειώνει τον boilerplate με annotations. Στο project χρησιμοποιούνται: `@Getter`, `@Setter`, `@NoArgsConstructor`.
- Με `AccessLevel` ρυθμίζω την ορατότητα: `@Getter(AccessLevel.PACKAGE)`, `@Setter(AccessLevel.PACKAGE)`, `@Setter(AccessLevel.NONE)`.
- Άλλα annotations που υπάρχουν (δεν χρησιμοποιούνται εδώ): `@AllArgsConstructor`, `@Builder`, `@ToString`.
- Θέλει enabled **annotation processing** στο IntelliJ.
- Προσοχή με `@ToString` σε σχέσεις διπλής κατεύθυνσης (άπειρη αναδρομή) → `@ToString.Exclude`. Εδώ το `toString` είναι γραμμένο με το χέρι.

## 10. Fetch: LAZY, N+1, JOIN FETCH
- Με `fetch = FetchType.LAZY` τα συσχετισμένα δεδομένα φορτώνονται μόνο όταν τα ζητήσω.
- **Πρόβλημα N+1:** `SELECT t FROM Teacher t` + πρόσβαση στα `courses` μέσα σε βρόχο = 1 query για τους teachers + N queries για τα courses.
- **Λύση:** `SELECT t FROM Teacher t LEFT JOIN FETCH t.courses` φέρνει τα πάντα με ένα query.
- Το απλό `JOIN` δεν φορτώνει τη συλλογή, μόνο φιλτράρει.

## 11. Lifecycle: persist, merge, remove
- `persist(entity)`: νέο αντικείμενο γίνεται managed και θα γίνει INSERT.
- **Dirty checking:** αν αλλάξω πεδίο σε managed entity (π.χ. από `find`), το Hibernate κάνει UPDATE στο commit χωρίς να καλέσω `merge`.
- `merge(entity)`: για detached αντικείμενα, αντιγράφει την κατάστασή τους σε managed.
- `remove(entity)`: διαγραφή. Στη σχέση πρώτα `teacher.removeCourse(course)` και μετά `em.remove(course)`.

## 12. Περισσότερα JPQL
- `LIKE 'Java%'`, `IN :list`, `ORDER BY`, `IS NULL` / `IS NOT NULL`.
- Πλοήγηση στη σχέση: `c.teacher.lastname`.
- `t.courses IS EMPTY`: teachers χωρίς courses (εναλλακτικά `LEFT JOIN ... WHERE c IS NULL`).
- `COUNT` + `GROUP BY`: το αποτέλεσμα είναι `List<Object[]>`.
- Native query: `em.createNativeQuery("SELECT * FROM teachers", Teacher.class)` (SQL πάνω σε πίνακες, όχι JPQL).
- Πάντα named parameters (`:lastname`) και όχι συνένωση string, για αποφυγή SQL injection.

## 13. Περισσότερα Criteria
- `cb.like`, `cb.isNotNull`, `path.in(...)`, `cb.parameter(...)` + `setParameter`.
- Πολλαπλά ορίσματα στο `where(...)` = AND.
- `t.join("courses")` για join, `distinct(true)`.
- `multiselect` + `groupBy` + `having(cb.gt(cb.count(...), 2L))` για aggregates.
- Προσοχή στον τύπο: αν επιλέγω ένα πεδίο (π.χ. `t.get("lastname")`) το `CriteriaQuery` πρέπει να είναι `CriteriaQuery<String>`. Αν επιλέγω πολλές τιμές (`multiselect`), πρέπει να είναι `CriteriaQuery<Object[]>`. Με `CriteriaQuery<Teacher>` δεν ταιριάζουν.

## 14. Στο μοντέλο
- Το `Course` έχει `equals` / `hashCode` με βάση το `title` (που είναι `unique`), ώστε το `Set<Course>` να μην έχει διπλότυπα.
- Το `Teacher.getAllCourses()` επιστρέφει `Collections.unmodifiableSet`, άρα οι αλλαγές γίνονται μόνο από τα `addCourse` / `removeCourse`, που συγχρονίζουν και τις δύο πλευρές.
- Το `Course.setTeacher` είναι `AccessLevel.PACKAGE`, άρα δεν το καλεί κανείς απ' έξω.
- Το `toString` είναι γραμμένο με το χέρι και όχι με Lombok `@ToString`, γι' αυτό δεν υπάρχει άπειρη αναδρομή.