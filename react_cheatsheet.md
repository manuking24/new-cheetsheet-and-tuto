# Complete React.js Cheatsheet

A comprehensive guide to modern React development using functional components and hooks.

## Table of Contents
1. [Setting up React](#setting-up-react)
2. [JSX Syntax](#jsx-syntax)
3. [Components](#components)
4. [Props and PropTypes](#props-and-proptypes)
5. [State with useState](#state-with-usestate)
6. [Side Effects with useEffect](#side-effects-with-useeffect)
7. [Event Handling](#event-handling)
8. [Conditional Rendering](#conditional-rendering)
9. [Lists and Keys](#lists-and-keys)
10. [Forms and Controlled Components](#forms-and-controlled-components)
11. [useRef, useMemo, and useCallback](#useref-usememo-and-usecallback)
12. [Context API](#context-api)
13. [Custom Hooks](#custom-hooks)
14. [Routing with React Router](#routing-with-react-router)
15. [Styling](#styling)
16. [API Data Fetching](#api-data-fetching)
17. [Error Handling](#error-handling)
18. [Best Practices](#best-practices)

---

## Setting up React

React apps can be created using Vite (recommended) or Create React App.

### With Vite (Recommended)
```bash
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev
```

### With Create React App
```bash
npx create-react-app my-react-app
cd my-react-app
npm start
```

**Explanation:** Vite is faster and more modern than Create React App. It provides hot module replacement, faster builds, and better development experience. The `--template react` flag creates a React project with JavaScript (use `--template react-ts` for TypeScript).

---

## JSX Syntax

JSX allows you to write HTML-like syntax in JavaScript.

```jsx
import React from 'react';

function App() {
  const name = 'John';
  const isLoggedIn = true;
  
  return (
    <div className="app">
      <h1>Hello, {name}!</h1>
      <p>Today is {new Date().toLocaleDateString()}</p>
      {isLoggedIn && <button>Logout</button>}
      <img src="/logo.png" alt="Logo" />
    </div>
  );
}

export default App;
```

**Explanation:** JSX is transpiled to `React.createElement()` calls. Key points:
- Use `className` instead of `class`
- JavaScript expressions go inside `{}`
- Components must return a single parent element or React Fragment
- Self-closing tags must end with `/>`
- camelCase for HTML attributes (onClick, onChange)

---

## Components

Functional components are the modern way to create React components.

```jsx
// Basic functional component
function Welcome() {
  return <h1>Welcome to React!</h1>;
}

// Arrow function component
const Greeting = () => {
  return <p>Hello from arrow function!</p>;
};

// Component with JSX fragment
function UserCard() {
  return (
    <>
      <h2>John Doe</h2>
      <p>Software Developer</p>
    </>
  );
}

// Using the components
function App() {
  return (
    <div>
      <Welcome />
      <Greeting />
      <UserCard />
    </div>
  );
}
```

**Explanation:** Components are reusable pieces of UI. They must start with a capital letter. React Fragments (`<>...</>`) allow returning multiple elements without adding extra DOM nodes.

---

## Props and PropTypes

Props pass data from parent to child components.

```jsx
import PropTypes from 'prop-types';

// Component with props
function UserProfile({ name, age, isActive = false, onEdit }) {
  return (
    <div className={`user-profile ${isActive ? 'active' : ''}`}>
      <h3>{name}</h3>
      <p>Age: {age}</p>
      <button onClick={() => onEdit(name)}>Edit Profile</button>
    </div>
  );
}

// PropTypes for type checking
UserProfile.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired,
  isActive: PropTypes.bool,
  onEdit: PropTypes.func.isRequired
};

// Using the component
function App() {
  const handleEdit = (userName) => {
    console.log(`Editing ${userName}'s profile`);
  };

  return (
    <UserProfile 
      name="Alice" 
      age={25} 
      isActive={true}
      onEdit={handleEdit}
    />
  );
}
```

**Explanation:** Props are read-only and flow down from parent to child. Default values can be set using default parameters. PropTypes provide runtime type checking in development mode.

---

## State with useState

useState hook manages component state in functional components.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const [user, setUser] = useState({ name: '', email: '' });

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(prev => prev - 1);

  const updateUser = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      
      <div>
        <input 
          placeholder="Name"
          onChange={(e) => updateUser('name', e.target.value)}
        />
        <input 
          placeholder="Email"
          onChange={(e) => updateUser('email', e.target.value)}
        />
        <p>User: {user.name} ({user.email})</p>
      </div>
    </div>
  );
}
```

**Explanation:** useState returns an array with current state and setter function. Always use the setter function to update state. For objects/arrays, create new instances to ensure re-renders. Use functional updates when new state depends on previous state.

---

## Side Effects with useEffect

useEffect handles side effects like API calls, subscriptions, and DOM manipulation.

```jsx
import { useState, useEffect } from 'react';

function UserData() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [count, setCount] = useState(0);

  // Effect with no dependencies - runs after every render
  useEffect(() => {
    document.title = `Count: ${count}`;
  });

  // Effect with empty dependency array - runs once on mount
  useEffect(() => {
    fetchUser();
    
    // Cleanup function - runs on unmount
    return () => {
      console.log('Component unmounting');
    };
  }, []);

  // Effect with dependencies - runs when dependencies change
  useEffect(() => {
    if (count > 0) {
      console.log(`Count changed to: ${count}`);
    }
  }, [count]);

  const fetchUser = async () => {
    try {
      const response = await fetch('/api/user');
      const userData = await response.json();
      setUser(userData);
    } catch (error) {
      console.error('Error fetching user:', error);
    } finally {
      setLoading(false);
    }
  };

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h2>{user?.name}</h2>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Explanation:** useEffect runs side effects after render. Dependencies array controls when effect runs:
- No array: runs after every render
- Empty array `[]`: runs once on mount
- With values `[count]`: runs when those values change
- Return cleanup function for subscriptions/timers

---

## Event Handling

Handle user interactions with event handlers.

```jsx
import { useState } from 'react';

function EventHandling() {
  const [message, setMessage] = useState('');
  const [position, setPosition] = useState({ x: 0, y: 0 });

  // Basic event handler
  const handleClick = () => {
    alert('Button clicked!');
  };

  // Handler with event object
  const handleInputChange = (event) => {
    setMessage(event.target.value);
  };

  // Handler with parameters
  const handleButtonClick = (buttonName) => {
    console.log(`${buttonName} button clicked`);
  };

  // Mouse event handler
  const handleMouseMove = (event) => {
    setPosition({ x: event.clientX, y: event.clientY });
  };

  // Form submission handler
  const handleSubmit = (event) => {
    event.preventDefault();
    console.log('Form submitted with message:', message);
  };

  return (
    <div onMouseMove={handleMouseMove}>
      <form onSubmit={handleSubmit}>
        <input 
          type="text" 
          value={message}
          onChange={handleInputChange}
          placeholder="Enter message"
        />
        <button type="submit">Submit</button>
      </form>
      
      <button onClick={handleClick}>Simple Click</button>
      <button onClick={() => handleButtonClick('Save')}>Save</button>
      <button onClick={() => handleButtonClick('Cancel')}>Cancel</button>
      
      <p>Mouse position: {position.x}, {position.y}</p>
      <p>Current message: {message}</p>
    </div>
  );
}
```

**Explanation:** Event handlers receive SyntheticEvent objects that wrap native events. Use arrow functions to pass parameters. Always call `preventDefault()` to prevent default browser behavior when needed.

---

## Conditional Rendering

Render different content based on conditions.

```jsx
import { useState } from 'react';

function ConditionalRendering() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [userRole, setUserRole] = useState('guest');
  const [notifications, setNotifications] = useState([]);

  // Conditional rendering with &&
  const renderNotifications = () => (
    <>
      {notifications.length > 0 && (
        <div className="notifications">
          <h3>Notifications ({notifications.length})</h3>
          {notifications.map((notif, index) => (
            <p key={index}>{notif}</p>
          ))}
        </div>
      )}
    </>
  );

  // Conditional rendering with ternary operator
  const renderUserStatus = () => (
    <div>
      {isLoggedIn ? (
        <div>
          <h2>Welcome back!</h2>
          <button onClick={() => setIsLoggedIn(false)}>Logout</button>
        </div>
      ) : (
        <div>
          <h2>Please log in</h2>
          <button onClick={() => setIsLoggedIn(true)}>Login</button>
        </div>
      )}
    </div>
  );

  // Multiple conditions with switch-like pattern
  const renderByRole = () => {
    if (userRole === 'admin') {
      return <AdminPanel />;
    } else if (userRole === 'user') {
      return <UserDashboard />;
    } else {
      return <GuestView />;
    }
  };

  return (
    <div>
      {renderUserStatus()}
      {renderNotifications()}
      
      <div>
        <label>User Role: </label>
        <select value={userRole} onChange={(e) => setUserRole(e.target.value)}>
          <option value="guest">Guest</option>
          <option value="user">User</option>
          <option value="admin">Admin</option>
        </select>
      </div>
      
      {renderByRole()}
    </div>
  );
}

// Example components for different roles
const AdminPanel = () => <div>Admin Panel - Full Access</div>;
const UserDashboard = () => <div>User Dashboard - Limited Access</div>;
const GuestView = () => <div>Guest View - Public Content Only</div>;
```

**Explanation:** Use `&&` for simple conditional rendering, ternary operators for either/or scenarios, and if/else statements for complex conditions. Be careful with falsy values in `&&` expressions.

---

## Lists and Keys

Render arrays of data efficiently with proper keys.

```jsx
import { useState } from 'react';

function ListsAndKeys() {
  const [users, setUsers] = useState([
    { id: 1, name: 'Alice', age: 25, active: true },
    { id: 2, name: 'Bob', age: 30, active: false },
    { id: 3, name: 'Charlie', age: 35, active: true }
  ]);

  const [filter, setFilter] = useState('all');

  // Filter users based on status
  const filteredUsers = users.filter(user => {
    if (filter === 'active') return user.active;
    if (filter === 'inactive') return !user.active;
    return true;
  });

  // Toggle user active status
  const toggleUserStatus = (userId) => {
    setUsers(users.map(user => 
      user.id === userId 
        ? { ...user, active: !user.active }
        : user
    ));
  };

  // Remove user
  const removeUser = (userId) => {
    setUsers(users.filter(user => user.id !== userId));
  };

  // Add new user
  const addUser = () => {
    const newUser = {
      id: Date.now(),
      name: `User ${users.length + 1}`,
      age: Math.floor(Math.random() * 50) + 18,
      active: true
    };
    setUsers([...users, newUser]);
  };

  return (
    <div>
      <div>
        <button onClick={addUser}>Add User</button>
        <select value={filter} onChange={(e) => setFilter(e.target.value)}>
          <option value="all">All Users</option>
          <option value="active">Active Users</option>
          <option value="inactive">Inactive Users</option>
        </select>
      </div>

      <ul>
        {filteredUsers.map(user => (
          <li key={user.id} className={user.active ? 'active' : 'inactive'}>
            <span>{user.name} (Age: {user.age})</span>
            <button onClick={() => toggleUserStatus(user.id)}>
              {user.active ? 'Deactivate' : 'Activate'}
            </button>
            <button onClick={() => removeUser(user.id)}>Remove</button>
          </li>
        ))}
      </ul>

      {filteredUsers.length === 0 && (
        <p>No users match the current filter.</p>
      )}
    </div>
  );
}
```

**Explanation:** Always provide unique `key` props when rendering lists. Keys help React identify which items have changed. Use stable, unique identifiers (like IDs) rather than array indices when possible. Keys should be unique among siblings, not globally.

---

## Forms and Controlled Components

Handle form input with controlled components where React controls the input value.

```jsx
import { useState } from 'react';

function FormHandling() {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    age: '',
    country: '',
    interests: [],
    newsletter: false,
    comments: ''
  });

  const [errors, setErrors] = useState({});

  // Handle input changes
  const handleInputChange = (event) => {
    const { name, value, type, checked } = event.target;
    
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));

    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  // Handle checkbox array (interests)
  const handleInterestChange = (interest) => {
    setFormData(prev => ({
      ...prev,
      interests: prev.interests.includes(interest)
        ? prev.interests.filter(i => i !== interest)
        : [...prev.interests, interest]
    }));
  };

  // Form validation
  const validateForm = () => {
    const newErrors = {};

    if (!formData.firstName.trim()) {
      newErrors.firstName = 'First name is required';
    }

    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }

    if (!formData.age || formData.age < 18) {
      newErrors.age = 'Must be 18 or older';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  // Handle form submission
  const handleSubmit = (event) => {
    event.preventDefault();
    
    if (validateForm()) {
      console.log('Form submitted:', formData);
      // Reset form
      setFormData({
        firstName: '',
        lastName: '',
        email: '',
        age: '',
        country: '',
        interests: [],
        newsletter: false,
        comments: ''
      });
    }
  };

  const interestOptions = ['JavaScript', 'React', 'Node.js', 'Python', 'Design'];

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>First Name:</label>
        <input
          type="text"
          name="firstName"
          value={formData.firstName}
          onChange={handleInputChange}
        />
        {errors.firstName && <span className="error">{errors.firstName}</span>}
      </div>

      <div>
        <label>Last Name:</label>
        <input
          type="text"
          name="lastName"
          value={formData.lastName}
          onChange={handleInputChange}
        />
      </div>

      <div>
        <label>Email:</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleInputChange}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>

      <div>
        <label>Age:</label>
        <input
          type="number"
          name="age"
          value={formData.age}
          onChange={handleInputChange}
        />
        {errors.age && <span className="error">{errors.age}</span>}
      </div>

      <div>
        <label>Country:</label>
        <select name="country" value={formData.country} onChange={handleInputChange}>
          <option value="">Select a country</option>
          <option value="us">United States</option>
          <option value="ca">Canada</option>
          <option value="uk">United Kingdom</option>
        </select>
      </div>

      <div>
        <label>Interests:</label>
        {interestOptions.map(interest => (
          <label key={interest}>
            <input
              type="checkbox"
              checked={formData.interests.includes(interest)}
              onChange={() => handleInterestChange(interest)}
            />
            {interest}
          </label>
        ))}
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            name="newsletter"
            checked={formData.newsletter}
            onChange={handleInputChange}
          />
          Subscribe to newsletter
        </label>
      </div>

      <div>
        <label>Comments:</label>
        <textarea
          name="comments"
          value={formData.comments}
          onChange={handleInputChange}
          rows={4}
        />
      </div>

      <button type="submit">Submit</button>
    </form>
  );
}
```

**Explanation:** Controlled components have their value controlled by React state. This provides full control over form data and enables real-time validation. Always use `preventDefault()` in form submission handlers. Manage form state with a single object for complex forms.

---

## useRef, useMemo, and useCallback

Performance optimization and DOM access hooks.

```jsx
import { useState, useRef, useMemo, useCallback, useEffect } from 'react';

function OptimizationHooks() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState(['apple', 'banana', 'cherry']);
  const [filter, setFilter] = useState('');

  // useRef - Direct DOM access and persistent values
  const inputRef = useRef(null);
  const renderCount = useRef(0);

  useEffect(() => {
    renderCount.current += 1;
  });

  const focusInput = () => {
    inputRef.current.focus();
  };

  // useMemo - Expensive calculations
  const expensiveValue = useMemo(() => {
    console.log('Calculating expensive value...');
    return count * 1000 + Math.random();
  }, [count]); // Only recalculate when count changes

  // useMemo - Filtered list
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item => 
      item.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);

  // useCallback - Memoized functions
  const handleAddItem = useCallback(() => {
    const newItem = `item-${Date.now()}`;
    setItems(prev => [...prev, newItem]);
  }, []); // Function never changes

  const handleRemoveItem = useCallback((index) => {
    setItems(prev => prev.filter((_, i) => i !== index));
  }, []); // Function never changes

  // useCallback with dependencies
  const handleFilteredAction = useCallback((action) => {
    console.log(`Performing ${action} on ${filteredItems.length} items`);
  }, [filteredItems.length]); // Recreate when filteredItems.length changes

  return (
    <div>
      <h2>Optimization Hooks Demo</h2>
      <p>Render count: {renderCount.current}</p>
      
      <div>
        <input 
          ref={inputRef}
          type="text" 
          placeholder="Focus me with button"
        />
        <button onClick={focusInput}>Focus Input</button>
      </div>

      <div>
        <button onClick={() => setCount(count + 1)}>
          Count: {count}
        </button>
        <p>Expensive value: {expensiveValue}</p>
      </div>

      <div>
        <input
          type="text"
          value={filter}
          onChange={(e) => setFilter(e.target.value)}
          placeholder="Filter items"
        />
        <button onClick={handleAddItem}>Add Item</button>
      </div>

      <ul>
        {filteredItems.map((item, index) => (
          <li key={item}>
            {item}
            <button onClick={() => handleRemoveItem(index)}>Remove</button>
          </li>
        ))}
      </ul>

      <button onClick={() => handleFilteredAction('process')}>
        Process Filtered Items
      </button>

      <ChildComponent 
        onAction={handleFilteredAction}
        items={filteredItems}
      />
    </div>
  );
}

// Child component that receives memoized props
const ChildComponent = React.memo(({ onAction, items }) => {
  console.log('ChildComponent rendered');
  
  return (
    <div>
      <h3>Child Component</h3>
      <p>Received {items.length} items</p>
      <button onClick={() => onAction('child-action')}>
        Child Action
      </button>
    </div>
  );
});
```

**Explanation:**
- **useRef**: Access DOM elements directly or store mutable values that persist across renders without causing re-renders
- **useMemo**: Memoize expensive calculations, only recalculate when dependencies change
- **useCallback**: Memoize functions to prevent unnecessary re-renders of child components
- Use these hooks judiciously; premature optimization can hurt performance

---

## Context API

Share state across components without prop drilling.

```jsx
import { createContext, useContext, useState, useReducer } from 'react';

// Create contexts
const ThemeContext = createContext();
const UserContext = createContext();

// Theme context with simple state
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  const value = {
    theme,
    toggleTheme
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// User context with reducer for complex state
const userReducer = (state, action) => {
  switch (action.type) {
    case 'LOGIN':
      return {
        ...state,
        isAuthenticated: true,
        user: action.payload
      };
    case 'LOGOUT':
      return {
        ...state,
        isAuthenticated: false,
        user: null
      };
    case 'UPDATE_PROFILE':
      return {
        ...state,
        user: { ...state.user, ...action.payload }
      };
    default:
      return state;
  }
};

function UserProvider({ children }) {
  const [state, dispatch] = useReducer(userReducer, {
    isAuthenticated: false,
    user: null
  });

  const login = (userData) => {
    dispatch({ type: 'LOGIN', payload: userData });
  };

  const logout = () => {
    dispatch({ type: 'LOGOUT' });
  };

  const updateProfile = (updates) => {
    dispatch({ type: 'UPDATE_PROFILE', payload: updates });
  };

  const value = {
    ...state,
    login,
    logout,
    updateProfile
  };

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// Custom hooks for using contexts
const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

const useUser = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
};

// Components using the contexts
function Header() {
  const { theme, toggleTheme } = useTheme();
  const { isAuthenticated, user, logout } = useUser();

  return (
    <header className={`header ${theme}`}>
      <h1>My App</h1>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} theme
      </button>
      
      {isAuthenticated ? (
        <div>
          <span>Welcome, {user.name}!</span>
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <LoginButton />
      )}
    </header>
  );
}

function LoginButton() {
  const { login } = useUser();

  const handleLogin = () => {
    login({ id: 1, name: 'John Doe', email: 'john@example.com' });
  };

  return <button onClick={handleLogin}>Login</button>;
}

function Profile() {
  const { theme } = useTheme();
  const { isAuthenticated, user, updateProfile } = useUser();

  const handleUpdateProfile = () => {
    updateProfile({ name: 'Jane Doe' });
  };

  if (!isAuthenticated) {
    return <div>Please log in to view profile</div>;
  }

  return (
    <div className={`profile ${theme}`}>
      <h2>Profile</h2>
      <p>Name: {user.name}</p>
      <p>Email: {user.email}</p>
      <button onClick={handleUpdateProfile}>Update Name</button>
    </div>
  );
}

// Main App component
function App() {
  return (
    <ThemeProvider>
      <UserProvider>
        <div className="app">
          <Header />
          <Profile />
        </div>
      </UserProvider>
    </ThemeProvider>
  );
}
```

**Explanation:** Context API prevents prop drilling by allowing components to access shared state directly. Create context with `createContext()`, provide values with `Provider`, and consume with `useContext()`. Always create custom hooks for contexts to provide better error handling and developer experience.

---

## Custom Hooks

Create reusable stateful logic with custom hooks.

```jsx
import { useState, useEffect, useCallback, useRef } from 'react';

// Custom hook for API calls
function useApi(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(url, options);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [url, JSON.stringify(options)]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
}

// Custom hook for local storage
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue];
}

// Custom hook for debounced value
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Custom hook for previous value
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  });
  
  return ref.current;
}

// Custom hook for toggle functionality
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return [value, { toggle, setTrue, setFalse }];
}

// Custom hook for window size
function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined,
  });

  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener('resize', handleResize);
    handleResize(); // Call once to set initial size

    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return windowSize;
}

