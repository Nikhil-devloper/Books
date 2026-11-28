# Sequelize Interview Prep — ICICI Node.js Backend (First Technical Round)

Sequelize is **critical** for this JD. Relate answers to TypeORM where you have hands-on experience, but lead with Sequelize APIs.

---

### Q1. What is an ORM, and why use Sequelize in Node.js? `[Critical]`

**Answer:** An ORM (Object-Relational Mapper) maps database tables to objects/classes so you work with models and methods instead of writing every SQL statement by hand. Sequelize is a promise-based Node.js ORM for Postgres, MySQL, MariaDB, SQLite, and MSSQL. It gives models, associations, migrations, transactions, and hooks while still allowing raw SQL when needed.

**Example:**
```js
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize('bank_db', 'user', 'pass', {
  host: 'localhost',
  dialect: 'postgres', // or 'mysql'
  logging: false,
});

await sequelize.authenticate(); // verifies connection
```

**Follow-up:** Trade-offs vs raw SQL — productivity and safety vs less control over complex query plans; teams often mix both.

**TypeORM note:** Same idea — TypeORM uses entities/repositories; Sequelize uses models and static/instance methods. Both sit between app code and the RDBMS driver.

---

### Q2. Explain Sequelize architecture (connection, dialect, models, query interface). `[Critical]`

**Answer:** You create one `Sequelize` instance (connection manager + config). A **dialect** adapts SQL for Postgres/MySQL/etc. **Models** define table shape and become the API for queries. Under the hood, Sequelize builds SQL via its query generator and runs it through a connection from the pool. Associations, hooks, and transactions hang off models and the sequelize instance.

**Example:**
```js
// Typical project shape
// config/database.js  → new Sequelize(...)
// models/User.js      → sequelize.define('User', { ... })
// models/index.js     → import models, call associate(), export { sequelize, User, Account }
```

**Follow-up:** One Sequelize instance per database is normal; multiple instances if you talk to multiple DBs.

**TypeORM note:** TypeORM’s `DataSource` ≈ Sequelize instance; `Entity` ≈ model; repositories ≈ model query methods.

---

### Q3. How do you define a model in Sequelize? `[Critical]`

**Answer:** Use `sequelize.define` or a class extending `Model` with `Model.init`. You declare attributes (columns), types, constraints, and options like `tableName`, `timestamps`, and `paranoid`. Models should live in separate files and register associations in an `associate` helper.

**Example:**
```js
const { DataTypes, Model } = require('sequelize');

class User extends Model {}
User.init(
  {
    id: { type: DataTypes.UUID, defaultValue: DataTypes.UUIDV4, primaryKey: true },
    email: { type: DataTypes.STRING(255), allowNull: false, unique: true },
    status: { type: DataTypes.ENUM('ACTIVE', 'LOCKED'), defaultValue: 'ACTIVE' },
    balanceCents: { type: DataTypes.INTEGER, allowNull: false, defaultValue: 0, field: 'balance_cents' },
  },
  {
    sequelize,
    modelName: 'User',
    tableName: 'users',
    underscored: true, // created_at, updated_at
    timestamps: true,
  }
);
```

**Follow-up:** Prefer `field` / `underscored` to match existing DB naming without fighting SQL conventions.

**TypeORM note:** Closest to `@Entity()` + `@Column()` decorators, or `EntitySchema`. Attribute options map to column options (`nullable`, `unique`, `enum`, etc.).

---

### Q4. What are migrations in Sequelize, and why prefer them over sync? `[Critical]`

**Answer:** Migrations are versioned up/down scripts that change the schema in a controlled, repeatable way across environments. They are the production-safe way to create/alter tables. `sync` auto-creates or alters schema from models and is fine for local prototyping, but risky in production because it can drop or mutate columns unexpectedly.

