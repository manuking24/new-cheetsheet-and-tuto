# Complete React Native Cheatsheet

A comprehensive guide to React Native development using functional components and hooks.

## Table of Contents
1. [Project Setup](#project-setup)
2. [Folder Structure](#folder-structure)
3. [JSX and Components](#jsx-and-components)
4. [State Management with Hooks](#state-management-with-hooks)
5. [Navigation](#navigation)
6. [Data Passing Between Screens](#data-passing-between-screens)
7. [User Input and Forms](#user-input-and-forms)
8. [Lists and Scrolling](#lists-and-scrolling)
9. [Platform-Specific Code](#platform-specific-code)
10. [API Integration](#api-integration)
11. [Images and Assets](#images-and-assets)
12. [Styling](#styling)
13. [Gestures and Touchables](#gestures-and-touchables)
14. [Permissions](#permissions)
15. [Custom Hooks](#custom-hooks)
16. [AsyncStorage](#asyncstorage)
17. [Device Features](#device-features)
18. [Debugging and Testing](#debugging-and-testing)
19. [Performance Best Practices](#performance-best-practices)

---

## Project Setup

### Using Expo (Recommended for beginners)
```bash
# Install Expo CLI
npm install -g @expo/cli

# Create new project
npx create-expo-app MyApp
cd MyApp

# Start development server
npx expo start
```

### Using React Native CLI
```bash
# Install React Native CLI
npm install -g react-native-cli

# Create new project
npx react-native init MyApp
cd MyApp

# Start Metro bundler
npx react-native start

# Run on iOS/Android (separate terminals)
npx react-native run-ios
npx react-native run-android
```

**When to use:**
- **Expo**: Quick prototyping, simpler setup, managed workflow
- **React Native CLI**: Need custom native modules, full control over build process

---

## Folder Structure

```
MyApp/
├── src/
│   ├── components/          # Reusable UI components
│   ├── screens/            # Screen components
│   ├── navigation/         # Navigation configuration
│   ├── hooks/              # Custom hooks
│   ├── services/           # API calls and external services
│   ├── utils/              # Helper functions
│   ├── constants/          # App constants
│   └── assets/             # Images, fonts, etc.
├── App.js                  # Root component
└── package.json
```

**Explanation:** This structure separates concerns and makes the codebase maintainable. Each folder has a specific purpose, making it easy to locate and organize code.

---

## JSX and Components

### Basic Functional Component
```jsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const WelcomeScreen = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to React Native!</Text>
      <Text style={styles.subtitle}>Building amazing mobile apps</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
  },
});

export default WelcomeScreen;
```

**Explanation:** Functional components are the modern way to create React Native components. They use JSX to describe the UI structure. The `StyleSheet.create()` method optimizes styles and provides better performance than inline styles.

---

## State Management with Hooks

### useState Hook
```jsx
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

const CounterScreen = () => {
  const [count, setCount] = useState(0);
  const [isEven, setIsEven] = useState(true);

  const increment = () => {
    const newCount = count + 1;
    setCount(newCount);
    setIsEven(newCount % 2 === 0);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.countText}>Count: {count}</Text>
      <Text style={styles.statusText}>
        {isEven ? 'Even' : 'Odd'}
      </Text>
      <TouchableOpacity style={styles.button} onPress={increment}>
        <Text style={styles.buttonText}>Increment</Text>
      </TouchableOpacity>
    </View>
  );
};
```

### useEffect Hook
```jsx
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet } from 'react-native';

const TimerScreen = () => {
  const [seconds, setSeconds] = useState(0);
  const [isActive, setIsActive] = useState(true);

  useEffect(() => {
    let interval = null;
    
    if (isActive) {
      interval = setInterval(() => {
        setSeconds(seconds => seconds + 1);
      }, 1000);
    }

    // Cleanup function
    return () => {
      if (interval) clearInterval(interval);
    };
  }, [isActive]); // Dependency array

  useEffect(() => {
    console.log('Component mounted');
    
    // Cleanup on unmount
    return () => {
      console.log('Component will unmount');
    };
  }, []); // Empty dependency array = run once on mount

  return (
    <View style={styles.container}>
      <Text style={styles.timerText}>{seconds} seconds</Text>
    </View>
  );
};
```

**Explanation:** `useState` manages component state, while `useEffect` handles side effects like timers, API calls, and subscriptions. The dependency array controls when the effect runs.

---

## Navigation

### Installation
```bash
npm install @react-navigation/native
npx expo install react-native-screens react-native-safe-area-context

# For Stack Navigator
npm install @react-navigation/native-stack

# For Tab Navigator
npm install @react-navigation/bottom-tabs

# For Drawer Navigator
npm install @react-navigation/drawer
```

### Stack Navigation
```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import HomeScreen from './src/screens/HomeScreen';
import DetailsScreen from './src/screens/DetailsScreen';

const Stack = createNativeStackNavigator();

const App = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen 
          name="Home" 
          component={HomeScreen}
          options={{ title: 'Welcome' }}
        />
        <Stack.Screen 
          name="Details" 
          component={DetailsScreen}
          options={{ title: 'Details' }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

### Tab Navigation
```jsx
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { Ionicons } from '@expo/vector-icons';

const Tab = createBottomTabNavigator();

const TabNavigator = () => {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName;
          
          if (route.name === 'Home') {
            iconName = focused ? 'home' : 'home-outline';
          } else if (route.name === 'Settings') {
            iconName = focused ? 'settings' : 'settings-outline';
          }
          
          return <Ionicons name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: 'tomato',
        tabBarInactiveTintColor: 'gray',
      })}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Settings" component={SettingsScreen} />
    </Tab.Navigator>
  );
};
```

**Explanation:** React Navigation provides different navigation patterns. Stack navigation allows pushing/popping screens, while tab navigation provides bottom tabs. Configure options to customize appearance and behavior.

---

## Data Passing Between Screens

### Passing Parameters
```jsx
// HomeScreen.js
import React from 'react';
import { View, Text, TouchableOpacity } from 'react-native';

const HomeScreen = ({ navigation }) => {
  const goToDetails = () => {
    navigation.navigate('Details', {
      itemId: 86,
      otherParam: 'Hello from Home!',
      user: { name: 'John', age: 30 }
    });
  };

  return (
    <View>
      <TouchableOpacity onPress={goToDetails}>
        <Text>Go to Details</Text>
      </TouchableOpacity>
    </View>
  );
};

// DetailsScreen.js
const DetailsScreen = ({ route, navigation }) => {
  // Extract parameters
  const { itemId, otherParam, user } = route.params;

  const goBack = () => {
    navigation.goBack();
  };

  const goBackWithData = () => {
    navigation.navigate('Home', {
      result: 'Data from Details screen'
    });
  };

  return (
    <View>
      <Text>Item ID: {itemId}</Text>
      <Text>Message: {otherParam}</Text>
      <Text>User: {user.name}, Age: {user.age}</Text>
      <TouchableOpacity onPress={goBack}>
        <Text>Go Back</Text>
      </TouchableOpacity>
    </View>
  );
};
```

### Using Navigation Events
```jsx
import React, { useEffect } from 'react';
import { useFocusEffect } from '@react-navigation/native';

const ProfileScreen = ({ navigation }) => {
  // Run when screen comes into focus
  useFocusEffect(
    React.useCallback(() => {
      console.log('Screen focused');
      
      return () => {
        console.log('Screen unfocused');
      };
    }, [])
  );

  useEffect(() => {
    const unsubscribe = navigation.addListener('beforeRemove', (e) => {
      // Prevent default behavior of leaving the screen
      e.preventDefault();
      
      // Prompt the user before leaving the screen
      Alert.alert(
        'Discard changes?',
        'You have unsaved changes. Are you sure to discard them and leave the screen?',
        [
          { text: "Don't leave", style: 'cancel', onPress: () => {} },
          {
            text: 'Discard',
            style: 'destructive',
            onPress: () => navigation.dispatch(e.data.action),
          },
        ]
      );
    });

    return unsubscribe;
  }, [navigation]);

  return (
    // Your component JSX
  );
};
```

**Explanation:** Pass data between screens using route parameters. Use navigation events to handle screen focus/blur and prevent navigation when needed.

---

## User Input and Forms

### Text Input and Form Handling
```jsx
import React, { useState } from 'react';
import { 
  View, 
  Text, 
  TextInput, 
  TouchableOpacity, 
  Alert,
  KeyboardAvoidingView,
  Platform 
} from 'react-native';

const LoginForm = () => {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
  });
  const [errors, setErrors] = useState({});

  const validateForm = () => {
    const newErrors = {};
    
    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 6) {
      newErrors.password = 'Password must be at least 6 characters';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = () => {
    if (validateForm()) {
      Alert.alert('Success', 'Form submitted successfully!');
      // Handle form submission
    }
  };

  const updateField = (field, value) => {
    setFormData(prev => ({
      ...prev,
      [field]: value
    }));
    
    // Clear error when user starts typing
    if (errors[field]) {
      setErrors(prev => ({
        ...prev,
        [field]: ''
      }));
    }
  };

  return (
    <KeyboardAvoidingView 
      style={styles.container}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    >
      <Text style={styles.title}>Login</Text>
      
      <View style={styles.inputContainer}>
        <TextInput
          style={[styles.input, errors.email && styles.inputError]}
          placeholder="Email"
          value={formData.email}
          onChangeText={(value) => updateField('email', value)}
          keyboardType="email-address"
          autoCapitalize="none"
          autoCorrect={false}
        />
        {errors.email && <Text style={styles.errorText}>{errors.email}</Text>}
      </View>

      <View style={styles.inputContainer}>
        <TextInput
          style={[styles.input, errors.password && styles.inputError]}
          placeholder="Password"
          value={formData.password}
          onChangeText={(value) => updateField('password', value)}
          secureTextEntry
          autoCapitalize="none"
        />
        {errors.password && <Text style={styles.errorText}>{errors.password}</Text>}
      </View>

      <TouchableOpacity style={styles.button} onPress={handleSubmit}>
        <Text style={styles.buttonText}>Login</Text>
      </TouchableOpacity>
    </KeyboardAvoidingView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
  },
  inputContainer: {
    marginBottom: 15,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    padding: 15,
    borderRadius: 8,
    fontSize: 16,
  },
  inputError: {
    borderColor: '#ff6b6b',
  },
  errorText: {
    color: '#ff6b6b',
    fontSize: 12,
    marginTop: 5,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 10,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});
```

**Explanation:** Handle form input with controlled components using `useState`. Validate data and provide user feedback. `KeyboardAvoidingView` prevents keyboard from covering input fields.

---

## Lists and Scrolling

### ScrollView for Small Lists
```jsx
import React from 'react';
import { ScrollView, View, Text, StyleSheet } from 'react-native';

const ScrollViewExample = () => {
  const items = Array.from({ length: 20 }, (_, i) => `Item ${i + 1}`);

  return (
    <ScrollView 
      style={styles.container}
      showsVerticalScrollIndicator={false}
      contentContainerStyle={styles.contentContainer}
    >
      {items.map((item, index) => (
        <View key={index} style={styles.item}>
          <Text style={styles.itemText}>{item}</Text>
        </View>
      ))}
    </ScrollView>
  );
};
```

### FlatList for Large Lists
```jsx
import React, { useState, useEffect } from 'react';
import { 
  FlatList, 
  View, 
  Text, 
  TouchableOpacity, 
  ActivityIndicator,
  RefreshControl 
} from 'react-native';

const UserList = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);
  const [page, setPage] = useState(1);

  const fetchUsers = async (pageNum = 1, isRefresh = false) => {
    try {
      const response = await fetch(`https://jsonplaceholder.typicode.com/users?_page=${pageNum}&_limit=10`);
      const newUsers = await response.json();
      
      if (isRefresh) {
        setUsers(newUsers);
      } else {
        setUsers(prev => [...prev, ...newUsers]);
      }
    } catch (error) {
      console.error('Error fetching users:', error);
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  const onRefresh = () => {
    setRefreshing(true);
    setPage(1);
    fetchUsers(1, true);
  };

  const loadMore = () => {
    if (!loading) {
      setLoading(true);
      setPage(prev => prev + 1);
      fetchUsers(page + 1);
    }
  };

  const renderUser = ({ item }) => (
    <TouchableOpacity style={styles.userItem}>
      <Text style={styles.userName}>{item.name}</Text>
      <Text style={styles.userEmail}>{item.email}</Text>
    </TouchableOpacity>
  );

  const renderFooter = () => {
    if (!loading) return null;
    return (
      <View style={styles.footerLoader}>
        <ActivityIndicator size="small" color="#007AFF" />
      </View>
    );
  };

  const renderEmpty = () => (
    <View style={styles.emptyContainer}>
      <Text style={styles.emptyText}>No users found</Text>
    </View>
  );

  return (
    <FlatList
      data={users}
      renderItem={renderUser}
      keyExtractor={(item) => item.id.toString()}
      onEndReached={loadMore}
      onEndReachedThreshold={0.1}
      ListFooterComponent={renderFooter}
      ListEmptyComponent={renderEmpty}
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
      }
      showsVerticalScrollIndicator={false}
    />
  );
};
```

**Explanation:** Use `ScrollView` for small, static lists and `FlatList` for large, dynamic lists. `FlatList` is optimized for performance with virtualization and provides built-in features like pull-to-refresh and infinite scrolling.

---

## Platform-Specific Code

### Using Platform.OS
```jsx
import React from 'react';
import { View, Text, StyleSheet, Platform } from 'react-native';

const PlatformSpecificComponent = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.platformText}>
        Running on: {Platform.OS}
      </Text>
      <Text style={styles.versionText}>
        {Platform.OS === 'ios' 
          ? `iOS Version: ${Platform.Version}`
          : `Android API Level: ${Platform.Version}`
        }
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    paddingTop: Platform.OS === 'ios' ? 20 : 0,
  },
  platformText: {
    fontSize: 18,
    fontWeight: 'bold',
    ...Platform.select({
      ios: {
        fontFamily: 'Arial',
        color: '#007AFF',
      },
      android: {
        fontFamily: 'Roboto',
        color: '#4CAF50',
      },
    }),
  },
  versionText: {
    fontSize: 14,
    marginTop: 10,
    color: Platform.OS === 'ios' ? '#666' : '#999',
  },
});
```

### Platform-Specific Files
```jsx
// Button.ios.js
export default function Button() {
  return <Text>iOS Button</Text>;
}

// Button.android.js  
export default function Button() {
  return <Text>Android Button</Text>;
}

// Usage
import Button from './Button'; // Automatically picks the right file
```

**Explanation:** Use `Platform.OS` to detect the current platform and `Platform.select()` to apply platform-specific styles or behavior. You can also create separate files for each platform.

---

## API Integration

### Fetch with Error Handling
```jsx
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, ActivityIndicator, Alert } from 'react-native';

const PostsList = () => {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchPosts = async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch('https://jsonplaceholder.typicode.com/posts');
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const data = await response.json();
      setPosts(data);
    } catch (err) {
      setError(err.message);
      Alert.alert('Error', 'Failed to fetch posts. Please try again.');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchPosts();
  }, []);

  const createPost = async (postData) => {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(postData),
      });
      
      if (!response.ok) {
        throw new Error('Failed to create post');
      }
      
      const newPost = await response.json();
      setPosts(prev => [newPost, ...prev]);
      Alert.alert('Success', 'Post created successfully!');
    } catch (err) {
      Alert.alert('Error', err.message);
    }
  };

  if (loading) {
    return (
      <View style={styles.centerContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text>Loading posts...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.centerContainer}>
        <Text style={styles.errorText}>Error: {error}</Text>
        <TouchableOpacity onPress={fetchPosts} style={styles.retryButton}>
          <Text style={styles.retryText}>Retry</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <FlatList
      data={posts}
      renderItem={({ item }) => (
        <View style={styles.postItem}>
          <Text style={styles.postTitle}>{item.title}</Text>
          <Text style={styles.postBody}>{item.body}</Text>
        </View>
      )}
      keyExtractor={(item) => item.id.toString()}
    />
  );
};
```

### Using Axios (Alternative)
```jsx
import axios from 'axios';

// Create axios instance with base configuration
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Add request interceptor
api.interceptors.request.use(
  (config) => {
    // Add auth token if available
    const token = getAuthToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Add response interceptor
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Handle unauthorized access
      logout();
    }
    return Promise.reject(error);
  }
);

const ApiService = {
  getPosts: () => api.get('/posts'),
  createPost: (data) => api.post('/posts', data),
  updatePost: (id, data) => api.put(`/posts/${id}`, data),
  deletePost: (id) => api.delete(`/posts/${id}`),
};
```

**Explanation:** Handle API calls with proper error handling and loading states. Use interceptors with Axios for global request/response handling. Always provide user feedback for loading and error states.

---

## Images and Assets

### Local Images
```jsx
import React from 'react';
import { View, Image, StyleSheet } from 'react-native';

const ImageExample = () => {
  return (
    <View style={styles.container}>
      {/* Static local image */}
      <Image 
        source={require('../assets/logo.png')} 
        style={styles.logo}
        resizeMode="contain"
      />
      
      {/* Platform-specific images */}
      <Image 
        source={require('../assets/icon.png')} 
        style={styles.icon}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  logo: {
    width: 200,
    height: 100,
  },
  icon: {
    width: 50,
    height: 50,
    marginTop: 20,
  },
});
```

### Remote Images with Loading States
```jsx
import React, { useState } from 'react';
import { 
  View, 
  Image, 
  ActivityIndicator, 
  Text, 
  StyleSheet 
} from 'react-native';

const RemoteImage = ({ uri, style }) => {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(false);

  return (
    <View style={[styles.imageContainer, style]}>
      {loading && (
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="small" color="#007AFF" />
        </View>
      )}
      
      {error ? (
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>Failed to load image</Text>
        </View>
      ) : (
        <Image
          source={{ uri }}
          style={styles.image}
          onLoadStart={() => setLoading(true)}
          onLoadEnd={() => setLoading(false)}
          onError={() => {
            setLoading(false);
            setError(true);
          }}
          resizeMode="cover"
        />
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  imageContainer: {
    backgroundColor: '#f0f0f0',
    justifyContent: 'center',
    alignItems: 'center',
  },
  image: {
    width: '100%',
    height: '100%',
  },
  loadingContainer: {
    position: 'absolute',
    zIndex: 1,
  },
  errorContainer: {
    justifyContent: 'center',
    alignItems: 'center',
    width: '100%',
    height: '100%',
  },
  errorText: {
    color: '#666',
    fontSize: 12,
  },
});
```

### Background Images
```jsx
import React from 'react';
import { ImageBackground, Text, StyleSheet } from 'react-native';

const BackgroundImageExample = () => {
  return (
    <ImageBackground
      source={{ uri: 'https://example.com/background.jpg' }}
      style={styles.background}
      resizeMode="cover"
    >
      <Text style={styles.text}>Content over background</Text>
    </ImageBackground>
  );
};

const styles = StyleSheet.create({
  background: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  text: {
    color: 'white',
    fontSize: 20,
    fontWeight: 'bold',
    textAlign: 'center',
    backgroundColor: 'rgba(0,0,0,0.5)',
    padding: 10,
    borderRadius: 5,
  },
});
```

**Explanation:** Use `require()` for local images and `uri` for remote images. Handle loading states for better UX. `ImageBackground` allows overlaying content on images.

---

## Styling

### StyleSheet Basics
```jsx
import React from 'react';
import { View, Text, StyleSheet, Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');

const StylingExample = () => {
  return (
    <View style={styles.container}>
      <View style={styles.card}>
        <Text style={styles.title}>Card Title</Text>
        <Text style={styles.description}>This is a description</Text>
        <View style={styles.buttonContainer}>
          <View style={[styles.button, styles.primaryButton]}>
            <Text style={[styles.buttonText, styles.primaryButtonText]}>
              Primary
            </Text>
          </View>
          <View style={[styles.button, styles.secondaryButton]}>
            <Text style={[styles.buttonText, styles.secondaryButtonText]}>
              Secondary
            </Text>
          </View>
        </View>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
    padding: 20,
    justifyContent: 'center',
  },
  card: {
    backgroundColor: 'white',
    borderRadius: 12,
    padding: 20,
    shadowColor: '#000',
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 5, // Android shadow
    width: width - 40,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  description: {
    fontSize: 16,
    color: '#666',
    lineHeight: 24,
    marginBottom: 20,
  },
  buttonContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  button: {
    flex: 1,
    paddingVertical: 12,
    paddingHorizontal: 20,
    borderRadius: 8,
    alignItems: 'center',
    marginHorizontal: 5,
  },
  primaryButton: {
    backgroundColor: '#007AFF',
  },
  secondaryButton: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: '#007AFF',
  },
  buttonText: {
    fontSize: 16,
    fontWeight: '600',
  },
  primaryButtonText: {
    color: 'white',
  },
  secondaryButtonText: {
    color: '#007AFF',
  },
});
```

### Responsive Design
```jsx
import React from 'react';
import { View, Text, StyleSheet, Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');

const ResponsiveComponent = () => {
  const isTablet = width >= 768;
  const isLandscape = width > height;

  return (
    <View style={[
      styles.container,
      isTablet && styles.tabletContainer,
      isLandscape && styles.landscapeContainer
    ]}>
      <Text style={[
        styles.title,
        isTablet && styles.tabletTitle
      ]}>
        Responsive Text
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  tabletContainer: {
    padding: 40,
    maxWidth: 800,
    alignSelf: 'center',
  },
  landscapeContainer: {
    flexDirection: 'row',
  },
  title: {
    fontSize: width * 0.05, // 5% of screen width
    textAlign: 'center',
  },
  tabletTitle: {
    fontSize: 32,
  },
});
```

### NativeWind (Tailwind CSS for React Native)
```bash
npm install nativewind
npm install --save-dev tailwindcss@3.3.2
```

```jsx
// tailwind.config.js
module.exports = {
  content: ["./App.{js,jsx,ts,tsx}", "./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
}

// babel.config.js
module.exports = {
  presets: ['babel-preset-expo'],
  plugins: ["nativewind/babel"],
}
```

```jsx
import React from 'react';
import { View, Text } from 'react-native';

const NativeWindExample = () => {
  return (
    <View className="flex-1 bg-gray-100 p-4">
      <View className="bg-white rounded-lg p-6 shadow-lg">
        <Text className="text-2xl font-bold text-gray-800 mb-2">
          NativeWind Example
        </Text>
        <Text className="text-gray-600 mb-4">
          Using Tailwind CSS classes in React Native
        </Text>
        <View className="flex-row justify-between">
          <View className="bg-blue-500 px-4 py-2 rounded-md">
            <Text className="text-white font-semibold">Primary</Text>
          </View>
          <View className="border border-blue-500 px-4 py-2 rounded-md">
            <Text className="text-blue-500 font-semibold">Secondary</Text>
          </View>
        </View>
      </View>
    </View>
  );
};
```

**Explanation:** StyleSheet provides optimized styling. Use Dimensions for responsive design. NativeWind brings Tailwind CSS utility classes to React Native for rapid styling.

---

## Gestures and Touchables

### Basic Touchables
```jsx
import React, { useState } from 'react';
import { 
  View, 
  Text, 
  TouchableOpacity, 
  TouchableHighlight,
  TouchableWithoutFeedback,
  Pressable,
  Alert,
  StyleSheet 
} from 'react-native';

const TouchableExample = () => {
  const [pressCount, setPressCount] = useState(0);

  return (
    <View style={styles.container}>
      {/* TouchableOpacity - Most common */}
      <TouchableOpacity 
        style={styles.button}
        onPress={() => setPressCount(prev => prev + 1)}
        activeOpacity={0.7}
      >
        <Text style={styles.buttonText}>
          TouchableOpacity ({pressCount})
        </Text>
      </TouchableOpacity>

      {/* TouchableHighlight - Shows highlight on press */}
      <TouchableHighlight
        style={styles.button}
        onPress={() => Alert.alert('TouchableHighlight pressed')}
        underlayColor="#0056b3"
      >
        <Text style={styles.buttonText}>TouchableHighlight</Text>
      </TouchableHighlight>

      {/* Pressable - Most flexible */}
      <Pressable
        style={({ pressed }) => [
          styles.button,
          pressed && styles.pressedButton
        ]}
        onPress={() => Alert.alert('Pressable pressed')}
        onLongPress={() => Alert.alert('Long press detected')}
        onPressIn={() => console.log('Press in')}
        onPressOut={() => console.log('Press out')}
      >
        {({ pressed }) => (
          <Text style={[
            styles.buttonText,
            pressed && styles.pressedText
          ]}>
            Pressable {pressed ? '(Pressed)' : ''}
          </Text>
        )}
      </Pressable>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    padding: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginVertical: 10,
  },
  pressedButton: {
    backgroundColor: '#0056b3',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  pressedText: {
    opacity: 0.8,
  },
});
```

### Advanced Gestures with react-native-gesture-handler
```bash
npm install react-native-gesture-handler
```

```jsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { 
  GestureHandlerRootView,
  PanGestureHandler,
  TapGestureHandler,
  PinchGestureHandler,
  State 
} from 'react-native-gesture-handler';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  useAnimatedGestureHandler,
  withSpring,
} from 'react-native-reanimated';

const GestureExample = () => {
  const translateX = useSharedValue(0);
  const translateY = useSharedValue(0);
  const scale = useSharedValue(1);

  const panGestureHandler = useAnimatedGestureHandler({
    onStart: (_, context) => {
      context.startX = translateX.value;
      context.startY = translateY.value;
    },
    onActive: (event, context) => {
      translateX.value = context.startX + event.translationX;
      translateY.value = context.startY + event.translationY;
    },
    onEnd: () => {
      translateX.value = withSpring(0);
      translateY.value = withSpring(0);
    },
  });

  const pinchGestureHandler = useAnimatedGestureHandler({
    onActive: (event) => {
      scale.value = event.scale;
    },
    onEnd: () => {
      scale.value = withSpring(1);
    },
  });

  const animatedStyle = useAnimatedStyle(() => {
    return {
      transform: [
        { translateX: translateX.value },
        { translateY: translateY.value },
        { scale: scale.value },
      ],
    };
  });

  return (
    <GestureHandlerRootView style={styles.container}>
      <PinchGestureHandler onGestureEvent={pinchGestureHandler}>
        <Animated.View>
          <PanGestureHandler onGestureEvent={panGestureHandler}>
            <Animated.View style={[styles.box, animatedStyle]}>
              <Text style={styles.text}>Drag and Pinch Me!</Text>
            </Animated.View>
          </PanGestureHandler>
        </Animated.View>
      </PinchGestureHandler>
    </GestureHandlerRootView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  box: {
    width: 150,
    height: 150,
    backgroundColor: '#007AFF',
    borderRadius: 12,
    justifyContent: 'center',
    alignItems: 'center',
  },
  text: {
    color: 'white',
    fontWeight: 'bold',
    textAlign: 'center',
  },
});
```

**Explanation:** Use TouchableOpacity for basic interactions, Pressable for more control. react-native-gesture-handler provides advanced gesture recognition with better performance.

---

## Permissions

### Using expo-permissions (Expo)
```bash
npx expo install expo-permissions expo-camera expo-location
```

```jsx
import React, { useState, useEffect } from 'react';
import { View, Text, Alert, TouchableOpacity } from 'react-native';
import * as Permissions from 'expo-permissions';
import * as Location from 'expo-location';
import { Camera } from 'expo-camera';

const PermissionsExample = () => {
  const [cameraPermission, setCameraPermission] = useState(null);
  const [locationPermission, setLocationPermission] = useState(null);
  const [location, setLocation] = useState(null);

  useEffect(() => {
    checkPermissions();
  }, []);

  const checkPermissions = async () => {
    // Check camera permission
    const { status: cameraStatus } = await Camera.requestCameraPermissionsAsync();
    setCameraPermission(cameraStatus === 'granted');

    // Check location permission
    const { status: locationStatus } = await Location.requestForegroundPermissionsAsync();
    setLocationPermission(locationStatus === 'granted');
  };

  const requestCameraPermission = async () => {
    const { status } = await Camera.requestCameraPermissionsAsync();
    setCameraPermission(status === 'granted');
    
    if (status !== 'granted') {
      Alert.alert(
        'Permission Required',
        'Camera permission is required to use this feature.',
        [
          { text: 'Cancel', style: 'cancel' },
          { text: 'Settings', onPress: () => Linking.openSettings() }
        ]
      );
    }
  };

  const getCurrentLocation = async () => {
    try {
      if (!locationPermission) {
        Alert.alert('Permission Required', 'Location permission is required.');
        return;
      }

      const location = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.High,
      });
      
      setLocation(location);
    } catch (error) {
      Alert.alert('Error', 'Failed to get location');
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Permissions Example</Text>
      
      <Text style={styles.status}>
        Camera: {cameraPermission ? '✅ Granted' : '❌ Denied'}
      </Text>
      <Text style={styles.status}>
        Location: {locationPermission ? '✅ Granted' : '❌ Denied'}
      </Text>

      <TouchableOpacity 
        style={styles.button} 
        onPress={requestCameraPermission}
      >
        <Text style={styles.buttonText}>Request Camera Permission</Text>
      </TouchableOpacity>

      <TouchableOpacity 
        style={styles.button} 
        onPress={getCurrentLocation}
      >
        <Text style={styles.buttonText}>Get Location</Text>
      </TouchableOpacity>

      {location && (
        <Text style={styles.locationText}>
          Lat: {location.coords.latitude.toFixed(6)}{'\n'}
          Lng: {location.coords.longitude.toFixed(6)}
        </Text>
      )}
    </View>
  );
};
```

### Using react-native-permissions (Bare React Native)
```bash
npm install react-native-permissions
```

```jsx
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions';
import { Platform } from 'react-native';

const PermissionService = {
  checkCameraPermission: async () => {
    const permission = Platform.OS === 'ios' 
      ? PERMISSIONS.IOS.CAMERA 
      : PERMISSIONS.ANDROID.CAMERA;
    
    const result = await check(permission);
    return result === RESULTS.GRANTED;
  },

  requestCameraPermission: async () => {
    const permission = Platform.OS === 'ios' 
      ? PERMISSIONS.IOS.CAMERA 
      : PERMISSIONS.ANDROID.CAMERA;
    
    const result = await request(permission);
    return result === RESULTS.GRANTED;
  },

  checkLocationPermission: async () => {
    const permission = Platform.OS === 'ios' 
      ? PERMISSIONS.IOS.LOCATION_WHEN_IN_USE 
      : PERMISSIONS.ANDROID.ACCESS_FINE_LOCATION;
    
    const result = await check(permission);
    return result === RESULTS.GRANTED;
  },
};
```

**Explanation:** Always check and request permissions before accessing device features. Provide clear feedback to users and handle permission denials gracefully.

---

## Custom Hooks

### Simple Custom Hook
```jsx
import { useState, useEffect } from 'react';

// Custom hook for API calls
const useApi = (url) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(url);
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
    };

    fetchData();
  }, [url]);

  const refetch = () => {
    fetchData();
  };

  return { data, loading, error, refetch };
};

// Usage
const PostsScreen = () => {
  const { data: posts, loading, error, refetch } = useApi('https://jsonplaceholder.typicode.com/posts');

  if (loading) return <Text>Loading...</Text>;
  if (error) return <Text>Error: {error}</Text>;

  return (
    <FlatList
      data={posts}
      renderItem={({ item }) => <Text>{item.title}</Text>}
      keyExtractor={(item) => item.id.toString()}
      onRefresh={refetch}
      refreshing={loading}
    />
  );
};
```

### Advanced Custom Hook with Local Storage
```jsx
import { useState, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Custom hook for persistent state
const useAsyncStorage = (key, initialValue) => {
  const [storedValue, setStoredValue] = useState(initialValue);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadStoredValue();
  }, [key]);

  const loadStoredValue = async () => {
    try {
      const item = await AsyncStorage.getItem(key);
      if (item !== null) {
        setStoredValue(JSON.parse(item));
      }
    } catch (error) {
      console.error('Error loading stored value:', error);
    } finally {
      setLoading(false);
    }
  };

  const setValue = async (value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      await AsyncStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('Error storing value:', error);
    }
  };

  const removeValue = async () => {
    try {
      await AsyncStorage.removeItem(key);
      setStoredValue(initialValue);
    } catch (error) {
      console.error('Error removing stored value:', error);
    }
  };

  return [storedValue, setValue, removeValue, loading];
};

