
## 1. State vs Props

| Feature           | Props                          | State                                       |
| ----------------- | ------------------------------ | ------------------------------------------- |
| Mutability        | Immutable                      | Mutable                                     |
| Scope             | Passed from parent             | Local to the component                      |
| Update mechanism  | Cannot be updated by component | Updated using `useState` or `this.setState` |
| Re-render trigger | Yes (when parent re-renders)   | Yes (when state changes)                    |

### Complete Example: Props vs State

```jsx
// Parent Component - Demonstrates Props
import React, { useState } from 'react';

const Parent = () => {
  const [parentCount, setParentCount] = useState(0);
  const userName = "John Doe";

  return (
    <div>
      <h2>Parent Component</h2>
      <button onClick={() => setParentCount(parentCount + 1)}>
        Update Parent Count: {parentCount}
      </button>
      
      {/* Passing props to child */}
      <Child 
        name={userName} 
        count={parentCount} 
        onUpdate={() => setParentCount(parentCount + 1)}
      />
    </div>
  );
};

// Child Component - Demonstrates Props and State
const Child = ({ name, count, onUpdate }) => {
  const [childState, setChildState] = useState(0);

  return (
    <div style={{ border: '1px solid #ccc', margin: '10px', padding: '10px' }}>
      <h3>Child Component</h3>
      
      {/* Using Props (immutable) */}
      <p>Name from Props: {name}</p>
      <p>Count from Props: {count}</p>
      <button onClick={onUpdate}>Update Parent from Child</button>
      
      {/* Using State (mutable) */}
      <p>Child's Own State: {childState}</p>
      <button onClick={() => setChildState(childState + 1)}>
        Update Child State
      </button>
    </div>
  );
};

export default Parent;
```

## 2. Component Lifecycle

Component lifecycle refers to the sequence of methods that are invoked at different stages of a component's existence - from its creation (mounting), updates (updating), and removal from the DOM (unmounting).

### Function Component Lifecycle with Hooks

React hooks like `useEffect`, `useState`, etc., allow you to handle lifecycle events in function components.

```jsx
import React, { useState, useEffect } from 'react';

const LifecycleExample = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [count, setCount] = useState(0);

  // 🔵 componentDidMount equivalent
  useEffect(() => {
    console.log("Component mounted");
    
    // Setup: API call, subscriptions, timers
    const timer = setInterval(() => {
      console.log("Timer tick");
    }, 1000);

    // 🔴 componentWillUnmount equivalent (cleanup)
    return () => {
      console.log("Component unmounted - cleaning up");
      clearInterval(timer);
    };
  }, []); // Empty dependency array = runs once

  // 🟡 componentDidUpdate equivalent - runs on specific prop/state changes
  useEffect(() => {
    console.log("Component updated due to userId change");
    
    // Fetch user data when userId changes
    if (userId) {
      fetch(`/api/users/${userId}`)
        .then(res => res.json())
        .then(userData => setUser(userData));
    }
  }, [userId]); // Runs when userId changes

  // 🟠 componentDidUpdate for count changes
  useEffect(() => {
    console.log("Count updated:", count);
    document.title = `Count: ${count}`;
  }, [count]);

  return (
    <div>
      <h2>Lifecycle Example</h2>
      <p>User: {user ? user.name : 'Loading...'}</p>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

// Usage Example
const App = () => {
  const [showComponent, setShowComponent] = useState(true);
  const [userId, setUserId] = useState(1);

  return (
    <div>
      <button onClick={() => setShowComponent(!showComponent)}>
        {showComponent ? 'Unmount' : 'Mount'} Component
      </button>
      
      <button onClick={() => setUserId(userId + 1)}>
        Change User ID: {userId}
      </button>

      {showComponent && <LifecycleExample userId={userId} />}
    </div>
  );
};
```

### Why Cleanup Function is Required

If you start something like a timer, subscription (API, socket), or event listener, and don't stop it when the component unmounts, it keeps running in the background, causing:

1. **Memory leaks**
2. **Network issues**
3. **Duplicate event handling**
4. **Performance degradation**
5. **Crashes**

```jsx
// ❌ Bad - No cleanup
useEffect(() => {
  const subscription = websocket.subscribe(handleMessage);
  const timer = setInterval(updateData, 1000);
  window.addEventListener('resize', handleResize);
  // These keep running even after component unmounts!
}, []);

// ✅ Good - With cleanup
useEffect(() => {
  const subscription = websocket.subscribe(handleMessage);
  const timer = setInterval(updateData, 1000);
  window.addEventListener('resize', handleResize);

  return () => {
    subscription.unsubscribe();
    clearInterval(timer);
    window.removeEventListener('resize', handleResize);
  };
}, []);
```

## 3. Optimize React Applications

Optimizing a React application is essential to ensure fast load times, smooth interactions, and scalability as your app grows.

### 1. Use React.memo for Component Memoization

Prevent unnecessary re-renders of functional components when props haven't changed.