**Example:**
```js
// migrations/202411280001-create-users.js
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('users', {
      id: { type: Sequelize.UUID, primaryKey: true },
      email: { type: Sequelize.STRING(255), allowNull: false, unique: true },
      created_at: { type: Sequelize.DATE, allowNull: false },
      updated_at: { type: Sequelize.DATE, allowNull: false },
    });
    await queryInterface.addIndex('users', ['email']);
  },
  async down(queryInterface) {
    await queryInterface.dropTable('users');
  },
};
```
```bash
npx sequelize-cli db:migrate
npx sequelize-cli db:migrate:undo
```

**Follow-up:** Always write a reversible `down` when possible; treat migrations as code review artifacts.

**TypeORM note:** Same role as TypeORM migrations (`migration:generate` / `migration:run`). Both teams should avoid `synchronize: true` / `sync({ force: true })` in prod.

---

### Q5. What does `sync` do? When is `force` / `alter` acceptable? `[High]`

**Answer:** `sequelize.sync()` creates missing tables from models. `sync({ alter: true })` tries to match columns to the model. `sync({ force: true })` drops and recreates tables — **data loss**. Use plain `sync` only in demos/tests; never `force`/`alter` against shared or production databases.

**Example:**
```js
// Dev/test only
await sequelize.sync();
// NEVER in production:
// await sequelize.sync({ force: true });
```

**Follow-up:** Interviewers often ask this to check production judgment — answer with “migrations in prod.”

**TypeORM note:** Equivalent warning for `synchronize: true`.

---

### Q6. How do you create a record? `[Critical]`

**Answer:** `Model.create(values)` inserts one row and returns the instance. Use `bulkCreate` for many rows. You can pass `fields` to whitelist columns and `returning: true` (Postgres) to get DB-generated values.

**Example:**
```js
const user = await User.create({
  email: 'dev@icici.example',
  status: 'ACTIVE',
});

const users = await User.bulkCreate(
  [
    { email: 'a@bank.com' },
    { email: 'b@bank.com' },
  ],
  { ignoreDuplicates: true, validate: true }
);
```

**Follow-up:** `build()` + `save()` is useful when you mutate before insert; `create` is the common path.

**TypeORM note:** `repository.save()` / `insert()` ≈ `create`/`bulkCreate`. TypeORM `save` can upsert; Sequelize uses `upsert` explicitly.

---

### Q7. How do you read records: findOne, findAll, findByPk? `[Critical]`

**Answer:** `findByPk(id)` loads by primary key. `findOne({ where })` returns the first match or `null`. `findAll({ where, order, limit, offset })` returns an array. Use `attributes` to select columns and `raw: true` if you want plain objects instead of model instances.

**Example:**
```js
const byId = await User.findByPk(userId);

const one = await User.findOne({
  where: { email: 'dev@icici.example' },
  attributes: ['id', 'email', 'status'],
});

const page = await User.findAll({
  where: { status: 'ACTIVE' },
  order: [['created_at', 'DESC']],
  limit: 20,
  offset: 40,
});
```

**Follow-up:** `findAndCountAll` is handy for pagination metadata (`count` + `rows`).

**TypeORM note:** `findOneBy` / `find` / `findOne({ where: { id } })` map closely; TypeORM’s QueryBuilder is closer to Sequelize’s more advanced `where` + `include` combinations.

---

### Q8. How do you update and delete? `[Critical]`

**Answer:** Instance methods: load then `user.update({...})` or `user.destroy()`. Static methods: `User.update(values, { where })` returns `[affectedCount]`; `User.destroy({ where })` returns deleted count. Always scope `where` carefully — an empty where can update/delete everything depending on version/config.

**Example:**
```js
const user = await User.findByPk(id);
await user.update({ status: 'LOCKED' });
await user.destroy(); // soft-delete if paranoid: true

// Static
await User.update(
  { status: 'LOCKED' },
  { where: { id } }
);

await User.destroy({ where: { id } });
```

**Follow-up:** Prefer instance updates when hooks/side effects must run on that row; batch static updates for bulk admin jobs.

**TypeORM note:** `repository.update` / `delete` / `softDelete` are the analogs; soft delete needs `@DeleteDateColumn` like Sequelize `paranoid`.

---

