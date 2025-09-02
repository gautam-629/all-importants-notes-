```sql
export interface PaginationMeta {
  currentPage: number;
  itemPerPage: number;
  totalItems: number;
  totalPages: number;
  hasNextPage: boolean;
  hasPrevious: boolean;
}

// Function to create PaginationMeta
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

// Example usage with MySQL2
import mysql from 'mysql2/promise';

async function fetchUsers(page: number, perPage: number) {
  const connection = await mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: 'yourpassword',
    database: 'yourdatabase',
  });

  const offset = (page - 1) * perPage;

  // Fetch paginated data
  const [rows] = await connection.execute(
    'SELECT * FROM users ORDER BY id LIMIT ? OFFSET ?',
    [perPage, offset]
  );

  // Fetch total count
  const [[{ total }]] = await connection.execute(
    'SELECT COUNT(*) AS total FROM users'
  );

  const meta = getPaginationMeta(page, perPage, total);

  return {
    data: rows,
    meta,
  };
}

// Example call
fetchUsers(2, 10).then(result => console.log(result));

```