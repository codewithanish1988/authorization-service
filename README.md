
# Spring Boot OAuth2 Authorization Server

This project is a **Spring Boot-based OAuth2 Authorization Server** that implements the **Authorization Code** and **Refresh Token** grant flows.  
It allows client applications (such as web apps or mobile apps) to securely authenticate users and obtain access tokens and refresh tokens for API access.

---

## Features

- **OAuth 2.0 Authorization Code flow**
- **Refresh token flow**
- Configurable client credentials
- Secure password storage using BCrypt
- User authentication with Spring Security
- Configurable token expiration

---

## Architecture

The Authorization Server exposes the following key endpoints:

| Endpoint                     | Purpose                                               |
|--------------------------------|-------------------------------------------------------|
| `/oauth2/authorize`          | Starts the authorization code flow (login page)      |
| `/oauth2/token`              | Exchanges authorization code for tokens              |
| `/.well-known/jwks.json`     | Public keys for JWT verification                     |
| `/login/oauth2/code/{client}`| Redirect URI where the auth code is returned        |

---

## How It Works

### **Authorization Code Flow**

1. **Client App** redirects user to `/oauth2/authorize`:
    ```
    GET http://localhost:8080/oauth2/authorize?response_type=code&client_id=api-client-id&scope=openid%20read%20write&redirect_uri=http://localhost:5000/oauth2/callback
    ```

2. User logs in with username/password on Spring Boot login page.

3. Authorization Server redirects to:
    ```
    http://localhost:5000/oauth2/callback?code=AUTH_CODE
    ```

4. Client app sends a POST request to `/oauth2/token`:
    ```bash
    curl --location 'http://localhost:8080/oauth2/token'       --header 'Content-Type: application/x-www-form-urlencoded'       --header 'Authorization: Basic YXBpLWNsaWVudC1pZDphcGkxMjM='       --data-urlencode 'grant_type=authorization_code'       --data-urlencode 'code=AUTH_CODE'       --data-urlencode 'redirect_uri=http://localhost:5000/oauth2/callback'
    ```

5. Server responds with:
    ```json
    {
      "access_token": "...",
      "refresh_token": "...",
      "token_type": "Bearer",
      "expires_in": 3600
    }
    ```

---

### **Refresh Token Flow**

To refresh an access token:
```bash
curl --location 'http://localhost:8080/oauth2/token'   --header 'Content-Type: application/x-www-form-urlencoded'   --header 'Authorization: Basic YXBpLWNsaWVudC1pZDphcGkxMjM='   --data-urlencode 'grant_type=refresh_token'   --data-urlencode 'refresh_token=REFRESH_TOKEN'
```

---

## Security

- **Passwords** are stored using BCrypt encryption.
- **Client secrets** must be sent via Basic Authentication (or configured otherwise).
- Tokens are signed with a private key and exposed via `/jwks.json`.

---

## Running the Server

1. Build the project:
    ```bash
    ./gradlew build
    ```

2. Run the server:
    ```bash
    ./gradlew bootRun
    ```

The server will start on `http://localhost:8080`.

---

## Example Client Flow 

For a web app:

1. Redirect user to:
    ```
    http://localhost:8080/oauth2/authorize?response_type=code&client_id=api-client-id&scope=openid%20read%20write&redirect_uri=http://localhost:5000/oauth2/callback
    ```

2. Handle redirect in Flutter at:
    ```
    http://localhost:5000/oauth2/callback?code=AUTH_CODE
    ```

3. Exchange code for tokens via `/oauth2/token`.

---

## References

- [Spring Authorization Server Docs](https://docs.spring.io/spring-authorization-server/docs/current/reference/html/)
- [OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)
- [BCrypt Password Encoding](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#pe-bcrypt)