### Q9. What is `bulkCreate` and what pitfalls exist? `[High]`

**Answer:** `bulkCreate(rows, options)` inserts many rows in fewer round-trips. By default some hooks may not run the same way as `create`; set `individualHooks: true` if you need per-row hooks (slower). Use `updateOnDuplicate` (MySQL) or `conflictAttributes` / dialect-specific upsert options carefully; validate with `validate: true`.

**Example:**
```js
await Account.bulkCreate(rows, {
  validate: true,
  ignoreDuplicates: true,
  // individualHooks: true, // only if hooks are required
});
```

**Follow-up:** For huge imports, chunk batches (e.g. 500–1000) to avoid huge packets and long locks.

**TypeORM note:** Similar to `insert().values([...])` or multiple `save`; watch hook and subscriber differences on bulk paths.

---

### Q10. Explain associations: hasOne, belongsTo, hasMany, belongsToMany. `[Critical]`

**Answer:** Associations declare relationships so Sequelize can join and nest data. `hasOne`/`belongsTo` model 1:1 (FK usually on the `belongsTo` side). `hasMany`/`belongsTo` model 1:N. `belongsToMany` models M:N via a join table (`through`). Define both sides so includes and eager loading work cleanly.

**Example:**
```js
// 1:1 Profile
User.hasOne(Profile, { foreignKey: 'userId', as: 'profile' });
Profile.belongsTo(User, { foreignKey: 'userId', as: 'user' });

// 1:N Accounts
User.hasMany(Account, { foreignKey: 'userId', as: 'accounts' });
Account.belongsTo(User, { foreignKey: 'userId', as: 'owner' });

// M:N Roles
User.belongsToMany(Role, { through: 'user_roles', as: 'roles' });
Role.belongsToMany(User, { through: 'user_roles', as: 'users' });
```

**Follow-up:** Always set `as` aliases consistently and use the same alias in `include`.

**TypeORM note:** `@OneToOne`, `@ManyToOne`, `@OneToMany`, `@ManyToMany` + `@JoinTable` map 1:1 to these. FK ownership rules are the same idea.

---

### Q11. Eager loading vs lazy loading — what is `include`? `[Critical]`

**Answer:** Eager loading fetches associations in the same query (or controlled follow-up queries) via `include`. Lazy loading loads associations later (e.g. `user.getAccounts()`), which can cause N+1 if done in a loop. Prefer explicit `include` for known read paths; keep includes shallow and selective for performance.

**Example:**
```js
const user = await User.findByPk(id, {
  include: [
    { model: Profile, as: 'profile' },
    {
      model: Account,
      as: 'accounts',
      where: { status: 'OPEN' }, // inner join if required: true (default when where present)
      required: false, // LEFT OUTER JOIN even with where — be explicit
      attributes: ['id', 'accountNumber', 'balanceCents'],
    },
  ],
});
```

**Follow-up:** Nested `include` is powerful but expensive; cap depth and columns.

**TypeORM note:** `relations: ['accounts']` or `leftJoinAndSelect` ≈ `include`. Lazy relations in TypeORM have the same N+1 risk.

---

### Q12. How do you filter on associated models? `[High]`

**Answer:** Put `where` inside the nested `include`, or use `$association.column$` syntax in the top-level where (Sequelize nested syntax). Decide between INNER vs OUTER join with `required: true/false`. Mixing filters without understanding join type is a common bug source.

**Example:**
```js
const usersWithSalaryAccount = await User.findAll({
  include: [{
    model: Account,
    as: 'accounts',
    where: { type: 'SALARY' },
    required: true, // only users who have such an account
  }],
});
```

**Follow-up:** Pagination + filtered includes can inflate `count`; use separate count strategies or `distinct: true` with `findAndCountAll`.

**TypeORM note:** QueryBuilder `innerJoin` + `andWhere` on joined alias is clearer for complex filters than relation sugar.

---

### Q13. Managed vs unmanaged transactions in Sequelize. `[Critical]`