```jsx
import React, { useState, memo } from 'react';

// ❌ Without React.memo - re-renders on every parent update
const ExpensiveChild = ({ name, data }) => {
  console.log("ExpensiveChild rendered");
  
  // Expensive computation
  const processedData = data.map(item => ({
    ...item,
    processed: item.value * 2
  }));

  return (
    <div>
      <h3>Hello {name}</h3>
      <ul>
        {processedData.map(item => (
          <li key={item.id}>{item.processed}</li>
        ))}
      </ul>
    </div>
  );
};

// ✅ With React.memo - only re-renders when props change
const OptimizedChild = memo(({ name, data }) => {
  console.log("OptimizedChild rendered");
  
  const processedData = data.map(item => ({
    ...item,
    processed: item.value * 2
  }));

  return (
    <div>
      <h3>Hello {name}</h3>
      <ul>
        {processedData.map(item => (
          <li key={item.id}>{item.processed}</li>
        ))}
      </ul>
    </div>
  );
});

// Parent component
const MemoExample = () => {
  const [count, setCount] = useState(0);
  const [name] = useState("John");
  const [data] = useState([
    { id: 1, value: 10 },
    { id: 2, value: 20 },
    { id: 3, value: 30 }
  ]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      
      <ExpensiveChild name={name} data={data} />
      <OptimizedChild name={name} data={data} />
    </div>
  );
};
```

### 2. Use useMemo and useCallback

#### useMemo: Memoizes calculated values

```jsx
import React, { useState, useMemo } from 'react';

const UseMemoExample = () => {
  const [users, setUsers] = useState([
    { id: 1, name: 'Alice Johnson', age: 28 },
    { id: 2, name: 'Bob Smith', age: 34 },
    { id: 3, name: 'Charlie Brown', age: 22 },
    { id: 4, name: 'Diana Prince', age: 31 }
  ]);
  const [search, setSearch] = useState('');
  const [count, setCount] = useState(0);

  // ❌ Without useMemo - filters on every render
  const expensiveFilter = users.filter(user => {
    console.log("Filtering users...");
    return user.name.toLowerCase().includes(search.toLowerCase());
  });

  // ✅ With useMemo - only filters when users or search changes
  const filteredUsers = useMemo(() => {
    console.log("useMemo: Filtering users...");
    return users.filter(user => 
      user.name.toLowerCase().includes(search.toLowerCase())
    );
  }, [users, search]);

  const expensiveCalculation = useMemo(() => {
    console.log("Expensive calculation running...");
    return users.reduce((sum, user) => sum + user.age, 0) / users.length;
  }, [users]);

  return (
    <div>
      <input
        type="text"
        placeholder="Search users..."
        value={search}
        onChange={(e) => setSearch(e.target.value)}
      />
      
      <button onClick={() => setCount(count + 1)}>
        Unrelated Count: {count}
      </button>

      <p>Average Age: {expensiveCalculation}</p>
      
      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>{user.name} - {user.age}</li>
        ))}
      </ul>
    </div>
  );
};
```

#### useCallback: Memoizes functions

Problem: Passing functions as props causing unnecessary child re-renders

```jsx
import React, { useState, useCallback, memo } from 'react';

// Child component that receives a function prop
const Button = memo(({ onClick, children }) => {
  console.log(`Button "${children}" rendered`);
  return <button onClick={onClick}>{children}</button>;
});

const UseCallbackExample = () => {
  const [count, setCount] = useState(0);
  const [todos, setTodos] = useState(['Learn React', 'Build App']);

  // ❌ Without useCallback - new function on every render
  const handleCountClick = () => {
    setCount(count + 1);
  };

  const handleResetClick = () => {
    setCount(0);
  };

  // ✅ With useCallback - same function reference
  const handleCountClickOptimized = useCallback(() => {
    setCount(prev => prev + 1);
  }, []); // No dependencies needed with functional update

  const handleResetClickOptimized = useCallback(() => {
    setCount(0);
  }, []);

  const addTodo = useCallback((todo) => {
    setTodos(prev => [...prev, todo]);
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      
      {/* These will cause unnecessary re-renders */}
      <Button onClick={handleCountClick}>
        Increment (Not Optimized)
      </Button>
      
      <Button onClick={handleResetClick}>
        Reset (Not Optimized)
      </Button>

      {/* These won't cause unnecessary re-renders */}
      <Button onClick={handleCountClickOptimized}>
        Increment (Optimized)
      </Button>
      
      <Button onClick={handleResetClickOptimized}>
        Reset (Optimized)
      </Button>

      <TodoList todos={todos} onAdd={addTodo} />
    </div>
  );
};

const TodoList = memo(({ todos, onAdd }) => {
  console.log("TodoList rendered");
  
  return (
    <div>
      <h3>Todos:</h3>
      <ul>
        {todos.map((todo, index) => (
          <li key={index}>{todo}</li>
        ))}
      </ul>
      <button onClick={() => onAdd(`Todo ${todos.length + 1}`)}>
        Add Todo
      </button>
    </div>
  );
});
```

### 3. Code Splitting with React Lazy + Suspense
Load components only when needed to reduce initial bundle size.
**Code splitting** is a technique to **divide your JavaScript bundle** into smaller pieces ("chunks") so that your app only loads the code it **needs right now**, rather than loading _everything_ at once when the page first loads.