// Component using custom hooks
function CustomHooksDemo() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  const previousSearchTerm = usePrevious(debouncedSearchTerm);
  
  const [darkMode, setDarkMode] = useLocalStorage('darkMode', false);
  const [isVisible, visibilityControls] = useToggle(false);
  const windowSize = useWindowSize();

  // Use API hook with debounced search term
  const { data: searchResults, loading, error } = useApi(
    debouncedSearchTerm 
      ? `https://api.example.com/search?q=${debouncedSearchTerm}`
      : null
  );

  return (
    <div className={darkMode ? 'dark' : 'light'}>
      <h2>Custom Hooks Demo</h2>
      
      <div>
        <input
          type="text"
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          placeholder="Search..."
        />
        <p>Current search: {debouncedSearchTerm}</p>
        <p>Previous search: {previousSearchTerm}</p>
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            checked={darkMode}
            onChange={(e) => setDarkMode(e.target.checked)}
          />
          Dark Mode
        </label>
      </div>

      <div>
        <button onClick={visibilityControls.toggle}>
          {isVisible ? 'Hide' : 'Show'} Content
        </button>
        {isVisible && <p>This content is toggleable!</p>}
      </div>

      <div>
        <p>Window size: {windowSize.width} x {windowSize.height}</p>
      </div>

      <div>
        {loading && <p>Searching...</p>}
        {error && <p>Error: {error}</p>}
        {searchResults && (
          <div>
            <h3>Search Results:</h3>
            <pre>{JSON.stringify(searchResults, null, 2)}</pre>
          </div>
        )}
      </div>
    </div>
  );
}
```

**Explanation:** Custom hooks extract component logic into reusable functions. They must start with "use" and can call other hooks. They allow sharing stateful logic between components without changing component hierarchy. Common patterns include API calls, local storage, debouncing, and DOM event handling.

---

## Routing with React Router

Navigate between different views with React Router.

```jsx
import { 
  BrowserRouter as Router, 
  Routes, 
  Route, 
  Link, 
  NavLink,
  useNavigate, 
  useParams, 
  useLocation,
  Navigate,
  Outlet
} from 'react-router-dom';
import { useState } from 'react';

