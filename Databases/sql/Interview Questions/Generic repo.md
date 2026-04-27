```javascript
// ============================================================================
// IMPORTS
// ============================================================================
import { DataSource, QueryRunner, Column, Entity } from "typeorm";


// ============================================================================
// 1. GENERIC REPOSITORY INTERFACE
// ============================================================================
export interface GenericRepository<T> {
  create(item: T): Promise<T>;
  update(item: T): Promise<T>;
  delete(id: string | number): Promise<boolean>;
  deleteMultiple(ids: Array<string | number>): Promise<boolean>;

  findById(id: string | number): Promise<T | null>;
  findAll(): Promise<T[]>;
  findWhere(where: Partial<T>): Promise<T[]>;
  findOneWhere(where: Partial<T>): Promise<T | null>;
  count(where?: Partial<T>): Promise<number>;
  exists(where: Partial<T>): Promise<boolean>;

  save(item: T): Promise<T>;

  findPaged(
    where: Partial<T>,
    options: { skip?: number; take?: number; order?: any }
  ): Promise<{ data: T[]; total: number }>;

  withTransaction<R>(
    operation: (queryRunner: QueryRunner) => Promise<R>
  ): Promise<R>;

  createWithRunner(queryRunner: QueryRunner, table: string, item: T): Promise<T>;
  updateWithRunner(queryRunner: QueryRunner, table: string, item: T): Promise<T>;
  deleteWithRunner(queryRunner: QueryRunner, table: string, id: string | number): Promise<boolean>;

  createMultipleWithRunner(queryRunner: QueryRunner, table: string, items: T[]): Promise<T[]>;
  updateMultipleWithRunner(queryRunner: QueryRunner, table: string, items: T[]): Promise<T[]>;
  deleteMultipleWithRunner(queryRunner: QueryRunner, table: string, ids: Array<string | number>): Promise<boolean>;

  findByIdWithRunner(queryRunner: QueryRunner, table: string, id: string | number): Promise<T | null>;
  findWhereWithRunner(queryRunner: QueryRunner, table: string, where: Partial<T>): Promise<T[]>;
}


// ============================================================================
// 2. USER ENTITY DOMAIN MODEL
// ============================================================================
export class UserEntity {
  id?: number;
  email: string;
  password: string;
  username: string;
  walletAddress: string;
  createdAt?: Date;
  updatedAt?: Date;
}


// ============================================================================
// 3. DATABASE ENTITY (POSTGRES)
// ============================================================================
@Entity({ name: "users" })
export class UserPgSqlEntity {
  @Column({ type: "varchar", length: 255, unique: true })
  email: string;

  @Column({ type: "text" })
  password: string;

  @Column({ type: "varchar", length: 200 })
  username: string;

  @Column({ type: "varchar", length: 200 })
  walletAddress: string;
}


// ============================================================================
// 4. BASE REPOSITORY IMPLEMENTATION
// ============================================================================
export class BaseRepository<T> implements GenericRepository<T> {
  protected readonly tableName: string;

  constructor(
    protected readonly dataSource: DataSource,
    protected readonly entity: any,
  ) {
    this.tableName = dataSource.getMetadata(entity).tableName;
  }

  // -------------------- CRUD --------------------
  async create(item: T): Promise<T> {
    const keys = Object.keys(item);
    const values = Object.values(item);
    const cols = keys.join(", ");
    const params = keys.map((_, i) => `$${i + 1}`).join(", ");

    const result = await this.dataSource.query(
      `INSERT INTO ${this.tableName} (${cols}) VALUES (${params}) RETURNING *`,
      values
    );

    return result[0];
  }

  async update(item: any): Promise<T> {
    if (!item.id) throw new Error("ID required");

    const keys = Object.keys(item).filter(k => k !== "id");
    const values = keys.map(k => item[k]);
    values.push(item.id);

    const setClause = keys.map((k, i) => `${k}=$${i + 1}`).join(", ");

    const result = await this.dataSource.query(
      `UPDATE ${this.tableName} SET ${setClause} WHERE id=$${keys.length + 1} RETURNING *`,
      values
    );

    return result[0];
  }

  async delete(id: string | number): Promise<boolean> {
    await this.dataSource.query(`DELETE FROM ${this.tableName} WHERE id=$1`, [id]);
    return true;
  }

  async deleteMultiple(ids: (string | number)[]): Promise<boolean> {
    await this.dataSource.query(`DELETE FROM ${this.tableName} WHERE id = ANY($1)`, [ids]);
    return true;
  }

  // -------------------- LOOKUPS --------------------
  async findById(id: string | number): Promise<T | null> {
    const rows = await this.dataSource.query(
      `SELECT * FROM ${this.tableName} WHERE id=$1 LIMIT 1`,
      [id]
    );
    return rows[0] || null;
  }

  async findAll(): Promise<T[]> {
    return this.dataSource.query(`SELECT * FROM ${this.tableName}`);
  }

  async findWhere(where: Partial<T>): Promise<T[]> {
    const keys = Object.keys(where);
    const values = Object.values(where);
    const cond = keys.map((k, i) => `${k}=$${i + 1}`).join(" AND ");

    return this.dataSource.query(
      `SELECT * FROM ${this.tableName} WHERE ${cond}`,
      values
    );
  }

  async findOneWhere(where: Partial<T>): Promise<T | null> {
    const rows = await this.findWhere(where);
    return rows[0] || null;
  }

  async count(where?: Partial<T>): Promise<number> {
    if (!where) {
      const res = await this.dataSource.query(`SELECT COUNT(*) FROM ${this.tableName}`);
      return Number(res[0].count);
    }

    const keys = Object.keys(where);
    const values = Object.values(where);
    const cond = keys.map((k, i) => `${k}=$${i + 1}`).join(" AND ");

    const res = await this.dataSource.query(
      `SELECT COUNT(*) FROM ${this.tableName} WHERE ${cond}`,
      values
    );

    return Number(res[0].count);
  }

  async exists(where: Partial<T>): Promise<boolean> {
    return (await this.count(where)) > 0;
  }

  // -------------------- SAVE --------------------
  async save(item: any): Promise<T> {
    return item.id ? this.update(item) : this.create(item);
  }

  // -------------------- PAGINATION --------------------
  async findPaged(
    where: Partial<T>,
    options: { skip?: number; take?: number; order?: any }
  ): Promise<{ data: T[]; total: number }> {
    const keys = Object.keys(where);
    const values = Object.values(where);
    const cond = keys.length ? `WHERE ${keys.map((k, i) => `${k}=$${i + 1}`).join(" AND ")}` : "";

    const order = options.order
      ? `ORDER BY ${Object.entries(options.order).map(([k, v]) => `${k} ${v}`).join(", ")}`
      : "";

    const dataQuery = `
      SELECT * FROM ${this.tableName}
      ${cond}
      ${order}
      OFFSET ${options.skip || 0}
      LIMIT ${options.take || 20}
    `;

    const countQuery = `SELECT COUNT(*) FROM ${this.tableName} ${cond}`;

    const [data, count] = await Promise.all([
      this.dataSource.query(dataQuery, values),
      this.dataSource.query(countQuery, values),
    ]);

    return { data, total: Number(count[0].count) };
  }

  // -------------------- TRANSACTION --------------------
  async withTransaction<R>(operation: (qr: QueryRunner) => Promise<R>): Promise<R> {
    const qr = this.dataSource.createQueryRunner();
    await qr.connect();
    await qr.startTransaction();

    try {
      const result = await operation(qr);
      await qr.commitTransaction();
      return result;
    } catch (err) {
      await qr.rollbackTransaction();
      throw err;
    } finally {
      await qr.release();
    }
  }

  // -------------------- RUNNER METHODS --------------------
  async createWithRunner(qr: QueryRunner, table: string, item: any): Promise<T> {
    const keys = Object.keys(item);
    const values = Object.values(item);
    const cols = keys.join(", ");
    const params = keys.map((_, i) => `$${i + 1}`).join(", ");

    const res = await qr.query(
      `INSERT INTO ${table} (${cols}) VALUES (${params}) RETURNING *`,
      values
    );
    return res[0];
  }

  async updateWithRunner(qr: QueryRunner, table: string, item: any): Promise<T> {
    const keys = Object.keys(item).filter(k => k !== "id");
    const values = keys.map(k => item[k]);
    values.push(item.id);

    const set = keys.map((k, i) => `${k}=$${i + 1}`).join(", ");

    const res = await qr.query(
      `UPDATE ${table} SET ${set} WHERE id=$${keys.length + 1} RETURNING *`,
      values
    );
    return res[0];
  }

  async deleteWithRunner(qr: QueryRunner, table: string, id: string | number): Promise<boolean> {
    await qr.query(`DELETE FROM ${table} WHERE id=$1`, [id]);
    return true;
  }

  async createMultipleWithRunner(qr: QueryRunner, table: string, items: any[]) {
    const out = [];
    for (const item of items) out.push(await this.createWithRunner(qr, table, item));
    return out;
  }

  async updateMultipleWithRunner(qr: QueryRunner, table: string, items: any[]) {
    const out = [];
    for (const item of items) out.push(await this.updateWithRunner(qr, table, item));
    return out;
  }

  async deleteMultipleWithRunner(qr: QueryRunner, table: string, ids: any[]) {
    await qr.query(`DELETE FROM ${table} WHERE id = ANY($1)`, [ids]);
    return true;
  }

  async findByIdWithRunner(qr: QueryRunner, table: string, id: any) {
    const rows = await qr.query(`SELECT * FROM ${table} WHERE id=$1 LIMIT 1`, [id]);
    return rows[0] || null;
  }

  async findWhereWithRunner(qr: QueryRunner, table: string, where: any) {
    const keys = Object.keys(where);
    const values = Object.values(where);
    const cond = keys.map((k, i) => `${k}=$${i + 1}`).join(" AND ");

    return qr.query(`SELECT * FROM ${table} WHERE ${cond}`, values);
  }
}


// ============================================================================
// 5. USER REPOSITORY
// ============================================================================
export class UserRepository extends BaseRepository<UserEntity> {
  constructor(dataSource: DataSource) {
    super(dataSource, UserPgSqlEntity);
  }
}


// ============================================================================
// 6. USER SERVICE
// ============================================================================
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly dataSource: DataSource,
  ) {}

  async createUser(data: Partial<UserEntity>): Promise<UserEntity> {
    return this.userRepository.create(data as UserEntity);
  }

  async updateUser(id: number | string, data: Partial<UserEntity>): Promise<UserEntity> {
    const user = await this.userRepository.findById(id);
    if (!user) throw new Error("User not found");

    return this.userRepository.update({ ...user, ...data });
  }

  async deleteUser(id: number | string): Promise<boolean> {
    return this.userRepository.delete(id);
  }

  async getUserById(id: number | string) {
    return this.userRepository.findById(id);
  }

  async getAllUsers() {
    return this.userRepository.findAll();
  }

  async pagedUsers(where: any, skip = 0, take = 20) {
    return this.userRepository.findPaged(where, { skip, take });
  }

  // -------------------- Transaction Example --------------------
async createUserWithMultipleTables(data: Partial<UserEntity>) {
  return this.userRepository.withTransaction(async (qr) => {
    
    // 1. Create user (users table)
    const user = await this.userRepository.createWithRunner(
      qr,
      this.userRepository.tableName,
      data
    );

    // 2. Create wallet for that user (wallets table)
    await qr.query(
      `INSERT INTO wallets ("userId", balance) VALUES ($1, $2)`,
      [user.id, 0]
    );

    // 3. Create profile for that user (profiles table)
    await qr.query(
      `INSERT INTO profiles ("userId", bio, avatarUrl) VALUES ($1, $2, $3)`,
      [user.id, "New user", null]
    );

    // 4. If user has referralCode → update referral count (referrals table)
    if (data["referralCode"]) {
      await qr.query(
        `
        UPDATE referrals 
        SET count = count + 1 
        WHERE code = $1
        `,
        [data["referralCode"]]
      );
    }

    // 5. Write user creation log (audit_logs table)
    await qr.query(
      `INSERT INTO audit_logs ("userId", action) VALUES ($1, $2)`,
      [user.id, "USER_CREATED"]
    );

    // If everything succeeds → commit
    return user;
  });
}
```