**Answer:** Managed: `sequelize.transaction(async (t) => { ... })` — auto-commit on success, rollback on throw. Unmanaged: `const t = await sequelize.transaction()` then you call `t.commit()` / `t.rollback()` yourself. Always pass `{ transaction: t }` into every query in the unit of work. Banking-style transfers need transactions plus proper isolation.

**Example:**
```js
// Managed (preferred)
await sequelize.transaction(async (t) => {
  const from = await Account.findByPk(fromId, {
    transaction: t,
    lock: t.LOCK.UPDATE, // SELECT FOR UPDATE
  });
  const to = await Account.findByPk(toId, { transaction: t, lock: t.LOCK.UPDATE });

  if (from.balanceCents < amount) throw new Error('INSUFFICIENT_FUNDS');

  await from.decrement('balanceCents', { by: amount, transaction: t });
  await to.increment('balanceCents', { by: amount, transaction: t });
  await Ledger.create({ fromId, toId, amount }, { transaction: t });
});

// Unmanaged
const t = await sequelize.transaction();
try {
  await Account.update({ status: 'FROZEN' }, { where: { id }, transaction: t });
  await t.commit();
} catch (e) {
  await t.rollback();
  throw e;
}
```

**Follow-up:** Mention isolation: `sequelize.transaction({ isolationLevel: Transaction.ISOLATION_LEVELS.SERIALIZABLE }, ...)`.

**TypeORM note:** `queryRunner.startTransaction()` ≈ unmanaged; `dataSource.transaction(async manager => ...)` ≈ managed.

---

### Q14. What are hooks (lifecycle events)? `[High]`

**Answer:** Hooks run before/after model lifecycle events: `beforeCreate`, `afterCreate`, `beforeUpdate`, `beforeDestroy`, `beforeSave`, etc. Use them for hashing passwords, audit fields, or invariants — but keep them thin; heavy side effects (emails, external APIs) belong in services so transactions stay predictable.

**Example:**
```js
User.beforeCreate(async (user) => {
  user.email = user.email.toLowerCase().trim();
});

User.afterUpdate(async (user, options) => {
  // Prefer emitting domain events outside DB hooks for reliability
});
```

**Follow-up:** Global hooks vs model hooks; `individualHooks` matters for bulk update/destroy.

**TypeORM note:** Entity listeners / subscribers (`@BeforeInsert`, `EntitySubscriberInterface`) play the same role.

---

### Q15. What is paranoid mode (soft deletes)? `[High]`

**Answer:** `paranoid: true` adds a `deletedAt` column. `destroy()` sets `deletedAt` instead of removing the row. Default finds exclude soft-deleted rows; use `paranoid: false` to include them. Soft delete preserves history but every unique constraint and reporting query must account for deleted rows.

**Example:**
```js
User.init(attributes, { sequelize, paranoid: true, deletedAt: 'deleted_at' });

await user.destroy(); // sets deleted_at
await User.findAll(); // omits soft-deleted
await User.findAll({ paranoid: false }); // includes them
await user.restore(); // clears deleted_at
```

**Follow-up:** Partial unique indexes (Postgres) often needed so soft-deleted emails can be reused.

**TypeORM note:** `@DeleteDateColumn()` enables soft delete; `withDeleted()` mirrors `paranoid: false`.

---

### Q16. How do you run raw SQL safely? `[High]`

**Answer:** Use `sequelize.query(sql, { replacements / bind, type })` for reports or dialect-specific SQL. Prefer bind/replacements over string concatenation to prevent SQL injection. Map results with `QueryTypes.SELECT` when you want rows.

**Example:**
```js
const { QueryTypes } = require('sequelize');

const rows = await sequelize.query(
  `SELECT status, COUNT(*)::int AS cnt
   FROM accounts
   WHERE user_id = :userId
   GROUP BY status`,
  {
    replacements: { userId },
    type: QueryTypes.SELECT,
  }
);
```

**Follow-up:** Raw SQL is fine for complex analytics; keep business writes in models/transactions when possible for consistency and hooks.