// Form validation hook
const useFormValidation = (initialState, validationRules) => {
  const [values, setValues] = useState(initialState);
  const [errors, setErrors] = useState({});
  const [isValid, setIsValid] = useState(false);

  useEffect(() => {
    validateForm();
  }, [values]);

  const validateForm = () => {
    const newErrors = {};
    
    Object.keys(validationRules).forEach(field => {
      const rule = validationRules[field];
      const value = values[field];
      
      if (rule.required && !value) {
        newErrors[field] = `${field} is required`;
      } else if (rule.minLength && value.length < rule.minLength) {
        newErrors[field] = `${field} must be at least ${rule.minLength} characters`;
      } else if (rule.pattern && !rule.pattern.test(value)) {
        newErrors[field] = rule.message || `${field} is invalid`;
      }
    });
    
    setErrors(newErrors);
    setIsValid(Object.keys(newErrors).length === 0);
  };

  const setValue = (field, value) => {
    setValues(prev => ({
      ...prev,
      [field]: value
    }));
  };

  const resetForm = () => {
    setValues(initialState);
    setErrors({});
  };

  return {
    values,
    errors,
    isValid,
    setValue,
    resetForm,
  };
};

// Usage example
const LoginScreen = () => {
  const [credentials, setCredentials, removeCredentials] = useAsyncStorage('credentials', {
    email: '',
    rememberMe: false
  });

  const { values, errors, isValid, setValue, resetForm } = useFormValidation(
    { email: credentials.email || '', password: '' },
    {
      email: {
        required: true,
        pattern: /\S+@\S+\.\S+/,
        message: 'Please enter a valid email'
      },
      password: {
        required: true,
        minLength: 6
      }
    }
  );

  // Component implementation...
};
```

**Explanation:** Custom hooks encapsulate reusable logic. They follow the same rules as built-in hooks and can use other hooks internally. Great for API calls, form validation, and persistent state.

---

## AsyncStorage

### Installation and Basic Usage
```bash
npm install @react-native-async-storage/async-storage
```

```jsx
import AsyncStorage from '@react-native-async-storage/async-storage';
import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, TouchableOpacity, Alert } from 'react-native';

