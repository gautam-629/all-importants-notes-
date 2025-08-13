# 1: Filter a List of Users by Search Input

This example demonstrates how to filter a list of users based on a search input value using React with **TypeScript**, **useState**, **useEffect**, and **useRef** for debouncing.

---

## Users Data

```typescript
const users = [
  { id: 1, name: "Binod Gautam" },
  { id: 2, name: "Ram Prasad" },
  { id: 3, name: "Sita Kumari" },
];
````
```
```

## solutions
``` typescript

import React, { useEffect, useRef, useState } from "react";

const users = [
  { id: 1, name: "Binod Gautam" },
  { id: 2, name: "Ram Prasad" },
  { id: 3, name: "Sita Kumari" },
];

const App = () => {
  const [searchQuary, setSearchQuery] = useState<string>("");
  const [filterData, setFilterData] =
    useState<{ id: string; name: string }[]>(users);

  const timeOutRef = useRef<number | null>(null);

  // Filter function on button click
  const handleFilter = () => {
    const result = users.filter((user) =>
      user.name.toLowerCase().includes(searchQuary.toLowerCase())
    );
    setFilterData(result);
  };

  // Debounced filter on search input change
  useEffect(() => {
    if (timeOutRef.current) {
      clearTimeout(timeOutRef.current);
    }
    timeOutRef.current = setTimeout(() => {
      const result = users.filter((user) =>
        user.name.toLowerCase().includes(searchQuary.toLowerCase())
      );
      setFilterData(result);
    }, 1000);

    return () => clearTimeout(timeOutRef.current);
  }, [searchQuary]);

  return (
    <div className="flex justify-center items-center h-screen ">
      <div className="w-1/3 h-1/3 shadow-2xl rounded-2xl flex items-center justify-center flex-col">
        <div className="flex gap-1">
          <input
            className="bg-sky-200 rounded-md p-1"
            value={searchQuary}
            onChange={(e) => setSearchQuery(e.target.value)}
            type="text"
          />
          <button
            onClick={handleFilter}
            className="cursor-pointer bg-sky-300 px-1 rounded-md"
          >
            Search
          </button>
        </div>

        <ul>
          {filterData.map((value, index) => (
            <li className="p-1.5" key={value.id}>{`${index + 1}.${value.name}`}                </li>
          ))}
        </ul>
      </div>
    </div>
  );
};

export default App;
```