```jsx
import React, { Suspense, lazy, useState } from 'react';

// ✅ Lazy load heavy components
const HeavyComponent = lazy(() => import('./HeavyComponent'));
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));

// Loading component
const LoadingSpinner = () => (
  <div style={{ textAlign: 'center', padding: '20px' }}>
    <div>Loading...</div>
  </div>
);

const CodeSplittingExample = () => {
  const [currentView, setCurrentView] = useState('home');

  const renderView = () => {
    switch (currentView) {
      case 'heavy':
        return (
          <Suspense fallback={<LoadingSpinner />}>
            <HeavyComponent />
          </Suspense>
        );
      case 'dashboard':
        return (
          <Suspense fallback={<LoadingSpinner />}>
            <Dashboard />
          </Suspense>
        );
      case 'settings':
        return (
          <Suspense fallback={<LoadingSpinner />}>
            <Settings />
          </Suspense>
        );
      default:
        return <div>Welcome to the Home page!</div>;
    }
  };

  return (
    <div>
      <nav>
        <button onClick={() => setCurrentView('home')}>Home</button>
        <button onClick={() => setCurrentView('dashboard')}>Dashboard</button>
        <button onClick={() => setCurrentView('heavy')}>Heavy Component</button>
        <button onClick={() => setCurrentView('settings')}>Settings</button>
      </nav>
      
      <main>
        {renderView()}
      </main>
    </div>
  );
};

// Example of a heavy component that should be lazy loaded
const HeavyComponentContent = () => {
  // Simulate heavy computation
  const heavyData = Array.from({ length: 1000 }, (_, i) => ({
    id: i,
    value: Math.random() * 1000
  }));

  return (
    <div>
      <h2>Heavy Component Loaded!</h2>
      <p>This component contains {heavyData.length} items</p>
      <ul style={{ maxHeight: '200px', overflow: 'auto' }}>
        {heavyData.slice(0, 10).map(item => (
          <li key={item.id}>Item {item.id}: {item.value.toFixed(2)}</li>
        ))}
      </ul>
    </div>
  );
};

export default CodeSplittingExample;
```

### 4. Avoid Anonymous Functions in JSX (where possible)

```jsx
// ❌ Creates new function on every render
const BadExample = ({ items }) => (
  <ul>
    {items.map(item => (
      <li key={item.id}>
        <button onClick={() => handleClick(item.id)}>
          {item.name}
        </button>
      </li>
    ))}
  </ul>
);

// ✅ Better - extract to separate component
const ListItem = memo(({ item, onItemClick }) => (
  <li>
    <button onClick={() => onItemClick(item.id)}>
      {item.name}
    </button>
  </li>
));

const GoodExample = ({ items }) => {
  const handleItemClick = useCallback((id) => {
    console.log('Clicked item:', id);
  }, []);

  return (
    <ul>
      {items.map(item => (
        <ListItem 
          key={item.id} 
          item={item} 
          onItemClick={handleItemClick}
        />
      ))}
    </ul>
  );
};
```

### 5. Use Pagination or Infinite Scroll for Large Data Lists

```jsx
import React, { useState, useMemo } from 'react';

const PaginationExample = () => {
  const [currentPage, setCurrentPage] = useState(1);
  const [itemsPerPage] = useState(10);
  
  // Large dataset
  const allItems = Array.from({ length: 1000 }, (_, i) => ({
    id: i + 1,
    name: `Item ${i + 1}`,
    description: `Description for item ${i + 1}`
  }));

  // Memoized pagination calculation
  const paginatedData = useMemo(() => {
    const startIndex = (currentPage - 1) * itemsPerPage;
    const endIndex = startIndex + itemsPerPage;
    return allItems.slice(startIndex, endIndex);
  }, [currentPage, itemsPerPage, allItems]);

  const totalPages = Math.ceil(allItems.length / itemsPerPage);

  return (
    <div>
      <h2>Paginated List ({allItems.length} items)</h2>
      
      {/* Only render current page items */}
      <ul>
        {paginatedData.map(item => (
          <li key={item.id}>
            <strong>{item.name}</strong>: {item.description}
          </li>
        ))}
      </ul>

      {/* Pagination controls */}
      <div style={{ margin: '20px 0' }}>
        <button 
          disabled={currentPage === 1}
          onClick={() => setCurrentPage(prev => prev - 1)}
        >
          Previous
        </button>
        
        <span style={{ margin: '0 10px' }}>
          Page {currentPage} of {totalPages}
        </span>
        
        <button 
          disabled={currentPage === totalPages}
          onClick={() => setCurrentPage(prev => prev + 1)}
        >
          Next
        </button>
      </div>
    </div>
  );
};
```

### 6. Remove Unused State or Props

