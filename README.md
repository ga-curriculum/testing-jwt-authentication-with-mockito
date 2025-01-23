# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png) Testing Django Authentication with Mockito

| Title                                      | Type   | Duration | Author               |
|--------------------------------------------|--------|----------|----------------------|
| Testing Django Authentication with Mockito | Lesson | 3:30     | Suresh Melvin Sigera |

### Setting Up the Python Environment (10 minutes)

Before diving into the authentication testing implementation, we need to properly configure our development environment.
In Django development, isolating project dependencies is crucial for maintaining consistent test environments. We'll
start by activating our virtual environment, which creates an isolated Python environment for our project.

Let's begin with the environment setup:

```text
# On Unix/MacOS
source venv/bin/activate

# On Windows
.\venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Understanding Mockito and Unit Testing in Django (30 minutes)

In Django applications, testing authentication and user management systems is crucial but can be complex due to external dependencies and security considerations. Before diving into our implementation, let's understand the importance of Mockito and how it enhances our testing capabilities.

### Understanding Mockito

Mockito is a popular mocking framework that revolutionizes how we approach unit testing. At its core, Mockito allows us to create substitute objects (mocks) that simulate the behavior of real objects in a controlled way. This is particularly valuable when testing authentication systems because:

- Database Independence: Instead of relying on actual database connections and user records, we can mock these interactions, making our tests faster and more reliable.
- Security Testing: We can simulate various security scenarios (valid tokens, invalid tokens, expired sessions) without compromising real security measures.
- Dependency Isolation: Authentication often involves multiple interconnected services (token generation, user verification, permission checking). Mockito lets us test these components in isolation.
- Controlled Testing Environment: We can precisely control the behavior of external services and dependencies, making our tests deterministic and reproducible.

### The Role of `main_app/tests/mock/test_auth.py`

In our Django application, we'll be creating our authentication tests in `main_app/tests/mock/test_auth.py`. This file location follows Django's best practices for organizing test files:

- `main_app/ -` Our main application directory
- `tests/` - Segregates test files from application code
- `mock/` - Specifically for tests using mock objects
- `test_auth.py` - Focuses on authentication-related tests

This organization helps maintain a clear separation between different types of tests and makes it easier for other developers to locate and understand our testing strategy.

### Setting Up Our Test Environment

Before writing our tests, we need to ensure our testing environment is properly configured. Here's what we'll need:

```python
from django.urls import reverse
from rest_framework import status
from rest_framework.test import APITestCase
from django.contrib.auth.models import User
from unittest.mock import patch, MagicMock
```

These imports provide:

- `reverse`: For generating URLs for our API endpoints
- `status`: For HTTP status codes
- `APITestCase`: Django REST framework's test case class
- `User`: Django's built-in user model
- `patch` and `MagicMock`: Core components of Mockito for creating and managing mocks

### Testing Strategy

Our authentication testing strategy will cover several critical areas:

- Token Verification: Ensuring our system correctly validates authentication tokens
- User Registration: Testing the user creation process
- Login Authentication: Verifying credential validation
- Error Handling: Testing system responses to invalid inputs
- Security Measures: Ensuring proper protection against unauthorized access

This comprehensive approach helps ensure our authentication system remains secure and reliable while being maintainable and testable.

### Best Practices for Authentication Testing

When working with Mockito for authentication testing, we follow these best practices:

1. Isolated Tests: Each test should focus on a single aspect of authentication
2. Consistent Mocking: Use consistent mock objects across related tests
3. Security-First: Always include tests for security edge cases
4. Comprehensive Coverage: Test both success and failure scenarios

By following these practices, we create a robust test suite that helps maintain the security and reliability of our authentication system.


### File Structure and Organization

We'll organize our `test_auth.py` file into logical sections:

```python
class AuthTestCase(APITestCase):
    # Setup methods
    def setUp(self):
        # Initialize mock objects and common test data
        pass
        
    # Token verification tests
    def test_user_verification_success(self):
        # Test successful token verification
        pass
        
    def test_user_verification_no_token(self):
        # Test missing token scenarios
        pass
        
    # Registration tests
    def test_user_registration(self):
        # Test user registration process
        pass
        
    # Login tests
    def test_user_login(self):
        # Test login functionality
        pass