// Main App with Router
function App() {
  return (
    <Router>
      <div className="app">
        <Navigation />
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
          <Route path="/users" element={<UsersLayout />}>
            <Route index element={<UsersList />} />
            <Route path=":userId" element={<UserProfile />} />
            <Route path="create" element={<CreateUser />} />
          </Route>
          <Route path="/dashboard" element={<ProtectedRoute><Dashboard /></ProtectedRoute>} />
          <Route path="/login" element={<Login />} />
          <Route path="/old-page" element={<Navigate to="/new-page" replace />} />
          <Route path="*" element={<NotFound />} />
        </Routes>
      </div>
    </Router>
  );
}

// Navigation component
function Navigation() {
  return (
    <nav>
      <ul>
        <li><Link to="/">Home</Link></li>
        <li><Link to="/about">About</Link></li>
        <li>
          <NavLink 
            to="/contact" 
            className={({ isActive }) => isActive ? 'active' : ''}
          >
            Contact
          </NavLink>
        </li>
        <li><Link to="/users">Users</Link></li>
        <li><Link to="/dashboard">Dashboard</Link></li>
      </ul>
    </nav>
  );
}

// Basic route components
function Home() {
  const navigate = useNavigate();
  
  return (
    <div>
      <h1>Home Page</h1>
      <button onClick={() => navigate('/about')}>
        Go to About
      </button>
    </div>
  );
}