const AsyncStorageExample = () => {
  const [name, setName] = useState('');
  const [savedName, setSavedName] = useState('');

  useEffect(() => {
    loadSavedName();
  }, []);

  const loadSavedName = async () => {
    try {
      const value = await AsyncStorage.getItem('userName');
      if (value !== null) {
        setSavedName(value);
        setName(value);
      }
    } catch (error) {
      console.error('Error loading saved name:', error);
    }
  };

  const saveName = async () => {
    try {
      await AsyncStorage.setItem('userName', name);
      setSavedName(name);
      Alert.alert('Success', 'Name saved successfully!');
    } catch (error) {
      console.error('Error saving name:', error);
      Alert.alert('Error', 'Failed to save name');
    }
  };

  const clearName = async () => {
    try {
      await AsyncStorage.removeItem('userName');
      setName('');
      setSavedName('');
      Alert.alert('Success', 'Name cleared successfully!');
    } catch (error) {
      console.error('Error clearing name:', error);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>AsyncStorage Example</Text>
      
      {savedName ? (
        <Text style={styles.welcomeText}>Welcome back, {savedName}!</Text>
      ) : (
        <Text style={styles.welcomeText}>No saved name found</Text>
      )}

      <TextInput
        style={styles.input}
        placeholder="Enter your name"
        value={name}
        onChangeText={setName}
      />

      <TouchableOpacity style={styles.button} onPress={saveName}>
        <Text style={styles.buttonText}>Save Name</Text>
      </TouchableOpacity>

      <TouchableOpacity 
        style={[styles.button, styles.clearButton]} 
        onPress={clearName}
      >
        <Text style={styles.buttonText}>Clear Name</Text>
      </TouchableOpacity>
    </View>
  );
};
```

### Advanced AsyncStorage Operations
```jsx
const StorageService = {
  // Store object
  storeObject: async (key, value) => {
    try {
      const jsonValue = JSON.stringify(value);
      await AsyncStorage.setItem(key, jsonValue);
    } catch (error) {
      console.error('Error storing object:', error);
    }
  },

  // Get object
  getObject: async (key) => {
    try {
      const jsonValue = await AsyncStorage.getItem(key);
      return jsonValue != null ? JSON.parse(jsonValue) : null;
    } catch (error) {
      console.error('Error getting object:', error);
      return null;
    }
  },

  // Store multiple items
  storeMultiple: async (keyValuePairs) => {
    try {
      await AsyncStorage.multiSet(keyValuePairs);
    } catch (error) {
      console.error('Error storing multiple items:', error);
    }
  },

  // Get multiple items
  getMultiple: async (keys) => {
    try {
      const values = await AsyncStorage.multiGet(keys);
      return values.reduce((result, [key, value]) => {
        result[key] = value ? JSON.parse(value) : null;
        return result;
      }, {});
    } catch (error) {
      console.error('Error getting multiple items:', error);
      return {};
    }
  },

  // Get all keys
  getAllKeys: async () => {
    try {
      return await AsyncStorage.getAllKeys();
    } catch (error) {
      console.error('Error getting all keys:', error);
      return [];
    }
  },

  // Clear all data
  clearAll: async () => {
    try {
      await AsyncStorage.clear();
    } catch (error) {
      console.error('Error clearing all data:', error);
    }
  },
};

// Usage in a settings screen
const SettingsScreen = () => {
  const [settings, setSettings] = useState({
    theme: 'light',
    notifications: true,
    language: 'en',
  });

  useEffect(() => {
    loadSettings();
  }, []);

  const loadSettings = async () => {
    const savedSettings = await StorageService.getObject('userSettings');
    if (savedSettings) {
      setSettings(savedSettings);
    }
  };

  const updateSetting = async (key, value) => {
    const newSettings = { ...settings, [key]: value };
    setSettings(newSettings);
    await StorageService.storeObject('userSettings', newSettings);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Settings</Text>
      
      <View style={styles.settingItem}>
        <Text>Theme: {settings.theme}</Text>
        <TouchableOpacity 
          onPress={() => updateSetting('theme', settings.theme === 'light' ? 'dark' : 'light')}
        >
          <Text>Toggle</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.settingItem}>
        <Text>Notifications: {settings.notifications ? 'On' : 'Off'}</Text>
        <TouchableOpacity 
          onPress={() => updateSetting('notifications', !settings.notifications)}
        >
          <Text>Toggle</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};
```

**Explanation:** AsyncStorage provides persistent key-value storage. Always handle operations asynchronously and provide error handling. Use JSON.stringify/parse for complex objects.

---

## Device Features

### Camera Usage
```bash
npx expo install expo-camera expo-media-library
```

```jsx
import React, { useState, useRef, useEffect } from 'react';
import { View, Text, TouchableOpacity, Alert, Image } from 'react-native';
import { Camera } from 'expo-camera';
import * as MediaLibrary from 'expo-media-library';

const CameraScreen = () => {
  const [hasPermission, setHasPermission] = useState(null);
  const [type, setType] = useState(Camera.Constants.Type.back);
  const [photo, setPhoto] = useState(null);
  const cameraRef = useRef(null);

  useEffect(() => {
    (async () => {
      const { status } = await Camera.requestCameraPermissionsAsync();
      setHasPermission(status === 'granted');
    })();
  }, []);

  const takePicture = async () => {
    if (cameraRef.current) {
      try {
        const photo = await cameraRef.current.takePictureAsync({
          quality: 0.7,
          base64: true,
        });
        setPhoto(photo);
      } catch (error) {
        Alert.alert('Error', 'Failed to take picture');
      }
    }
  };

  const savePicture = async () => {
    if (photo) {
      try {
        await MediaLibrary.saveToLibraryAsync(photo.uri);
        Alert.alert('Success', 'Photo saved to gallery!');
      } catch (error) {
        Alert.alert('Error', 'Failed to save photo');
      }
    }
  };

  if (hasPermission === null) {
    return <Text>Requesting camera permission...</Text>;
  }
  
  if (hasPermission === false) {
    return <Text>No access to camera</Text>;
  }

  return (
    <View style={styles.container}>
      {photo ? (
        <View style={styles.previewContainer}>
          <Image source={{ uri: photo.uri }} style={styles.preview} />
          <View style={styles.buttonContainer}>
            <TouchableOpacity style={styles.button} onPress={savePicture}>
              <Text style={styles.buttonText}>Save</Text>
            </TouchableOpacity>
            <TouchableOpacity 
              style={styles.button} 
              onPress={() => setPhoto(null)}
            >
              <Text style={styles.buttonText}>Retake</Text>
            </TouchableOpacity>
          </View>
        </View>
      ) : (
        <Camera style={styles.camera} type={type} ref={cameraRef}>
          <View style={styles.buttonContainer}>
            <TouchableOpacity
              style={styles.button}
              onPress={() => {
                setType(
                  type === Camera.Constants.Type.back
                    ? Camera.Constants.Type.front
                    : Camera.Constants.Type.back
                );
              }}
            >
              <Text style={styles.buttonText}>Flip</Text>
            </TouchableOpacity>
            
            <TouchableOpacity style={styles.captureButton} onPress={takePicture}>
              <View style={styles.captureButtonInner} />
            </TouchableOpacity>
          </View>
        </Camera>
      )}
    </View>
  );
};
```

### Location Services
```bash
npx expo install expo-location
```

```jsx
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, Alert } from 'react-native';
import * as Location from 'expo-location';

