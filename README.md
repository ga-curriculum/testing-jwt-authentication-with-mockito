# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png) Testing Django Authentication with Mockito

| Title                                      | Type   | Duration | Author               |
|--------------------------------------------|--------|----------|----------------------|
| Testing Django Authentication with Mockito | Lesson | 3:00     | Suresh Melvin Sigera |

### Understanding Mockito and Unit Testing in Django
In Django applications, testing authentication and user management systems is crucial but can be complex due to external dependencies and security considerations. This lesson focuses on using Mockito, a powerful mocking framework, to create isolated unit tests for authentication-related functionality.
Let's create comprehensive tests in the `main_app/tests/mock/test_auth.py` file to ensure our authentication system works as expected.

### The Power of Mockito in Testing
Mockito is a mocking framework that allows us to create mock objects for our tests. When testing authentication systems, we often need to isolate our tests from external dependencies like databases or third-party services. Mockito helps us achieve this by:

1. Creating mock objects that simulate real objects' behavior
2. Controlling the behavior of these mock objects
3. Verifying that mock objects are used correctly in our tests

Let's dive into implementing our authentication tests:

First, we'll need our necessary imports:

```python
from django.urls import reverse
from rest_framework import status
from rest_framework.test import APITestCase
from django.contrib.auth.models import User
from unittest.mock import patch, MagicMock
```

### Setting Up the Test Environment

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

This `setUp` method initializes our test environment before each test case. We create a mock `User` object with predefined attributes and behaviors using `MagicMock`. This ensures each test starts with a clean, consistent state. The mock user has properties like `id`, `username`, and `email`, which are essential for testing authentication flows. We also set up verification URLs and mock tokens that will be used across different test cases.

### Test User Verification Success

The success verification test ensures that authenticated users can successfully validate their tokens:

```python
    @patch('django.contrib.auth.models.User.objects')
    @patch('rest_framework_simplejwt.tokens.RefreshToken.for_user')
    @patch('rest_framework_simplejwt.authentication.JWTAuthentication.get_validated_token')
    @patch('rest_framework_simplejwt.authentication.JWTAuthentication.get_user')
    def test_user_verification_success(self, mock_get_user, mock_get_validated_token,
                                       mock_token_for_user, mock_user_objects):
        """Test user verification with a mocked valid access token."""

        # Set up the mocks
        mock_get_validated_token.return_value = MagicMock()
        mock_get_user.return_value = self.mock_user
        mock_user_objects.get.return_value = self.mock_user

        # Mock token generation
        mock_token = MagicMock()
        mock_token.access_token = self.mock_access_token
        mock_token.__str__.return_value = self.mock_refresh_token
        mock_token_for_user.return_value = mock_token

        # Set authorization header
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {self.mock_access_token}')

        # Make the request
        response = self.client.get(self.verification_url)

        # Assertions
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertIn('access', response.data)
        self.assertIn('refresh', response.data)
        self.assertIn('user', response.data)
        self.assertEqual(response.data['user']['username'], 'testuser')

        # Verify our mocks were called correctly
        mock_get_validated_token.assert_called_once()
        mock_get_user.assert_called_once()
        mock_user_objects.get.assert_called()
```

This test method uses multiple patch decorators to mock various components of the authentication system. We mock the user objects, token generation, and JWT authentication to simulate a successful token validation scenario. The test verifies that when a valid token is provided, the system responds with the expected user data and new tokens.

### Test User Verification No Token

When building authentication systems, it's crucial to handle cases where users attempt to access protected resources without providing any authentication token. Here's how we test this scenario:

```python
    def test_user_verification_no_token(self):
        """Test user verification without authentication."""
        response = self.client.get(self.verification_url)
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
```

In this test, we simulate a scenario where a client attempts to access the token refresh endpoint without providing any authentication credentials. This is a common security concern that needs to be properly handled. The test makes a request without setting any authorization headers, which should trigger the authentication middleware to reject the request. We verify that the system responds with a 401 Unauthorized status code, indicating that authentication is required to access the resource.

### Test User Verification Invalid Token

Testing invalid token scenarios helps ensure the system's security integrity:

```python
    def test_user_verification_invalid_token(self):
        """Test user verification with invalid token."""
        self.client.credentials(HTTP_AUTHORIZATION='Bearer invalid_token')
        response = self.client.get(self.verification_url)
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
```

This test simulates attempts to access protected resources using invalid tokens. We explicitly set an invalid Bearer token in the request headers and verify that the system properly rejects it. This test is crucial for ensuring the system can distinguish between valid and invalid tokens and responds appropriately.

### Test User Registration

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

        mock_create_user.assert_called_once_with(
            username=data['username'],
            email=data['email'],
            password=data['password']
        )
```

This test mocks the user creation process and verifies that the registration endpoint correctly handles new user registration requests. It checks that the system creates a new user with the provided credentials and returns appropriate authentication tokens.

### Test User Login Success

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

        mock_authenticate.assert_called_once_with(
            username=data['username'],
            password=data['password']
        )
```

This test verifies that users can successfully log in with valid credentials. It mocks the authentication process and ensures the system generates and returns the appropriate tokens upon successful authentication.

### Test User Login Invalid Credentials

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

        mock_authenticate.assert_called_once_with(
            username=data['username'],
            password=data['password']
        )
```

This final test ensures the system properly handles invalid login attempts. It verifies that when incorrect credentials are provided, the system responds with an appropriate error message and unauthorized status code.

### Summary

This comprehensive test suite demonstrates best practices for testing Django authentication systems. By using Mockito and Django's testing framework, we've created isolated tests that verify various authentication scenarios without depending on external services. These tests cover the full spectrum of authentication flows, from registration through login and token verification, ensuring our system remains secure and reliable.
Remember to run these tests as part of your continuous integration pipeline to catch potential authentication issues early in the development cycle. The thorough testing approach demonstrated here provides confidence when making changes to the authentication system and helps maintain the security of your Django application.