**TypeORM note:** `dataSource.query` / `manager.query` with parameters is the same pattern.

---

### Q17. How does connection pooling work in Sequelize? `[High]`

**Answer:** Sequelize maintains a pool of DB connections so requests reuse sockets instead of connecting per query. Tune `pool.max`, `min`, `acquire`, and `idle` for your Node process count and DB `max_connections`. Too-high `max` across many app instances exhausts the database.

**Example:**
```js
const sequelize = new Sequelize(url, {
  pool: {
    max: 10,
    min: 0,
    acquire: 30000, // ms to wait for a free connection
    idle: 10000,    // ms before releasing idle connection
  },
});
```

**Follow-up:** In serverless, pooling is harder — often use a small max or an external pooler (PgBouncer).

**TypeORM note:** `extra` / pool options on `DataSource` are equivalent; same capacity-planning conversation.

---

### Q18. How does Sequelize help prevent SQL injection? `[Critical]`

**Answer:** Query methods build parameterized SQL for `where`, binds, and replacements. Never concatenate user input into SQL strings or into `sequelize.literal` / `where` raw fragments. Treat `literal`, `where: sequelize.literal(...)`, and unsafe order/attribute strings as dangerous if they include request data.

**Example:**
```js
// Safe
await User.findAll({ where: { email: req.body.email } });
await sequelize.query('SELECT * FROM users WHERE id = $1', {
  bind: [req.params.id],
  type: QueryTypes.SELECT,
});

// Unsafe — do not do this
// await sequelize.query(`SELECT * FROM users WHERE email = '${req.body.email}'`);
// order: [req.query.sort]  // validate against an allowlist first
```

**Follow-up:** Also validate/allowlist `order`, `attributes`, and dynamic table/column names — parameterization does not cover identifiers.

**TypeORM note:** Same rule — QueryBuilder parameters are safe; string-built SQL is not.

---

### Q19. What is the N+1 problem in Sequelize, and how do you fix it? `[Critical]`

**Answer:** N+1 means 1 query for parents plus N queries for each parent’s children when you lazy-load in a loop. Fix with eager `include`, or one IN-query for children then map in memory. Avoid calling `getAssociation()` inside `map`/`for` over large lists.

**Example:**
```js
// Bad — N+1
const users = await User.findAll({ limit: 50 });
for (const u of users) {
  const accounts = await u.getAccounts(); // query per user
}

// Good — eager load
const users = await User.findAll({
  limit: 50,
  include: [{ model: Account, as: 'accounts' }],
});
```

**Follow-up:** `separate: true` on hasMany includes can help avoid cartesian products for multiple hasMany includes.

**TypeORM note:** Identical issue with lazy relations; fix with `relations` / joins or DataLoader-style batching.

---

### Q20. How do you optimize Sequelize queries? `[High]`

**Answer:** Select only needed `attributes`, index filter/join columns, paginate with keyset when possible, keep includes narrow, avoid huge `findAll` without limits, and inspect generated SQL (`logging: console.log` or slow-query logs). Push heavy aggregations to SQL (`fn`, `group`, raw queries) instead of loading rows into Node.

**Example:**
```js
const { fn, col } = require('sequelize');

const summary = await Account.findAll({
  attributes: ['status', [fn('COUNT', col('id')), 'cnt']],
  group: ['status'],
  raw: true,
});

// Keyset pagination (better than deep OFFSET)
const next = await Transaction.findAll({
  where: { id: { [Op.lt]: lastSeenId } },
  order: [['id', 'DESC']],
  limit: 50,
});
```

**Follow-up:** Watch cartesian explosions from multiple hasMany includes — split queries or use `separate: true`.

**TypeORM note:** QueryBuilder gives finer control; same indexing and pagination principles apply.

---

### Q21. How do you use operators (`Op`) in where clauses? `[High]`

**Answer:** Import `Op` for comparisons, lists, ranges, and logical combinations (`and`/`or`). This keeps filters composable and parameterized. Prefer `Op` over deprecated string operators.