const LocationScreen = () => {
  const [location, setLocation] = useState(null);
  const [address, setAddress] = useState(null);
  const [watching, setWatching] = useState(false);

  const getCurrentLocation = async () => {
    try {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        Alert.alert('Permission denied', 'Location permission is required');
        return;
      }

      const location = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.High,
      });
      
      setLocation(location);
      
      // Reverse geocoding
      const [addressResult] = await Location.reverseGeocodeAsync({
        latitude: location.coords.latitude,
        longitude: location.coords.longitude,
      });
      
      setAddress(addressResult);
    } catch (error) {
      Alert.alert('Error', 'Failed to get location');
    }
  };

  const watchLocation = async () => {
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== 'granted') return;

    const subscription = await Location.watchPositionAsync(
      {
        accuracy: Location.Accuracy.High,
        timeInterval: 1000,
        distanceInterval: 1,
      },
      (newLocation) => {
        setLocation(newLocation);
      }
    );

    setWatching(true);
    
    // Stop watching after 30 seconds
    setTimeout(() => {
      subscription.remove();
      setWatching(false);
    }, 30000);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Location Services</Text>
      
      {location && (
        <View style={styles.locationInfo}>
          <Text>Latitude: {location.coords.latitude.toFixed(6)}</Text>
          <Text>Longitude: {location.coords.longitude.toFixed(6)}</Text>
          <Text>Accuracy: {location.coords.accuracy}m</Text>
          <Text>Altitude: {location.coords.altitude}m</Text>
          <Text>Speed: {location.coords.speed}m/s</Text>
        </View>
      )}

      {address && (
        <View style={styles.addressInfo}>
          <Text style={styles.subtitle}>Address:</Text>
          <Text>{address.street} {address.streetNumber}</Text>
          <Text>{address.city}, {address.region}</Text>
          <Text>{address.country} {address.postalCode}</Text>
        </View>
      )}

      <TouchableOpacity style={styles.button} onPress={getCurrentLocation}>
        <Text style={styles.buttonText}>Get Current Location</Text>
      </TouchableOpacity>

      <TouchableOpacity 
        style={[styles.button, watching && styles.activeButton]} 
        onPress={watchLocation}
        disabled={watching}
      >
        <Text style={styles.buttonText}>
          {watching ? 'Watching Location...' : 'Watch Location (30s)'}
        </Text>
      </TouchableOpacity>
    </View>
  );
};
```

### Push Notifications
```bash
npx expo install expo-notifications expo-device expo-constants
```

```jsx
import React, { useState, useEffect, useRef } from 'react';
import { View, Text, TouchableOpacity, Alert, Platform } from 'react-native';
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import Constants from 'expo-constants';