```jsx
// ❌ Bad - unused state and props
const BadComponent = ({ userName, userId, userEmail, theme }) => {
  const [count, setCount] = useState(0);
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [cache, setCache] = useState({});

  // Only using userName and count
  return (
    <div>
      <h2>Hello {userName}</h2>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

// ✅ Good - only necessary state and props
const GoodComponent = ({ userName }) => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h2>Hello {userName}</h2>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

## 4. State Management

### 1. Context API

No extra dependencies; avoids prop-drilling. Built-in and easy to set up—ideal for simple, scoped global states like theme, auth, or language.

```jsx
import React, { createContext, useContext, useState } from 'react';

// Create contexts
const ThemeContext = createContext();
const AuthContext = createContext();

// Theme Provider
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Auth Provider
const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);

  const login = async (credentials) => {
    setIsLoading(true);
    try {
      // Simulate API call
      const userData = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify(credentials)
      }).then(res => res.json());
      
      setUser(userData);
    } catch (error) {
      console.error('Login failed:', error);
    } finally {
      setIsLoading(false);
    }
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ 
      user, 
      isLoading, 
      login, 
      logout,
      isAuthenticated: !!user 
    }}>
      {children}
    </AuthContext.Provider>
  );
};

// Custom hooks for easier consumption
const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};

// Components using the context
const Header = () => {
  const { theme, toggleTheme } = useTheme();
  const { user, logout, isAuthenticated } = useAuth();

  return (
    <header style={{ 
      background: theme === 'dark' ? '#333' : '#fff',
      color: theme === 'dark' ? '#fff' : '#333',
      padding: '1rem'
    }}>
      <h1>My App</h1>
      
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>

      {isAuthenticated ? (
        <div>
          <span>Welcome, {user.name}!</span>
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <LoginForm />
      )}
    </header>
  );
};

const LoginForm = () => {
  const { login, isLoading } = useAuth();
  const [credentials, setCredentials] = useState({ email: '', password: '' });

  const handleSubmit = (e) => {
    e.preventDefault();
    login(credentials);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        placeholder="Email"
        value={credentials.email}
        onChange={(e) => setCredentials(prev => ({ 
          ...prev, 
          email: e.target.value 
        }))}
      />
      <input
        type="password"
        placeholder="Password"
        value={credentials.password}
        onChange={(e) => setCredentials(prev => ({ 
          ...prev, 
          password: e.target.value 
        }))}
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};

// App with providers
const App = () => (
  <ThemeProvider>
    <AuthProvider>
      <div>
        <Header />
        <main>
          <h2>Welcome to the app!</h2>
        </main>
      </div>
    </AuthProvider>
  </ThemeProvider>
);
```

### 2. Redux Toolkit

Ideal for large-scale apps; supports complex state, caching via RTK Query, time-travel debugging, and DevTools.

```jsx
// store/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk for API calls
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/users');
      if (!response.ok) throw new Error('Failed to fetch');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const addUser = createAsyncThunk(
  'users/addUser',
  async (userData, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });
      if (!response.ok) throw new Error('Failed to add user');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const userSlice = createSlice({
  name: 'users',
  initialState: {
    items: [],
    status: 'idle', // 'idle' | 'loading' | 'succeeded' | 'failed'
    error: null,
    filter: ''
  },
  reducers: {
    setFilter: (state, action) => {
      state.filter = action.payload;
    },
    clearError: (state) => {
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    builder
      // Fetch users
      .addCase(fetchUsers.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload;
      })
      // Add user
      .addCase(addUser.fulfilled, (state, action) => {
        state.items.push(action.payload);
      });
  }
});

export const { setFilter, clearError } = userSlice.actions;
export default userSlice.reducer;

// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';

export const store = configureStore({
  reducer: {
    users: userReducer
  },
  devTools: process.env.NODE_ENV !== 'production'
});

// Component using Redux
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { fetchUsers, addUser, setFilter, clearError } from './store/userSlice';

const UserManagement = () => {
  const dispatch = useDispatch();
  const { items: users, status, error, filter } = useSelector(state => state.users);

  useEffect(() => {
    if (status === 'idle') {
      dispatch(fetchUsers());
    }
  }, [status, dispatch]);

  const filteredUsers = users.filter(user =>
    user.name.toLowerCase().includes(filter.toLowerCase())
  );

  const handleAddUser = () => {
    const newUser = {
      name: 'New User',
      email: 'new@example.com'
    };
    dispatch(addUser(newUser));
  };

  if (status === 'loading') return <div>Loading...</div>;

  return (
    <div>
      <h2>User Management</h2>
      
      {error && (
        <div style={{ color: 'red' }}>
          Error: {error}
          <button onClick={() => dispatch(clearError())}>Clear</button>
        </div>
      )}

      <input
        type="text"
        placeholder="Filter users..."
        value={filter}
        onChange={(e) => dispatch(setFilter(e.target.value))}
      />

      <button onClick={handleAddUser}>Add User</button>

      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>{user.name} - {user.email}</li>
        ))}
      </ul>
    </div>
  );
};

// App with Redux Provider
import { Provider } from 'react-redux';
import { store } from './store';

const App = () => (
  <Provider store={store}>
    <UserManagement />
  </Provider>
);
```

### 3. Zustand

Lightweight hook-based store with minimal boilerplate.

```jsx
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