**Example:**
```js
const { Op } = require('sequelize');

await User.findAll({
  where: {
    status: { [Op.in]: ['ACTIVE', 'LOCKED'] },
    createdAt: { [Op.gte]: startDate },
    [Op.or]: [
      { email: { [Op.iLike]: '%@icici.com' } },
      { phone: { [Op.ne]: null } },
    ],
  },
});
```

**Follow-up:** For Postgres case-insensitive search, `iLike` + appropriate index (e.g. `citext` or expression index) matters at scale.

**TypeORM note:** `In()`, `MoreThan()`, `Like()`, `Or()` from `typeorm` Find Operators are the parallel API.

---

### Q22. What is `upsert`, and when would you use it? `[Medium]`

**Answer:** `Model.upsert(values)` inserts a row or updates on conflict depending on dialect/unique keys. Useful for idempotent sync jobs. Behavior differs across MySQL/Postgres — know your dialect’s conflict target.

**Example:**
```js
await User.upsert({
  email: 'dev@icici.example',
  status: 'ACTIVE',
});
```

**Follow-up:** For financial ledgers, prefer explicit find + update inside a transaction over blind upserts.

**TypeORM note:** `upsert` / `orUpdate` in QueryBuilder is the counterpart; still dialect-sensitive.

---

### Q23. How do scopes work? `[Medium]`

**Answer:** Default and named scopes encapsulate reusable `where`/`include`/`attributes`. They keep controllers thin and enforce invariants (e.g. only non-deleted, only open accounts).

**Example:**
```js
Account.addScope('open', { where: { status: 'OPEN' } });
Account.addScope('withOwner', { include: [{ model: User, as: 'owner' }] });

await Account.scope(['open', 'withOwner']).findAll();
```

**Follow-up:** Don’t hide expensive includes in default scopes without the team knowing.

**TypeORM note:** No first-class scopes; teams use repository helper methods or query builders instead.

---

### Q24. How do validations and constraints differ? `[Medium]`

**Answer:** Model validations (e.g. `isEmail`, custom `validate`) run in the application before SQL. DB constraints (NOT NULL, UNIQUE, FK, CHECK) are enforced by the database and remain the source of truth under concurrency. Use both: validations for clear errors, constraints for integrity.

**Example:**
```js
email: {
  type: DataTypes.STRING,
  allowNull: false,
  unique: true,
  validate: { isEmail: true },
}
```

**Follow-up:** Catch unique constraint errors (`UniqueConstraintError`) and map to 409 responses.

**TypeORM note:** class-validator is often layered on entities; DB constraints still required.

---

### Q25. Virtual fields, getters/setters — when to use? `[Medium]`

**Answer:** Getters/setters format or protect fields (mask account numbers). `VIRTUAL` attributes are computed in JS and not stored. Don’t put heavy computation in getters that run on every serialization.

**Example:**
```js
accountNumberMasked: {
  type: DataTypes.VIRTUAL,
  get() {
    const n = this.getDataValue('accountNumber') || '';
    return n.length < 4 ? '****' : `****${n.slice(-4)}`;
  },
}
```

**Follow-up:** Prefer DTO/serializer layers for API responses in larger codebases.

**TypeORM note:** `@AfterLoad` or transformer columns fill a similar niche.

---

### Q26. How do you handle migrations for associations / FKs? `[High]`

**Answer:** Create tables first, then `addConstraint`/`addIndex` for FKs in migrations. For M:N, create the through table with a composite unique on the pair of FKs. Order migrations so referenced tables exist before dependents.

**Example:**
```js
await queryInterface.addConstraint('accounts', {
  fields: ['user_id'],
  type: 'foreign key',
  name: 'accounts_user_id_fkey',
  references: { table: 'users', field: 'id' },
  onDelete: 'RESTRICT',
  onUpdate: 'CASCADE',
});
```

**Follow-up:** Banking schemas usually prefer `ON DELETE RESTRICT` over `CASCADE` for money-related tables.

**TypeORM note:** FK onDelete/onUpdate options are declared in relation decorators and must still be reflected in migrations for production.