// Configure notification behavior
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: false,
    shouldSetBadge: false,
  }),
});

const NotificationScreen = () => {
  const [expoPushToken, setExpoPushToken] = useState('');
  const [notification, setNotification] = useState(false);
  const notificationListener = useRef();
  const responseListener = useRef();

  useEffect(() => {
    registerForPushNotificationsAsync().then(token => setExpoPushToken(token));

    // Listen for incoming notifications
    notificationListener.current = Notifications.addNotificationReceivedListener(notification => {
      setNotification(notification);
    });

    // Listen for user interactions with notifications
    responseListener.current = Notifications.addNotificationResponseReceivedListener(response => {
      console.log('Notification response:', response);
    });

    return () => {
      Notifications.removeNotificationSubscription(notificationListener.current);
      Notifications.removeNotificationSubscription(responseListener.current);
    };
  }, []);

  const registerForPushNotificationsAsync = async () => {
    let token;

    if (Platform.OS === 'android') {
      await Notifications.setNotificationChannelAsync('default', {
        name: 'default',
        importance: Notifications.AndroidImportance.MAX,
        vibrationPattern: [0, 250, 250, 250],
        lightColor: '#FF231F7C',
      });
    }

    if (Device.isDevice) {
      const { status: existingStatus } = await Notifications.getPermissionsAsync();
      let finalStatus = existingStatus;
      
      if (existingStatus !== 'granted') {
        const { status } = await Notifications.requestPermissionsAsync();
        finalStatus = status;
      }
      
      if (finalStatus !== 'granted') {
        Alert.alert('Failed to get push token for push notification!');
        return;
      }
      
      token = (await Notifications.getExpoPushTokenAsync({
        projectId: Constants.expoConfig.extra.eas.projectId,
      })).data;
    } else {
      Alert.alert('Must use physical device for Push Notifications');
    }

    return token;
  };

  const schedulePushNotification = async () => {
    await Notifications.scheduleNotificationAsync({
      content: {
        title: "You've got mail! 📬",
        body: 'Here is the notification body',
        data: { data: 'goes here' },
      },
      trigger: { seconds: 2 },
    });
  };

  const sendPushNotification = async (expoPushToken) => {
    const message = {
      to: expoPushToken,
      sound: 'default',
      title: 'Original Title',
      body: 'And here is the body!',
      data: { someData: 'goes here' },
    };

    await fetch('https://exp.host/--/api/v2/push/send', {
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Accept-encoding': 'gzip, deflate',
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(message),
    });
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Push Notifications</Text>
      
      <Text style={styles.tokenText}>
        Push Token: {expoPushToken ? expoPushToken.substring(0, 50) + '...' : 'Loading...'}
      </Text>

      {notification && (
        <View style={styles.notificationContainer}>
          <Text style={styles.subtitle}>Last Notification:</Text>
          <Text>Title: {notification.request.content.title}</Text>
          <Text>Body: {notification.request.content.body}</Text>
        </View>
      )}

      <TouchableOpacity 
        style={styles.button} 
        onPress={schedulePushNotification}
      >
        <Text style={styles.buttonText}>Schedule Local Notification</Text>
      </TouchableOpacity>

      <TouchableOpacity 
        style={styles.button} 
        onPress={() => sendPushNotification(expoPushToken)}
      >
        <Text style={styles.buttonText}>Send Push Notification</Text>
      </TouchableOpacity>
    </View>
  );
};
```

**Explanation:** Device features require proper permissions and error handling. Camera provides photo/video capture, Location gives GPS coordinates, and Notifications enable push messaging. Always test on real devices.

---

## Debugging and Testing

### Console Debugging
```jsx
import React, { useEffect } from 'react';
import { View, Text } from 'react-native';