function About() {
  const location = useLocation();
  
  return (
    <div>
      <h1>About Page</h1>
      <p>Current path: {location.pathname}</p>
    </div>
  );
}

function Contact() {
  return <h1>Contact Page</h1>;
}

// Nested routes with Outlet
function UsersLayout() {
  return (
    <div>
      <h1>Users Section</h1>
      <nav>
        <Link to="/users">All Users</Link> | 
        <Link to="/users/create">Create User</Link>
      </nav>
      <Outlet /> {/* Child routes render here */}
    </div>
  );
}

function UsersList() {
  const users = [
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
    { id: 3, name: 'Charlie' }
  ];

  return (
    <div>
      <h2>Users List</h2>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            <Link to={`/users/${user.id}`}>{user.name}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
}

// Route with parameters
function UserProfile() {
  const { userId } = useParams();
  const navigate = useNavigate();
  
  // In real app, you'd fetch user data based on userId
  const user = { id: userId, name: `User ${userId}`, email: `user${userId}@example.com` };
  
  return (
    <div>
      <h2>User Profile</h2>
      <p>ID: {user.id}</p>
      <p>Name: {user.name}</p>
      <p>Email: {user.email}</p>
      <button onClick={() => navigate('/users')}>
        Back to Users
      </button>
    </div>
  );
}

function CreateUser() {
  const navigate = useNavigate();
  const [name, setName] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    // Create user logic here
    console.log('Creating user:', name);
    navigate('/users');
  };
  
  return (
    <div>
      <h2>Create User</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
          placeholder="User name"
          required
        />
        <button type="submit">Create</button>
      </form>
    </div>
  );
}

