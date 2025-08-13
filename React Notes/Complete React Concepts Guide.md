
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