// Create store with persistence
const useStore = create(
  persist(
    (set, get) => ({
      // State
      count: 0,
      users: [],
      isLoading: false,
      
      // Actions
      increment: () => set(state => ({ count: state.count + 1 })),
      
      decrement: () => set(state => ({ count: state.count - 1 })),
      
      reset: () => set({ count: 0 }),
      
      fetchUsers: async () => {
        set({ isLoading: true });
        try {
          const response = await fetch('/api/users');
          const users = await response.json();
          set({ users, isLoading: false });
        } catch (error) {
          set({ isLoading: false });
          console.error('Failed to fetch users:', error);
        }
      },
      
      addUser: (user) => set(state => ({
        users: [...state.users, { ...user, id: Date.now() }]
      })),
      
      removeUser: (id) => set(state => ({
        users: state.users.filter(user => user.id !== id)
      })),
      
      // Computed values
      get doubleCount() {
        return get().count * 2;
      }
    }),
    {
      name: 'app-storage', // localStorage key
      partialize: (state) => ({ count: state.count }) // only persist count
    }
  )
);

// Components using Zustand
const Counter = () => {
  const { count, increment, decrement, reset, doubleCount } = useStore();

  return (
    <div>
      <h3>Counter</h3>
      <p>Count: {count}</p>
      <p>Double Count: {doubleCount}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
};

const UserList = () => {
  const { users, isLoading, fetchUsers, addUser, removeUser } = useStore();

  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  const handleAddUser = () => {
    const name = prompt('Enter user name:');
    if (name) {
      addUser({ name, email: `${name}@example.com` });
    }
  };

  if (isLoading) return <div>Loading users...</div>;

  return (
    <div>
      <h3>Users</h3>
      <button onClick={handleAddUser}>Add User</button>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name} - {user.email}
            <button onClick={() => removeUser(user.id)}>Remove</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

const ZustandApp = () => (
  <div>
    <Counter />
    <UserList />
  </div>
);
```

## 5. Higher-Order Components (HOCs)

A Higher-Order Component (HOC) is a function that takes a component and returns a new component with enhanced behavior.

### Basic HOC Example

```jsx
// Basic HOC that adds extra props
const withExtraProps = (WrappedComponent) => {
  return (props) => {
    return (
      <WrappedComponent 
        {...props} 
        newProp="extra value"
        timestamp={Date.now()}
      />
    );
  };
};

// Usage
const BasicComponent = ({ name, newProp, timestamp }) => (
  <div>
    <h3>Hello {name}</h3>
    <p>Extra prop: {newProp}</p>
    <p>Timestamp: {new Date(timestamp).toLocaleString()}</p>
  </div>
);

const EnhancedComponent = withExtraProps(BasicComponent);

// Usage: <EnhancedComponent name="John" />
```

### Advanced HOC Examples

#### 1. withLoading HOC

```jsx
const withLoading = (WrappedComponent) => {
  return ({ isLoading, loadingMessage = "Loading...", ...props }) => {
    if (isLoading) {
      return (
        <div style={{ textAlign: 'center', padding: '20px' }}>
          <div>{loadingMessage}</div>
        </div>
      );
    }
    
    return <WrappedComponent {...props} />;
  };
};

// Usage
const UserProfile = ({ user }) => (
  <div>
    <h2>{user.name}</h2>
    <p>Email: {user.email}</p>
    <p>Role: {user.role}</p>
  </div>
);

const UserProfileWithLoading = withLoading(UserProfile);

// In parent component
const App = () => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(userData => {
        setUser(userData);
        setIsLoading(false);
      });
  }, []);

  return (
    <UserProfileWithLoading
      isLoading={isLoading}
      loadingMessage="Fetching user data..."
      user={user}
    />
  );
};
```

#### 2. withAuth HOC

```jsx
const withAuth = (WrappedComponent, requiredRole = null) => {
  return (props) => {
    const { user, isAuthenticated } = useAuth(); // Assume this hook exists

    if (!isAuthenticated) {
      return (
        <div style={{ textAlign: 'center', padding: '20px' }}>
          <h2>Access Denied</h2>
          <p>Please log in to access this page.</p>
          <button onClick={() => window.location.href = '/login'}>
            Go to Login
          </button>
        </div>
      );
    }

    if (requiredRole && user.role !== requiredRole) {
      return (
        <div style={{ textAlign: 'center', padding: '20px' }}>
          <h2>Insufficient Permissions</h2>
          <p>You don't have permission to access this page.</p>
          <p>Required role: {requiredRole}</p>
          <p>Your role: {user.role}</p>
        </div>
      );
    }

    return <WrappedComponent {...props} user={user} />;
  };
};

// Usage
const AdminPanel = ({ user }) => (
  <div>
    <h2>Admin Panel</h2>
    <p>Welcome, {user.name}!</p>
    <button>Manage Users</button>
    <button>System Settings</button>
  </div>
);

const Dashboard = ({ user }) => (
  <div>
    <h2>Dashboard</h2>
    <p>Hello, {user.name}!</p>
    <p>Your last login: {user.lastLogin}</p>
  </div>
);