// Protected route component
function ProtectedRoute({ children }) {
  const isAuthenticated = false; // Replace with real auth logic
  
  return isAuthenticated ? children : <Navigate to="/login" />;
}

function Dashboard() {
  return <h1>Dashboard (Protected)</h1>;
}

function Login() {
  const navigate = useNavigate();
  const location = useLocation();
  
  const handleLogin = () => {
    // Login logic here
    const from = location.state?.from?.pathname || '/dashboard';
    navigate(from, { replace: true });
  };
  
  return (
    <div>
      <h1>Login</h1>
      <button onClick={handleLogin}>Login</button>
    </div>
  );
}

function NotFound() {
  return (
    <div>
      <h1>404 - Page Not Found</h1>
      <Link to="/">Go Home</Link>
    </div>
  );
}
```

**Installation:**
```bash
npm install react-router-dom
```

**Explanation:** React Router enables client-side routing. Key concepts:
- `BrowserRouter`: Wraps the app for routing
- `Routes`/`Route`: Define route mappings
- `Link`/`NavLink`: Navigation components
- `useNavigate`: Programmatic navigation
- `useParams`: Access URL parameters
- `useLocation`: Access current location
- `Outlet`: Renders child routes in nested routing
- `Navigate`: Redirect component

---

## Styling

Different approaches to styling React components.

### CSS Modules
```jsx
// Button.module.css
.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.primary {
  background-color: #007bff;
  color: white;
}

.secondary {
  background-color: #6c757d;
  color: white;
}

.button:hover {
  opacity: 0.8;
}

// Button.jsx
import styles from './Button.module.css';

function Button({ variant = 'primary', children, ...props }) {
  return (
    <button 
      className={`${styles.button} ${styles[variant]}`}
      {...props}
    >
      {children}
    </button>
  );
}
```

### Styled Components (CSS-in-JS)
```jsx
import styled from 'styled-components';

const StyledButton = styled.button`
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  background-color: ${props => props.primary ? '#007bff' : '#6c757d'};
  color: white;
  
  &:hover {
    opacity: 0.8;
  }
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`;

function Button({ primary, children, ...props }) {
  return (
    <StyledButton primary={primary} {...props}>
      {children}
    </StyledButton>
  );
}
```

### Tailwind CSS
```jsx
function Button({ variant = 'primary', size = 'md', children, ...props }) {
  const baseClasses = 'font-medium rounded focus:outline-none focus:ring-2 focus:ring-offset-2';
  
  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
    secondary: 'bg-gray-600 text-white hover:bg-gray-700 focus:ring-gray-500',
    outline: 'border border-gray-300 text-gray-700 hover:bg-gray-50 focus:ring-blue-500'
  };
  
  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  };
  
  const classes = `${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]}`;
  
  return (
    <button className={classes} {...props}>
      {children}
    </button>
  );
}

// Usage
function App() {
  return (
    <div className="p-4 space-y-4">
      <h1 className="text-3xl font-bold text-gray-900">My App</h1>
      
      <div className="flex space-x-2">
        <Button variant="primary">Primary</Button>
        <Button variant="secondary">Secondary</Button>
        <Button variant="outline">Outline</Button>
      </div>
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        <div className="bg-white p-6 rounded-lg shadow">
          <h2 className="text-xl font-semibold mb-2">Card Title</h2>
          <p className="text-gray-600">Card content goes here.</p>
        </div>
      </div>
    </div>
  );
}
```

### Inline Styles (for dynamic styling)
```jsx
function DynamicButton({ color, size, children }) {
  const styles = {
    backgroundColor: color || '#007bff',
    color: 'white',
    border: 'none',
    padding: `${size * 8}px ${size * 16}px`,
    borderRadius: '4px',
    cursor: 'pointer',
    fontSize: `${14 + size * 2}px`
  };
  
  return (
    <button style={styles}>
      {children}
    </button>
  );
}
```

**Explanation:** 
- **CSS Modules**: Scoped CSS classes, prevents naming conflicts
- **Styled Components**: CSS-in-JS with props support and dynamic styling
- **Tailwind CSS**: Utility-first framework for rapid UI development
- **Inline Styles**: Good for dynamic styling based on props/state
- Choose based on team preferences, project requirements, and bundle size considerations

---

## API Data Fetching

Different patterns for fetching and managing API data.

```jsx
import { useState, useEffect, useCallback } from 'react';