const DebuggingExample = () => {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetchData();
  }, []);

  const fetchData = async () => {
    console.log('🚀 Starting data fetch...');
    console.time('API Call');
    
    try {
      const response = await fetch('https://api.example.com/data');
      console.log('📡 Response status:', response.status);
      
      const result = await response.json();
      console.log('📊 Data received:', result);
      console.table(result.slice(0, 5)); // Table format for arrays
      
      setData(result);
    } catch (error) {
      console.error('❌ Fetch failed:', error);
      console.trace(); // Stack trace
    } finally {
      console.timeEnd('API Call');
    }
  };

  // Performance monitoring
  const handlePress = () => {
    console.time('Button Press Handler');
    
    // Simulate some work
    for (let i = 0; i < 1000000; i++) {
      // Heavy computation
    }
    
    console.timeEnd('Button Press Handler');
  };

  return (
    <View>
      <Text>Debugging Example</Text>
    </View>
  );
};
```

### React DevTools Setup
```bash
# Install React DevTools
npm install -g react-devtools

# Start React DevTools
npx react-devtools
```

### Flipper Integration (React Native CLI)
```jsx
// In your component
import { logger } from 'flipper';

const MyComponent = () => {
  useEffect(() => {
    logger.log('Component mounted');
    logger.warn('This is a warning');
    logger.error('This is an error');
  }, []);

  return <View>...</View>;
};
```

### Unit Testing with Jest
```bash
npm install --save-dev jest @testing-library/react-native
```

```jsx
// __tests__/Calculator.test.js
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import Calculator from '../src/components/Calculator';