---

### Q27. Instance vs class (static) methods — practical difference? `[Medium]`

**Answer:** Class methods operate on the table (`User.findAll`). Instance methods operate on one row (`user.update`, `user.getAccounts`). Custom class methods encapsulate queries; custom instance methods encapsulate row behavior. Keep domain services above both for complex workflows.

**Example:**
```js
User.prototype.canTransact = function () {
  return this.status === 'ACTIVE';
};

User.findActiveByEmail = function (email) {
  return this.findOne({ where: { email, status: 'ACTIVE' } });
};
```

**Follow-up:** Prefer composition (services) over fat models as the codebase grows.

**TypeORM note:** Custom repository methods ≈ class methods; entity methods ≈ instance methods.

---

### Q28. How do you unit/integration test Sequelize code? `[Medium]`

**Answer:** Use a real test DB (Docker Postgres/MySQL) with migrations, wrap tests in transactions and rollback, or truncate tables between cases. Mocking the ORM often tests nothing useful. Assert SQL side effects and constraint failures, not only mock calls.

**Example:**
```js
beforeEach(async () => {
  await sequelize.truncate({ cascade: true, restartIdentity: true });
});
```

**Follow-up:** Parallel Jest workers need separate DBs/schemas to avoid cross-talk.

**TypeORM note:** Same strategy with test DataSource and migrations/`synchronize` only in ephemeral test DBs.

---

### Q29. Sequelize + Express: where should DB logic live? `[High]`

**Answer:** Controllers handle HTTP; services own transactions and business rules; models own persistence shape and simple queries. Pass `req` data as validated DTOs into services. Never start overlapping transactions per request without a clear boundary.

**Example:**
```js
// services/transferService.js
async function transfer({ fromId, toId, amount }) {
  return sequelize.transaction(async (t) => {
    /* lock rows, debit/credit, write ledger */
  });
}
```

**Follow-up:** Idempotency keys for payment APIs belong in the service + unique DB constraint.

**TypeORM note:** Same layered architecture; swap repository/EntityManager into the service.

---

### Q30. Quick mental map: Sequelize ↔ TypeORM for the interview. `[Critical]`

**Answer:** Be ready to say: “I’ve used TypeORM heavily; Sequelize concepts map cleanly — DataSource↔Sequelize, Entity↔Model, relations↔associations, QueryBuilder↔Op/include/raw query, subscribers↔hooks, soft delete columns↔paranoid.” Then answer in Sequelize method names from this sheet. Interviewers care that you can be productive in Sequelize quickly, not that you’ve never seen another ORM.

**Example:**
| Concern        | Sequelize              | TypeORM                 |
|----------------|------------------------|-------------------------|
| Connection     | `new Sequelize`        | `DataSource`            |
| Schema in code | `Model.init` / define  | `@Entity`               |
| Create         | `Model.create`         | `repo.save` / `insert`  |
| Read           | `findOne`/`findAll`    | `findOne`/`find`        |
| Relations      | `include`              | `relations` / joins     |
| Txn            | `sequelize.transaction`| `dataSource.transaction`|
| Soft delete    | `paranoid`             | `DeleteDateColumn`      |
| Migrations     | sequelize-cli          | typeorm migration CLI   |

**Follow-up:** Offer to implement a sample model + migration + transactional transfer in Sequelize as a take-home signal.

**TypeORM note:** This question *is* the TypeORM bridge — lead with mapping, finish with Sequelize fluency.

---

## Cheat sheet — commands & snippets

```bash
npx sequelize-cli init
npx sequelize-cli model:generate --name User --attributes email:string
npx sequelize-cli migration:generate --name add-status-to-users
npx sequelize-cli db:migrate
npx sequelize-cli db:seed:all
```

```js
const { Op, fn, col, literal, Transaction } = require('sequelize');
```

**Production red flags to mention:** `sync({ force: true })`, string-built SQL, missing transactions on multi-write flows, unbounded `findAll`, N+1 lazy loads, pool `max` too high × replica count.