// Custom hook for API calls with error handling
function useApiCall(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async (customUrl = url, customOptions = options) => {
    if (!customUrl) return;

    try {
      setLoading(true);
      setError(null);

      const response = await fetch(customUrl, {
        headers: {
          'Content-Type': 'application/json',
          ...customOptions.headers
        },
        ...customOptions
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const result = await response.json();
      setData(result);
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, [url, options]);

  useEffect(() => {
    if (url) {
      fetchData();
    }
  }, [fetchData, url]);

  return { data, loading, error, refetch: fetchData };
}

// Posts component with CRUD operations
function PostsManager() {
  const [posts, setPosts] = useState([]);
  const [selectedPost, setSelectedPost] = useState(null);
  const [isEditing, setIsEditing] = useState(false);
  
  // Fetch all posts
  const { data: fetchedPosts, loading: postsLoading, error: postsError } = useApiCall(
    'https://jsonplaceholder.typicode.com/posts'
  );

  useEffect(() => {
    if (fetchedPosts) {
      setPosts(fetchedPosts.slice(0, 10)); // Limit to 10 posts
    }
  }, [fetchedPosts]);

  // Create new post
  const createPost = async (postData) => {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(postData)
      });

      if (!response.ok) throw new Error('Failed to create post');
      
      const newPost = await response.json();
      setPosts(prev => [newPost, ...prev]);
      return newPost;
    } catch (error) {
      console.error('Error creating post:', error);
      throw error;
    }
  };

  // Update post
  const updatePost = async (id, postData) => {
    try {
      const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(postData)
      });

      if (!response.ok) throw new Error('Failed to update post');
      
      const updatedPost = await response.json();
      setPosts(prev => prev.map(post => 
        post.id === id ? updatedPost : post
      ));
      return updatedPost;
    } catch (error) {
      console.error('Error updating post:', error);
      throw error;
    }
  };

  // Delete post
  const deletePost = async (id) => {
    try {
      const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
        method: 'DELETE'
      });

      if (!response.ok) throw new Error('Failed to delete post');
      
      setPosts(prev => prev.filter(post => post.id !== id));
    } catch (error) {
      console.error('Error deleting post:', error);
      throw error;
    }
  };

  if (postsLoading) return <div>Loading posts...</div>;
  if (postsError) return <div>Error loading posts: {postsError}</div>;

  return (
    <div>
      <h1>Posts Manager</h1>
      
      <PostForm 
        onSubmit={createPost}
        post={isEditing ? selectedPost : null}
        onUpdate={updatePost}
        isEditing={isEditing}
        onCancel={() => {
          setSelectedPost(null);
          setIsEditing(false);
        }}
      />

      <PostsList 
        posts={posts}
        onEdit={(post) => {
          setSelectedPost(post);
          setIsEditing(true);
        }}
        onDelete={deletePost}
      />
    </div>
  );
}

// Post form component
function PostForm({ onSubmit, onUpdate, post, isEditing, onCancel }) {
  const [title, setTitle] = useState('');
  const [body, setBody] = useState('');
  const [submitting, setSubmitting] = useState(false);

  useEffect(() => {
    if (isEditing && post) {
      setTitle(post.title);
      setBody(post.body);
    } else {
      setTitle('');
      setBody('');
    }
  }, [isEditing, post]);

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!title.trim() || !body.trim()) return;

    try {
      setSubmitting(true);
      
      if (isEditing) {
        await onUpdate(post.id, { title, body, userId: 1 });
        onCancel();
      } else {
        await onSubmit({ title, body, userId: 1 });
        setTitle('');
        setBody('');
      }
    } catch (error) {
      alert('Error saving post: ' + error.message);
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="post-form">
      <h2>{isEditing ? 'Edit Post' : 'Create New Post'}</h2>
      
      <div>
        <input
          type="text"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          placeholder="Post title"
          required
        />
      </div>
      
      <div>
        <textarea
          value={body}
          onChange={(e) => setBody(e.target.value)}
          placeholder="Post content"
          rows={4}
          required
        />
      </div>
      
      <div>
        <button type="submit" disabled={submitting}>
          {submitting ? 'Saving...' : (isEditing ? 'Update' : 'Create')}
        </button>
        
        {isEditing && (
          <button type="button" onClick={onCancel}>
            Cancel
          </button>
        )}
      </div>
    </form>
  );
}

// Posts list component
function PostsList({ posts, onEdit, onDelete }) {
  return (
    <div className="posts-list">
      <h2>Posts ({posts.length})</h2>
      
      {posts.map(post => (
        <PostItem 
          key={post.id}
          post={post}
          onEdit={onEdit}
          onDelete={onDelete}
        />
      ))}
    </div>
  );
}

// Individual post item
function PostItem({ post, onEdit, onDelete }) {
  const [deleting, setDeleting] = useState(false);

  const handleDelete = async () => {
    if (!window.confirm('Are you sure you want to delete this post?')) return;
    
    try {
      setDeleting(true);
      await onDelete(post.id);
    } catch (error) {
      alert('Error deleting post: ' + error.message);
    } finally {
      setDeleting(false);
    }
  };

  return (
    <div className="post-item">
      <h3>{post.title}</h3>
      <p>{post.body}</p>
      
      <div className="post-actions">
        <button onClick={() => onEdit(post)}>Edit</button>
        <button 
          onClick={handleDelete} 
          disabled={deleting}
          className="delete-btn"
        >
          {deleting ? 'Deleting...' : 'Delete'}
        </button>
      </div>
    </div>
  );
}

// Advanced: Using React Query (install with: npm install @tanstack/react-query)
/*
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function PostsWithReactQuery() {
  const queryClient = useQueryClient();

  // Fetch posts
  const { data: posts, isLoading, error } = useQuery({
    queryKey: ['posts'],
    queryFn: () => fetch('/api/posts').then(res => res.json())
  });

  // Create post mutation
  const createPostMutation = useMutation({
    mutationFn: (newPost) => 
      fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newPost)
      }).then(res => res.json()),
    onSuccess: () => {
      queryClient.invalidateQueries(['posts']);
    }
  });

  return (
    <div>
      {isLoading && <div>Loading...</div>}
      {error && <div>Error: {error.message}</div>}
      {posts && (
        <div>
          {posts.map(post => (
            <div key={post.id}>{post.title}</div>
          ))}
        </div>
      )}
    </div>
  );
}
*/
```

**Explanation:** API fetching patterns include:
- Custom hooks for reusable API logic
- Error handling and loading states
- CRUD operations (Create, Read, Update, Delete)
- Optimistic updates for better UX
- Consider using libraries like React Query or SWR for advanced caching and synchronization

---

## Error Handling

Implement comprehensive error handling in React applications.

```jsx
import { Component, useState, useEffect } from 'react';