```


This organization helps maintain clarity and makes it easier to add new tests as our authentication system evolves.

### Testing Goals

Our authentication tests aim to verify:

1. User Management:
   - Successful user registration
   - Proper handling of duplicate registrations
   - User data validation
2. Authentication:
   - Login with valid credentials
   - Rejection of invalid credentials
   - Token generation and validation
3. Security:
   - Protection against unauthorized access
   - Proper handling of invalid tokens

By thoroughly testing these aspects, we ensure our authentication system remains robust and secure.


### Setting Up the Test Environment (15 minutes)

The `setUp` method is crucial as it prepares our test environment before each test case:

```python
class AuthTestCase(APITestCase):
    def setUp(self):
        self.mock_user = MagicMock(spec=User)
        self.mock_user.configure_mock(**{
            'id': 1,
            'username': 'testuser',
            'email': 'test@test.com',
            'is_authenticated': True,
            'is_active': True,
            'is_anonymous': False,
            '__str__': lambda self: 'testuser'
        })

        self.verification_url = reverse('token_refresh')
        self.mock_access_token = "mock_access_token_12345"
        self.mock_refresh_token = "mock_refresh_token_67890"
```

This `setUp` method initializes our test environment before each test case. We create a mock `User` object with
predefined attributes and behaviors using `MagicMock`. This ensures each test starts with a clean, consistent state. The
mock user has properties like `id`, `username`, and `email`, which are essential for testing authentication flows. We
also set up verification URLs and mock tokens that will be used across different test cases.

### Test User Verification Success (20 minutes)

The success verification test ensures that authenticated users can successfully validate their tokens:

```python
@patch('django.contrib.auth.models.User.objects')
@patch('rest_framework_simplejwt.tokens.RefreshToken.for_user')
@patch('rest_framework_simplejwt.authentication.JWTAuthentication.get_validated_token')
@patch('rest_framework_simplejwt.authentication.JWTAuthentication.get_user')
def test_user_verification_success(self, mock_get_user, mock_get_validated_token,
                                   mock_token_for_user, mock_user_objects):
    """Test user verification with a mocked valid access token."""

    mock_get_validated_token.return_value = MagicMock()
    mock_get_user.return_value = self.mock_user
    mock_user_objects.get.return_value = self.mock_user

    mock_token = MagicMock()
    mock_token.access_token = self.mock_access_token
    mock_token.__str__.return_value = self.mock_refresh_token
    mock_token_for_user.return_value = mock_token

    self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {self.mock_access_token}')
    response = self.client.get(self.verification_url)

    self.assertEqual(response.status_code, status.HTTP_200_OK)
    self.assertIn('access', response.data)
    self.assertIn('refresh', response.data)
    self.assertIn('user', response.data)
    self.assertEqual(response.data['user']['username'], 'testuser')
```

This test method uses multiple patch decorators to mock various components of the authentication system. We mock the
user objects, token generation, and JWT authentication to simulate a successful token validation scenario. The test
verifies that when a valid token is provided, the system responds with the expected user data and new tokens.

### Test User Verification No Token (10 minutes)

When building authentication systems, it's crucial to handle cases where users attempt to access protected resources
without providing any authentication token. Here's how we test this scenario:

```python
def test_user_verification_no_token(self):
    """Test user verification without authentication."""
    response = self.client.get(self.verification_url)
    self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
```

In this test, we simulate a scenario where a client attempts to access the token refresh endpoint without providing any
authentication credentials. This is a common security concern that needs to be properly handled. The test makes a
request without setting any authorization headers, which should trigger the authentication middleware to reject the
request. We verify that the system responds with a 401 Unauthorized status code, indicating that authentication is
required to access the resource.

### Test User Verification Invalid Token (15 minutes)

Testing invalid token scenarios helps ensure the system's security integrity:

```python
def test_user_verification_invalid_token(self):
    """Test user verification with invalid token."""
    self.client.credentials(HTTP_AUTHORIZATION='Bearer invalid_token')
    response = self.client.get(self.verification_url)
    self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