// Protected components
const ProtectedAdminPanel = withAuth(AdminPanel, 'admin');
const ProtectedDashboard = withAuth(Dashboard);
```

#### 3. withErrorBoundary HOC

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({
      error: error,
      errorInfo: errorInfo
    });
  }

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ 
          padding: '20px', 
          border: '1px solid red', 
          borderRadius: '4px',
          backgroundColor: '#fee' 
        }}>
          <h2>Something went wrong!</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            <summary>Error Details</summary>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo.componentStack}
          </details>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

const withErrorBoundary = (WrappedComponent) => {
  return (props) => (
    <ErrorBoundary>
      <WrappedComponent {...props} />
    </ErrorBoundary>
  );
};

// Component that might throw an error
const ProblematicComponent = ({ shouldThrow }) => {
  if (shouldThrow) {
    throw new Error('This is a test error!');
  }
  
  return <div>Component working fine!</div>;
};

const SafeComponent = withErrorBoundary(ProblematicComponent);
```

#### 4. withLocalStorage HOC

```jsx
const withLocalStorage = (WrappedComponent, storageKey, defaultValue = null) => {
  return (props) => {
    const [storedValue, setStoredValue] = useState(() => {
      try {
        const item = window.localStorage.getItem(storageKey);
        return item ? JSON.parse(item) : defaultValue;
      } catch (error) {
        console.error(`Error reading from localStorage:`, error);
        return defaultValue;
      }
    });

    const setValue = useCallback((value) => {
      try {
        setStoredValue(value);
        window.localStorage.setItem(storageKey, JSON.stringify(value));
      } catch (error) {
        console.error(`Error saving to localStorage:`, error);
      }
    }, [storageKey]);

    return (
      <WrappedComponent
        {...props}
        storedValue={storedValue}
        setStoredValue={setValue}
      />
    );
  };
};

// Usage
const UserPreferences = ({ storedValue: preferences, setStoredValue: setPreferences }) => {
  const handleThemeChange = (theme) => {
    setPreferences({ ...preferences, theme });
  };

  const handleLanguageChange = (language) => {
    setPreferences({ ...preferences, language });
  };

  return (
    <div>
      <h3>User Preferences</h3>
      
      <div>
        <label>Theme: </label>
        <select 
          value={preferences?.theme || 'light'}
          onChange={(e) => handleThemeChange(e.target.value)}
        >
          <option value="light">Light</option>
          <option value="dark">Dark</option>
        </select>
      </div>

      <div>
        <label>Language: </label>
        <select
          value={preferences?.language || 'en'}
          onChange={(e) => handleLanguageChange(e.target.value)}
        >
          <option value="en">English</option>
          <option value="es">Spanish</option>
          <option value="fr">French</option>
        </select>
      </div>

      <div>
        <h4>Current Preferences:</h4>
        <pre>{JSON.stringify(preferences, null, 2)}</pre>
      </div>
    </div>
  );
};

const UserPreferencesWithStorage = withLocalStorage(
  UserPreferences, 
  'userPreferences', 
  { theme: 'light', language: 'en' }
);
```

#### 5. Composing Multiple HOCs

```jsx
// Compose multiple HOCs together
const compose = (...hocs) => (Component) => 
  hocs.reduceRight((acc, hoc) => hoc(acc), Component);

// Or using a utility function
const enhance = compose(
  withAuth,
  withLoading,
  withErrorBoundary,
  withLocalStorage('dashboardSettings', { layout: 'grid' })
);

const Dashboard = ({ user, isLoading, storedValue, setStoredValue }) => (
  <div>
    <h2>Enhanced Dashboard</h2>
    <p>User: {user.name}</p>
    <p>Layout: {storedValue.layout}</p>
    <button onClick={() => setStoredValue({ layout: 'list' })}>
      Switch to List View
    </button>
  </div>
);

const EnhancedDashboard = enhance(Dashboard);

// Usage with all enhancements
const App = () => {
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setTimeout(() => setIsLoading(false), 2000);
  }, []);

  return (
    <EnhancedDashboard 
      isLoading={isLoading}
      loadingMessage="Loading dashboard..."
    />
  );
};
```

### HOCs vs Hooks Comparison

```jsx
// HOC approach
const withCounter = (WrappedComponent) => {
  return (props) => {
    const [count, setCount] = useState(0);
    
    return (
      <WrappedComponent
        {...props}
        count={count}
        increment={() => setCount(count + 1)}
        decrement={() => setCount(count - 1)}
      />
    );
  };
};

// Hook approach (Modern preference)
const useCounter = (initialValue = 0) => {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => setCount(prev => prev + 1), []);
  const decrement = useCallback(() => setCount(prev => prev - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);
  
  return { count, increment, decrement, reset };
};

// Component using HOC
const CounterComponentHOC = ({ count, increment, decrement }) => (
  <div>
    <p>Count: {count}</p>
    <button onClick={increment}>+</button>
    <button onClick={decrement}>-</button>
  </div>
);

const EnhancedCounterHOC = withCounter(CounterComponentHOC);

// Component using Hook (preferred)
const CounterComponentHook = () => {
  const { count, increment, decrement, reset } = useCounter(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
};
```