describe('Calculator', () => {
  test('renders correctly', () => {
    const { getByText } = render(<Calculator />);
    expect(getByText('Calculator')).toBeTruthy();
  });

  test('adds numbers correctly', () => {
    const { getByText, getByTestId } = render(<Calculator />);
    
    fireEvent.press(getByText('2'));
    fireEvent.press(getByText('+'));
    fireEvent.press(getByText('3'));
    fireEvent.press(getByText('='));
    
    expect(getByTestId('result')).toHaveTextContent('5');
  });

  test('handles division by zero', () => {
    const { getByText, getByTestId } = render(<Calculator />);
    
    fireEvent.press(getByText('5'));
    fireEvent.press(getByText('÷'));
    fireEvent.press(getByText('0'));
    fireEvent.press(getByText('='));
    
    expect(getByTestId('result')).toHaveTextContent('Error');
  });
});
```

### E2E Testing with Detox
```bash
npm install --save-dev detox
```

```js
// e2e/firstTest.e2e.js
describe('App', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should show login screen', async () => {
    await expect(element(by.id('loginScreen'))).toBeVisible();
  });

  it('should login successfully', async () => {
    await element(by.id('emailInput')).typeText('user@example.com');
    await element(by.id('passwordInput')).typeText('password123');
    await element(by.id('loginButton')).tap();
    await expect(element(by.id('homeScreen'))).toBeVisible();
  });
});
```

**Explanation:** Use console methods for basic debugging, React DevTools for component inspection, Jest for unit tests, and Detox for E2E testing. Always add testID props for testing.

---

## Performance Best Practices

### Optimization Techniques
```jsx
import React, { memo, useMemo, useCallback, useState } from 'react';
import { FlatList, View, Text, TouchableOpacity } from 'react-native';