```

This test simulates attempts to access protected resources using invalid tokens. We explicitly set an invalid Bearer
token in the request headers and verify that the system properly rejects it. This test is crucial for ensuring the
system can distinguish between valid and invalid tokens and responds appropriately.

### Test User Registration (30 minutes)

The registration test verifies that new users can successfully create accounts:

```python
@patch('main_app.views.User.objects.create_user')
@patch('rest_framework_simplejwt.tokens.RefreshToken.for_user')
@patch('main_app.views.User.objects.get')
def test_user_registration(self, mock_user_get, mock_token_for_user, mock_create_user):
    """Test user registration with mocked user creation."""
    url = reverse('register')
    data = {
        'username': 'newuser',
        'email': 'newuser@test.com',
        'password': 'newpassword123'
    }

    new_mock_user = MagicMock(spec=User)
    new_mock_user.configure_mock(**{
        'id': 2,
        'username': data['username'],
        'email': data['email'],
        'is_authenticated': True,
        'is_active': True,
        'is_anonymous': False,
        '__str__': lambda self: data['username']
    })

    mock_create_user.return_value = new_mock_user
    mock_user_get.return_value = new_mock_user

    mock_token = MagicMock()
    mock_token.access_token = self.mock_access_token
    mock_token.__str__.return_value = self.mock_refresh_token
    mock_token_for_user.return_value = mock_token

    response = self.client.post(url, data)

    self.assertEqual(response.status_code, status.HTTP_200_OK)
    self.assertIn('access', response.data)
    self.assertIn('refresh', response.data)
    self.assertIn('user', response.data)
    self.assertEqual(response.data['user']['username'], 'newuser') 
```

This test mocks the user creation process and verifies that the registration endpoint correctly handles new user
registration requests. It checks that the system creates a new user with the provided credentials and returns
appropriate authentication tokens.

### Test User Login Success (20 minutes)

Testing successful login scenarios ensures users can authenticate properly:

```python
@patch('main_app.views.authenticate')
@patch('rest_framework_simplejwt.tokens.RefreshToken.for_user')
def test_user_login(self, mock_token_for_user, mock_authenticate):
    """Test user login with mocked authentication."""
    url = reverse('login')
    data = {
        'username': 'testuser',
        'password': 'securepassword123'
    }

    mock_authenticate.return_value = self.mock_user

    mock_token = MagicMock()
    mock_token.access_token = self.mock_access_token
    mock_token.__str__.return_value = self.mock_refresh_token
    mock_token_for_user.return_value = mock_token

    response = self.client.post(url, data)

    self.assertEqual(response.status_code, status.HTTP_200_OK)
    self.assertIn('access', response.data)
    self.assertIn('refresh', response.data)
    self.assertIn('user', response.data)
    self.assertEqual(response.data['user']['username'], 'testuser')
```

This test verifies that users can successfully log in with valid credentials. It mocks the authentication process and
ensures the system generates and returns the appropriate tokens upon successful authentication.

### Test User Login Invalid Credentials (15 minutes)

Testing failed login attempts is crucial for security:

```python
@patch('main_app.views.authenticate')
def test_user_login_invalid_credentials(self, mock_authenticate):
    """Test user login with mocked failed authentication."""
    url = reverse('login')
    data = {
        'username': 'testuser',
        'password': 'wrongpassword'
    }

    mock_authenticate.return_value = None

    response = self.client.post(url, data)

    self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
    self.assertIn('error', response.data)
    self.assertEqual(response.data['error'], 'Invalid Credentials')
```

This final test ensures the system properly handles invalid login attempts. It verifies that when incorrect credentials
are provided, the system responds with an appropriate error message and unauthorized status code.

### Running the Tests (5 minutes)

To execute our test suite:

```text
python manage.py test main_app.tests -v 2
```

This command runs our authentication tests with verbose output, providing detailed information about each test's
execution and results. The `-v 2` flag enables verbose output, which is valuable for debugging and understanding test
behavior.

### Summary (10 minutes)

This comprehensive test suite demonstrates best practices for testing Django authentication systems. By using Mockito
and Django's testing framework, we've created isolated tests that verify various authentication scenarios without
depending on external services. These tests cover the full spectrum of authentication flows, from registration through
login and token verification, ensuring our system remains secure and reliable.
Remember to run these tests as part of your continuous integration pipeline to catch potential authentication issues
early in the development cycle. The thorough testing approach demonstrated here provides confidence when making changes
to the authentication system and helps maintain the security of your Django application.