## Summary

This guide covers the essential React concepts for building optimized, maintainable applications:

1. **State vs Props**: Understanding data flow and component communication
2. **Component Lifecycle**: Managing side effects and cleanup with hooks
3. **Optimization Techniques**: Using React.memo, useMemo, useCallback, and code splitting
4. **State Management**: Context API for simple cases, Redux Toolkit for complex apps, Zustand for lightweight solutions
5. **Higher-Order Components**: Legacy pattern for cross-cutting concerns (mostly replaced by hooks)

**Key Takeaways:**

- Always clean up side effects to prevent memory leaks
- Use optimization techniques judiciously - measure before optimizing
- Choose the right state management solution based on your app's complexity
- Prefer hooks over HOCs for reusable logic
- Keep components focused and break them down when they get too large

## 6. What is JSX and how does it work?

**Simple Answer:** JSX is like writing HTML inside JavaScript. It makes creating UI components easier and more readable.

**Example:**

```jsx
// Instead of writing:
React.createElement('h1', null, 'Hello World')

// You write:
<h1>Hello World</h1>
```

**How it works:**

- You write HTML-like code
- Babel (a tool) converts it to regular JavaScript
- React uses that JavaScript to create elements

**Why use it?**

- More readable and intuitive
- Looks like the UI you're building
- Catches errors at compile time

---

## 7. What is Reconciliation in React?

**Simple Answer:** Reconciliation is React's way of figuring out what changed in your UI and updating only those parts, instead of redrawing everything.

**Real-world analogy:** Imagine you have a todo list. Instead of rewriting the entire list when you mark one item complete, you just cross out that one item. That's what React does!

**How it works:**

1. You update state/props
2. React creates a new Virtual DOM
3. React compares new Virtual DOM with old Virtual DOM (this is reconciliation)
4. React updates only the changed parts in the real DOM

**Why it matters:**

- Makes your app fast
- Saves computer resources
- Users see smooth updates

---

## 8. What is useRef and its use cases?

**Simple Answer:** `useRef` is like a box where you can store a value that:

- Doesn't cause re-renders when changed
- Persists between renders
- Can directly access DOM elements

**Use Cases:**

### Use Case 1: Accessing DOM Elements

```jsx
function FocusInput() {
  const inputRef = useRef(null);
  
  const handleClick = () => {
    inputRef.current.focus(); // Focus the input
  };
  
  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus Input</button>
    </>
  );
}
```

### Use Case 9: Storing Previous Values

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();
  
  useEffect(() => {
    prevCountRef.current = count; // Store previous count
  });
  
  return <div>Now: {count}, Before: {prevCountRef.current}</div>;
}
```

### Use Case 10: Storing Mutable Data (without re-renders)

```jsx
function Timer() {
  const intervalRef = useRef(null);
  
  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      console.log('Tick');
    }, 1000);
  };
  
  const stopTimer = () => {
    clearInterval(intervalRef.current);
  };
  
  return (
    <>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </>
  );
}
```

**When to use:**

- Accessing DOM elements (input focus, scroll position)
- Storing timers or intervals
- Keeping track of previous values
- Any value that shouldn't trigger re-renders

---

## 11. What is a Custom Hook?

**Simple Answer:** A custom hook is your own reusable function that uses React hooks inside it. It's like creating a recipe that you can use in multiple places.

**When to create one:**

- You're copying the same logic in multiple components
- You want to organize complex logic
- You want to share functionality across your app

**Example: useFetch Custom Hook**

```jsx
// Custom Hook
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [url]);
  
  return { data, loading, error };
}

// Using the custom hook
function UserProfile() {
  const { data, loading, error } = useFetch('/api/user');
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error!</div>;
  return <div>{data.name}</div>;
}
```

**Rules for Custom Hooks:**

- Must start with "use" (e.g., useFetch, useAuth)
- Can use other hooks inside
- Can return anything (values, functions, objects)

---

## 12. Controlled vs Uncontrolled Components

**Simple Answer:** Who manages the form data - React or the browser?
### Controlled Components (React is in charge)
**How it works:**
- React state holds the value
- onChange updates the state
- Input always shows what's in state

```jsx
function ControlledInput() {
  const [name, setName] = useState('');
  
  return (
    <input 
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}
```

**Pros:**

- Full control over input
- Easy to validate in real-time
- Can format/transform input immediately
- Easy to disable/enable submit button

**Cons:**

- More code to write
- Re-renders on every keystroke

### Uncontrolled Components (Browser is in charge)

**How it works:**

- Browser manages the value
- Use `ref` to get value when needed
- React doesn't know the value until you ask

```jsx
function UncontrolledInput() {
  const inputRef = useRef();
  
  const handleSubmit = () => {
    console.log(inputRef.current.value); // Get value only when needed
  };
  
  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleSubmit}>Submit</button>
    </>
  );
}
```

**Pros:**

- Less code
- Better performance (no re-renders)
- Works with non-React code

**Cons:**

- Less control
- Harder to validate in real-time
- Need refs to access values

### When to use which?

**Use Controlled when:**

- You need validation as user types
- You need to format input (e.g., phone numbers)
- You want to disable submit until valid
- Multiple inputs depend on each other

**Use Uncontrolled when:**

- Simple forms (login, search)
- File inputs (must be uncontrolled)
- Integrating with non-React libraries
- Performance is critical

---

## 13. Error Boundaries in React

**Simple Answer:** Error boundaries are like a safety net that catches errors in your React components and shows a nice error message instead of crashing your entire app.

**Real-world analogy:** Think of it like an airbag in a car. If something goes wrong, the airbag (error boundary) activates to protect you, instead of letting the whole car crash.

**How to create one:**

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error) {
    // Update state so next render shows fallback UI
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    // Log error to error reporting service
    console.log('Error:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong. Please try again.</h1>;
    }
    
    return this.props.children;
  }
}

// How to use it
function App() {
  return (
    <ErrorBoundary>
      <MyComponent />
    </ErrorBoundary>
  );
}
```

