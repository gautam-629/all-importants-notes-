```sql
import express from "express";
import { Pool } from "pg";

const app = express();
const port = 3000;

// PostgreSQL connection pool (Recommended)
const pool = new Pool({
  host: "localhost",
  user: "postgres",
  password: "yourpassword",
  database: "yourdatabase",
  port: 5432,
});

export interface PaginationMeta {
  currentPage: number;
  itemPerPage: number;
  totalItems: number;
  totalPages: number;
  hasNextPage: boolean;
  hasPrevious: boolean;
}

function getPaginationMeta(
  currentPage: number,
  itemPerPage: number,
  totalItems: number
): PaginationMeta {
  const totalPages = Math.ceil(totalItems / itemPerPage);

  return {
    currentPage,
    itemPerPage,
    totalItems,
    totalPages,
    hasNextPage: currentPage < totalPages,
    hasPrevious: currentPage > 1,
  };
}

// REST API Endpoint
app.get("/users", async (req, res) => {
  try {
    const page = Number(req.query.page) || 1;
    const limit = Number(req.query.limit) || 10;
    const search = req.query.search || null;

    const offset = (page - 1) * limit;

    // Filtered data query
    const dataQuery = `
      SELECT id, name, email, created_at
      FROM users
      WHERE ($1::text IS NULL OR name ILIKE '%' || $1 || '%')
      ORDER BY id DESC
      LIMIT $2 OFFSET $3
    `;

    const dataResult = await pool.query(dataQuery, [
      search,
      limit,
      offset,
    ]);

    // Total count query
    const countQuery = `
      SELECT COUNT(*) AS total
      FROM users
      WHERE ($1::text IS NULL OR name ILIKE '%' || $1 || '%')
    `;

    const countResult = await pool.query(countQuery, [search]);

    const totalItems = Number(countResult.rows[0].total);

    const meta = getPaginationMeta(page, limit, totalItems);

    res.json({
      data: dataResult.rows,
      meta,
    });
  } catch (error) {
    console.error(error);
    res.status(500).json({ message: "Server Error" });
  }
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```