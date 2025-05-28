
## Testing with Jest

```javascript
// setup-tests.js
import { grab } from 'grab-api.js';

// Setup global mocks for testing
beforeEach(() => {
  // Clear previous mocks
  grab.mock = {};
  grab.log = [];
});

// user.test.js
import { grab } from 'grab-api.js';

describe('User API', () => {
  test('should fetch user data', async () => {
    // Setup mock
    grab.mock.users = {
      response: { id: 1, name: 'Test User', email: 'test@example.com' }
    };

    const result = await grab('users');
    
    expect(result.id).toBe(1);
    expect(result.name).toBe('Test User');
  });

  test('should handle user creation', async () => {
    grab.mock.users = {
      response: (params) => ({
        id: 123,
        ...params,
        created: true
      }),
      method: 'POST'
    };

    const newUser = await grab('users', {
      method: 'POST',
      name: 'John Doe',
      email: 'john@example.com'
    });

    expect(newUser.name).toBe('John Doe');
    expect(newUser.created).toBe(true);
  });

  test('should handle errors', async () => {
    grab.mock.users = {
      response: () => {
        throw new Error('User not found');
      }
    };

    const result = await grab('users');
    expect(result.error).toBe('User not found');
  });
});
```


## Mock Server for Testing

### Basic Mock Setup

```javascript
// Setup mock responses for testing
grab.mock.users = {
  response: [
    { id: 1, name: 'John Doe', email: 'john@example.com' },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
  ],
  delay: 1 // 1 second delay to simulate network
};

grab.mock['products/search'] = {
  response: (params) => ({
    results: [
      { id: 1, name: `Product matching "${params.query}"`, price: 29.99 },
      { id: 2, name: `Another product for "${params.query}"`, price: 19.99 }
    ],
    total: 2
  }),
  method: 'POST',
  delay: 0.5
};

// Now your API calls will use mock data
const users = await grab('users'); // Returns mock user data
```

### Dynamic Mock Responses

```javascript
// Mock with conditional responses
grab.mock['auth/login'] = {
  response: (params) => {
    if (params.email === 'admin@example.com' && params.password === 'admin123') {
      return {
        success: true,
        token: 'mock-jwt-token-12345',
        user: { id: 1, name: 'Admin User', role: 'admin' }
      };
    } else {
      return {
        success: false,
        error: 'Invalid credentials'
      };
    }
  },
  method: 'POST',
  delay: 1
};

// Error simulation
grab.mock['users/create'] = {
  response: (params) => {
    if (!params.email) {
      throw new Error('Email is required');
    }
    return { id: Date.now(), ...params, created: true };
  },
  method: 'POST'
};
```