**What errors do they catch?**

- ✅ Errors in rendering
- ✅ Errors in lifecycle methods
- ✅ Errors in constructors

**What errors do they NOT catch?**

- ❌ Event handlers (use try-catch)
- ❌ Async code (setTimeout, promises)
- ❌ Server-side rendering
- ❌ Errors in the error boundary itself

---

## 14. Virtual DOM and How React Uses It

**Simple Answer:** The Virtual DOM is like a blueprint of your UI that React keeps in memory. React uses it to figure out the fastest way to update your actual webpage.

**Real-world analogy:** Imagine you're redecorating your room:

- **Without Virtual DOM:** Move everything around, then realize what doesn't work, move it back, repeat.
- **With Virtual DOM:** Plan on paper first, compare with current layout, then only move what needs to change.

**How it works:**

1. **Initial Render:**
    
    - React creates a Virtual DOM (JavaScript object tree)
    - React creates the real DOM from it
    - User sees the page
2. **When state changes:**
    
    - React creates a NEW Virtual DOM
    - Compares new Virtual DOM with old Virtual DOM (diffing)
    - Calculates minimum changes needed
    - Updates only those parts in real DOM

**Example:**

```jsx
// You have this:
<div>
  <h1>Hello</h1>
  <p>Count: 0</p>
</div>

// State changes, now you need:
<div>
  <h1>Hello</h1>
  <p>Count: 1</p>
</div>

// React only updates the text "0" → "1"
// Doesn't recreate the entire div, h1, or p elements
```

**Why it's fast:**

- Updating real DOM is slow
- JavaScript operations (Virtual DOM) are fast
- React batches multiple changes together
- Only updates what actually changed

---

## 15. Implementing Dark/Light Mode with Context

**Simple Answer:** Create a "theme manager" that any component can access without passing props through every level.

**Step-by-step Implementation:**

### Step 1: Create Theme Context

```jsx
import { createContext, useState, useContext } from 'react';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook for easy access
export function useTheme() {
  return useContext(ThemeContext);
}
```

### Step 2: Wrap Your App

```jsx
function App() {
  return (
    <ThemeProvider>
      <Header />
      <MainContent />
      <Footer />
    </ThemeProvider>
  );
}
```

### Step 3: Use Theme in Components

```jsx
function Header() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <header className={theme}>
      <h1>My App</h1>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}

function MainContent() {
  const { theme } = useTheme();
  
  return (
    <main className={theme}>
      <p>Content here...</p>
    </main>
  );
}
```

### Step 4: Add CSS

```css
.light {
  background-color: white;
  color: black;
}

.dark {
  background-color: #1a1a1a;
  color: white;
}
```

**Why use Context?**

- No prop drilling (passing theme through every component)
- Any component can access theme
- Single source of truth
- Easy to maintain

**Advanced: Persist theme in localStorage**

```jsx
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState(() => {
    const saved = localStorage.getItem('theme');
    return saved || 'light';
  });
  
  const toggleTheme = () => {
    setTheme(prevTheme => {
      const newTheme = prevTheme === 'light' ? 'dark' : 'light';
      localStorage.setItem('theme', newTheme);
      return newTheme;
    });
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

---

## Quick Reference Table

|Concept|When to Use|Key Benefit|
|---|---|---|
|**JSX**|Always in React|Readable UI code|
|**useRef**|DOM access, persist values|No re-renders|
|**Custom Hooks**|Reusable logic|Code organization|
|**Controlled Components**|Complex forms|Full control|
|**Uncontrolled Components**|Simple forms|Better performance|
|**Error Boundaries**|Catch component errors|Prevent crashes|
|**Virtual DOM**|Automatic|Fast updates|
|**Context**|Global state|Avoid prop drilling|

## Tips for Interviews

1. **Explain with examples:** Always back up your answers with code examples
2. **Mention trade-offs:** Show you understand pros and cons
3. **Think about performance:** Discuss when something is fast or slow
4. **Real-world scenarios:** Connect concepts to actual use cases
5. **Ask clarifying questions:** If something is unclear, ask!

Good luck with your interview! 🚀