// Memoized component to prevent unnecessary re-renders
const ListItem = memo(({ item, onPress }) => {
  console.log('Rendering item:', item.id); // This should only log when necessary
  
  return (
    <TouchableOpacity onPress={() => onPress(item.id)}>
      <View style={styles.item}>
        <Text>{item.title}</Text>
      </View>
    </TouchableOpacity>
  );
});

const OptimizedList = () => {
  const [items, setItems] = useState([]);
  const [selectedItems, setSelectedItems] = useState(new Set());

  // Memoize expensive calculations
  const filteredItems = useMemo(() => {
    return items.filter(item => item.visible);
  }, [items]);

  const selectedCount = useMemo(() => {
    return selectedItems.size;
  }, [selectedItems]);

  // Memoize callback functions
  const handleItemPress = useCallback((itemId) => {
    setSelectedItems(prev => {
      const newSet = new Set(prev);
      if (newSet.has(itemId)) {
        newSet.delete(itemId);
      } else {
        newSet.add(itemId);
      }
      return newSet;
    });
  }, []);

  // Optimized FlatList configuration
  const renderItem = useCallback(({ item }) => (
    <ListItem item={item} onPress={handleItemPress} />
  ), [handleItemPress]);

  const keyExtractor = useCallback((item) => item.id.toString(), []);

  const getItemLayout = useCallback((data, index) => ({
    length: 60, // Fixed item height
    offset: 60 * index,
    index,
  }), []);

  return (
    <View style={styles.container}>
      <Text>Selected: {selectedCount}</Text>
      <FlatList
        data={filteredItems}
        renderItem={renderItem}
        keyExtractor={keyExtractor}
        getItemLayout={getItemLayout} // For fixed height items
        removeClippedSubviews={true} // Remove off-screen items
        maxToRenderPerBatch={10} // Render 10 items per batch
        updateCellsBatchingPeriod={50} // Update every 50ms
        initialNumToRender={10} // Initial render count
        windowSize={10} // Viewport multiplier
      />
    </View>
  );
};
```

### Image Optimization
```jsx
import React from 'react';
import { Image, View } from 'react-native';
import FastImage from 'react-native-fast-image';

const OptimizedImages = () => {
  return (
    <View>
      {/* Use FastImage for better performance */}
      <FastImage
        style={styles.image}
        source={{
          uri: 'https://example.com/image.jpg',
          priority: FastImage.priority.normal,
        }}
        resizeMode={FastImage.resizeMode.cover}
      />

      {/* For local images, use different resolutions */}
      <Image
        source={require('../assets/image@2x.png')} // Will auto-select @3x on high-res devices
        style={styles.image}
        resizeMode="cover"
      />
    </View>
  );
};
```

### Bundle Size Optimization
```jsx
// Use dynamic imports for code splitting
const LazyComponent = React.lazy(() => import('./HeavyComponent'));

const App = () => {
  return (
    <React.Suspense fallback={<Loading />}>
      <LazyComponent />
    </React.Suspense>
  );
};

// Import only what you need
import { debounce } from 'lodash/debounce'; // ✅ Good
import _ from 'lodash'; // ❌ Imports entire library

// Use tree shaking friendly imports
import { Button } from 'react-native-elements'; // ❌ May import entire library
import Button from 'react-native-elements/src/buttons/Button'; // ✅ Import specific component
```

### Memory Management
```jsx
import React, { useEffect, useRef } from 'react';

const MemoryOptimizedComponent = () => {
  const intervalRef = useRef(null);
  const timeoutRef = useRef(null);
  const subscriptionRef = useRef(null);

  useEffect(() => {
    // Set up subscriptions/timers
    intervalRef.current = setInterval(() => {
      console.log('Interval tick');
    }, 1000);

    timeoutRef.current = setTimeout(() => {
      console.log('Timeout executed');
    }, 5000);

    // API subscription
    subscriptionRef.current = api.subscribe(data => {
      console.log('Data received:', data);
    });

    // Cleanup function - CRITICAL for preventing memory leaks
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
      if (subscriptionRef.current) {
        subscriptionRef.current.unsubscribe();
      }
    };
  }, []);

  return <View>...</View>;
};
```

### Performance Monitoring
```jsx
import { InteractionManager } from 'react-native';

const PerformanceExample = () => {
  useEffect(() => {
    // Wait for interactions to complete before heavy work
    InteractionManager.runAfterInteractions(() => {
      // Heavy computation or API calls
      performExpensiveOperation();
    });
  }, []);

  const performExpensiveOperation = () => {
    console.time('ExpensiveOperation');
    
    // Your expensive code here
    
    console.timeEnd('ExpensiveOperation');
  };

  return <View>...</View>;
};
```

**Explanation:** Optimize performance by memoizing components and calculations, using efficient list rendering, managing memory properly, and monitoring performance. Use tools like Flipper and React DevTools Profiler to identify bottlenecks.

---

## Additional Best Practices

### Project Structure Best Practices
```
src/
├── components/
│   ├── common/          # Reusable UI components
│   ├── forms/           # Form-specific components
│   └── navigation/      # Navigation components
├── screens/             # Screen components
├── hooks/               # Custom hooks
├── services/            # API and external services
├── utils/               # Helper functions
├── constants/           # App constants
├── contexts/            # React contexts
├── types/               # TypeScript types (if using TS)
└── assets/              # Images, fonts, etc.
```

### Code Quality Tools
```bash
# ESLint and Prettier
npm install --save-dev eslint prettier eslint-plugin-react-native

# Husky for git hooks
npm install --save-dev husky lint-staged

# TypeScript (recommended)
npm install --save-dev typescript @types/react @types/react-native
```

### Error Boundaries
```jsx
import React from 'react';
import { View, Text, TouchableOpacity } from 'react-native';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Log to crash reporting service
  }

  render() {
    if (this.state.hasError) {
      return (
        <View style={styles.errorContainer}>
          <Text style={styles.errorTitle}>Something went wrong</Text>
          <Text style={styles.errorMessage}>
            {this.state.error?.message}
          </Text>
          <TouchableOpacity 
            style={styles.retryButton}
            onPress={() => this.setState({ hasError: false, error: null })}
          >
            <Text style={styles.retryText}>Try Again</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return this.props.children;
  }
}

// Usage
const App = () => (
  <ErrorBoundary>
    <YourAppContent />
  </ErrorBoundary>
);
```

### Security Best Practices
```jsx
// Store sensitive data securely
import * as SecureStore from 'expo-secure-store';

const SecurityService = {
  storeToken: async (token) => {
    await SecureStore.setItemAsync('authToken', token);
  },
  
  getToken: async () => {
    return await SecureStore.getItemAsync('authToken');
  },
  
  removeToken: async () => {
    await SecureStore.deleteItemAsync('authToken');
  },
};

// API security
const apiCall = async (endpoint, data) => {
  const token = await SecurityService.getToken();
  
  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`,
      // Add CSRF protection
      'X-Requested-With': 'XMLHttpRequest',
    },
    body: JSON.stringify(data),
  });
  
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  
  return response.json();
};
```

This comprehensive cheatsheet covers all the essential React Native concepts with practical examples and explanations. Each section provides working code that you can copy and adapt for your projects. Remember to always test on both iOS and Android devices, handle errors gracefully, and follow performance best practices for optimal user experience.