// Error Boundary Class Component
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { 
      hasError: false, 
      error: null, 
      errorInfo: null 
    };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Log error to console or error reporting service
    console.error('Error Boundary caught an error:', error, errorInfo);
    
    this.setState({
      error: error,
      errorInfo: errorInfo
    });

    // You can also log the error to an error reporting service here
    // logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Custom fallback UI
      return (
        <div className="error-boundary">
          <h2>Something went wrong!</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            <summary>Error Details (click to expand)</summary>
            <p><strong>Error:</strong> {this.state.error && this.state.error.toString()}</p>
            <p><strong>Stack trace:</strong></p>
            <code>{this.state.errorInfo.componentStack}</code>
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

// Custom hook for error handling
function useErrorHandler() {
  const [error, setError] = useState(null);

  const handleError = (error) => {
    console.error('Error occurred:', error);
    setError(error);
    
    // Optional: Send to error reporting service
    // reportError(error);
  };

  const clearError = () => setError(null);

  return { error, handleError, clearError };
}

// Component with various error scenarios
function ErrorProneComponent() {
  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const { error, handleError, clearError } = useErrorHandler();

  // Async error handling
  const fetchData = async () => {
    try {
      setLoading(true);
      clearError();
      
      const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      handleError(err);
    } finally {
      setLoading(false);
    }
  };

  // Function that might throw an error
  const riskyOperation = () => {
    try {
      if (count > 5) {
        throw new Error('Count is too high!');
      }
      setCount(count + 1);
    } catch (err) {
      handleError(err);
    }
  };

  // Trigger JavaScript error (will be caught by Error Boundary)
  const triggerError = () => {
    throw new Error('This is a deliberate error for testing Error Boundary');
  };

  if (error) {
    return (
      <div className="error-display">
        <h3>An error occurred:</h3>
        <p>{error.message}</p>
        <button onClick={clearError}>Try Again</button>
      </div>
    );
  }

  return (
    <div>
      <h2>Error Handling Demo</h2>
      
      <div>
        <p>Count: {count}</p>
        <button onClick={riskyOperation}>
          Increment (throws error if > 5)
        </button>
        <button onClick={() => setCount(0)}>Reset</button>
      </div>

      <div>
        <button onClick={fetchData} disabled={loading}>
          {loading ? 'Loading...' : 'Fetch Data'}
        </button>
        {data && (
          <div>
            <h4>Fetched Data:</h4>
            <p>{data.title}</p>
          </div>
        )}
      </div>

      <div>
        <button onClick={triggerError}>
          Trigger Error Boundary
        </button>
      </div>
    </div>
  );
}

// Retry mechanism component
function RetryableComponent() {
  const [retryCount, setRetryCount] = useState(0);
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const maxRetries = 3;

  const fetchWithRetry = async (attempt = 0) => {
    try {
      setLoading(true);
      setError(null);

      // Simulate API call that might fail
      const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${Math.random() > 0.7 ? '1' : 'invalid'}`);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const result = await response.json();
      setData(result);
      setRetryCount(0);
    } catch (err) {
      if (attempt < maxRetries) {
        console.log(`Attempt ${attempt + 1} failed, retrying...`);
        setRetryCount(attempt + 1);
        
        // Exponential backoff
        setTimeout(() => {
          fetchWithRetry(attempt + 1);
        }, Math.pow(2, attempt) * 1000);
      } else {
        setError(err);
        setRetryCount(0);
      }
    } finally {
      if (attempt === maxRetries || data) {
        setLoading(false);
      }
    }
  };

  return (
    <div>
      <h3>Retry Mechanism Demo</h3>
      <button onClick={() => fetchWithRetry()}>
        Fetch Data (70% chance of failure)
      </button>
      
      {loading && (
        <p>
          Loading... 
          {retryCount > 0 && ` (Retry ${retryCount}/${maxRetries})`}
        </p>
      )}
      
      {error && (
        <div className="error">
          <p>Error: {error.message}</p>
          <button onClick={() => fetchWithRetry()}>Try Again</button>
        </div>
      )}
      
      {data && (
        <div>
          <h4>Success!</h4>
          <p>Title: {data.title}</p>
        </div>
      )}
    </div>
  );
}

// Global error handler
function GlobalErrorHandler({ children }) {
  useEffect(() => {
    const handleUnhandledRejection = (event) => {
      console.error('Unhandled promise rejection:', event.reason);
      // Optionally send to error reporting service
    };

    const handleError = (event) => {
      console.error('Global error:', event.error);
      // Optionally send to error reporting service
    };

    window.addEventListener('unhandledrejection', handleUnhandledRejection);
    window.addEventListener('error', handleError);

    return () => {
      window.removeEventListener('unhandledrejection', handleUnhandledRejection);
      window.removeEventListener('error', handleError);
    };
  }, []);

  return children;
}

// Main App with error handling
function App() {
  return (
    <GlobalErrorHandler>
      <ErrorBoundary>
        <div className="app">
          <h1>React Error Handling</h1>
          <ErrorProneComponent />
          <RetryableComponent />
        </div>
      </ErrorBoundary>
    </GlobalErrorHandler>
  );
}
```

**Explanation:** Error handling in React involves:
- **Error Boundaries**: Catch JavaScript errors in component tree
- **Try-catch blocks**: Handle errors in async operations
- **Custom error hooks**: Centralize error handling logic
- **Retry mechanisms**: Automatic retry with exponential backoff
- **Global error handlers**: Catch unhandled errors
- Always provide user-friendly error messages and recovery options

---

## Best Practices

Essential best practices for React development.

### Performance Optimization

```jsx
import { memo, useCallback, useMemo, lazy, Suspense } from 'react';

// 1. Memoize expensive components
const ExpensiveComponent = memo(function ExpensiveComponent({ data, onUpdate }) {
  const processedData = useMemo(() => {
    return data.map(item => ({
      ...item,
      processed: item.value * 2
    }));
  }, [data]);

  const handleUpdate = useCallback((id, value) => {
    onUpdate(id, value);
  }, [onUpdate]);

  return (
    <div>
      {processedData.map(item => (
        <ItemComponent 
          key={item.id}
          item={item}
          onUpdate={handleUpdate}
        />
      ))}
    </div>
  );
});

// 2. Lazy loading components
const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}

// 3. Optimize re-renders
const ItemComponent = memo(({ item, onUpdate }) => {
  console.log(`Rendering item ${item.id}`);
  
  return (
    <div>
      <span>{item.name}: {item.processed}</span>
      <button onClick={() => onUpdate(item.id, item.value + 1)}>
        Update
      </button>
    </div>
  );
});
```

### Code Organization

```jsx
// utils/constants.js
export const API_ENDPOINTS = {
  USERS: '/api/users',
  POSTS: '/api/posts'
};

export const STORAGE_KEYS = {
  USER_TOKEN: 'userToken',
  THEME: 'theme'
};

// hooks/useLocalStorage.js
import { useState, useCallback } from 'react';

export function useLocalStorage(key, initialValue) {
  // Implementation here...
}

// components/Button/Button.jsx
import PropTypes from 'prop-types';
import './Button.css';

export function Button({ variant, size, children, ...props }) {
  return (
    <button 
      className={`btn btn--${variant} btn--${size}`}
      {...props}
    >
      {children}
    </button>
  );
}

Button.propTypes = {
  variant: PropTypes.oneOf(['primary', 'secondary', 'outline']),
  size: PropTypes.oneOf(['sm', 'md', 'lg']),
  children: PropTypes.node.isRequired
};

Button.defaultProps = {
  variant: 'primary',
  size: 'md'
};

// components/Button/index.js
export { Button } from './Button';

// services/api.js
class ApiService {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...options
    };

    const response = await fetch(url, config);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return response.json();
  }

  get(endpoint) {
    return this.request(endpoint);
  }

  post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
}

export const api = new ApiService('/api');
```

### Security Best Practices

```jsx
import DOMPurify from 'dompurify';

// 1. Sanitize user input
function UserContent({ htmlContent }) {
  const sanitizedHTML = useMemo(() => {
    return DOMPurify.sanitize(htmlContent);
  }, [htmlContent]);

  return (
    <div dangerouslySetInnerHTML={{ __html: sanitizedHTML }} />
  );
}

// 2. Validate props with PropTypes
import PropTypes from 'prop-types';

function UserProfile({ user }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

UserProfile.propTypes = {
  user: PropTypes.shape({
    name: PropTypes.string.isRequired,
    email: PropTypes.string.isRequired
  }).isRequired
};

// 3. Environment variables for sensitive data
const config = {
  apiUrl: process.env.REACT_APP_API_URL,
  apiKey: process.env.REACT_APP_API_KEY // Never expose in client-side code!
};
```

### Testing Best Practices

```jsx
// Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  test('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  test('calls onClick handler when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  test('applies correct CSS classes', () => {
    render(<Button variant="secondary" size="lg">Button</Button>);
    const button = screen.getByRole('button');
    
    expect(button).toHaveClass('btn', 'btn--secondary', 'btn--lg');
  });
});

// Custom hook testing
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('should increment counter', () => {
  const { result } = renderHook(() => useCounter());

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

### Accessibility Best Practices

```jsx
function AccessibleForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});

  return (
    <form role="form" aria-label="Login form">
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          aria-invalid={errors.email ? 'true' : 'false'}
          aria-describedby={errors.email ? 'email-error' : undefined}
          required
        />
        {errors.email && (
          <div id="email-error" role="alert" aria-live="polite">
            {errors.email}
          </div>
        )}
      </div>

      <div>
        <label htmlFor="password">Password:</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          aria-invalid={errors.password ? 'true' : 'false'}
          aria-describedby={errors.password ? 'password-error' : undefined}
          required
        />
        {errors.password && (
          <div id="password-error" role="alert" aria-live="polite">
            {errors.password}
          </div>
        )}
      </div>

      <button type="submit" aria-label="Submit login form">
        Login
      </button>
    </form>
  );
}

// Accessible modal
function Modal({ isOpen, onClose, title, children }) {
  const modalRef = useRef();

  useEffect(() => {
    if (isOpen && modalRef.current) {
      modalRef.current.focus();
    }
  }, [isOpen]);

  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === 'Escape') {
        onClose();
      }
    };

    if (isOpen) {
      document.addEventListener('keydown', handleEscape);
      return () => document.removeEventListener('keydown', handleEscape);
    }
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div
      className="modal-overlay"
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
    >
      <div
        ref={modalRef}
        className="modal-content"
        tabIndex={-1}
      >
        <div className="modal-header">
          <h2 id="modal-title">{title}</h2>
          <button
            onClick={onClose}
            aria-label="Close modal"
            className="modal-close"
          >
            ×
          </button>
        </div>
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>
  );
}
```

### General Best Practices Summary

**Component Design:**
- Keep components small and focused on a single responsibility
- Use functional components with hooks instead of class components
- Prefer composition over inheritance
- Extract reusable logic into custom hooks

**State Management:**
- Use local state for component-specific data
- Lift state up when multiple components need access
- Consider Context API for deeply nested prop passing
- Use reducers for complex state logic

**Performance:**
- Use React.memo() for expensive components
- Implement useMemo() and useCallback() for expensive calculations
- Lazy load components and routes
- Optimize bundle size with code splitting

**Code Quality:**
- Use TypeScript or PropTypes for type safety
- Write comprehensive tests
- Follow consistent naming conventions
- Use ESLint and Prettier for code formatting

**Security:**
- Sanitize user input before rendering
- Never store sensitive data in client-side code
- Validate all user inputs
- Use HTTPS in production

**Accessibility:**
- Use semantic HTML elements
- Provide proper ARIA labels and roles
- Ensure keyboard navigation works
- Test with screen readers

This cheatsheet covers the essential concepts and patterns you'll need for modern React development. Practice these examples and gradually incorporate the best practices into your projects!