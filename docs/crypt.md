# Salad Executor API Documentation

## Base URL
`https://saladexecutor.pythonanywhere.com`

## Endpoints

### 1. Generate Keys
Generate new keys for user accounts.

- **URL:** `/v0/generate`
- **Method:** `POST`
- **Auth Required:** Yes (Admin key required)

#### Request Body
```json
{
  "admin": "AdminSecretKey",
  "length": "weekly|monthly|lifetime",
  "amount": 1
}
```

- `admin`: The secret admin key (required)
- `length`: Duration of the key (default: "lifetime")
  - "weekly": 7 days
  - "monthly": 30 days
  - "lifetime": Very long duration
- `amount`: Number of keys to generate (default: 1)

#### Lua Example
```lua
local function generate_keys(admin_key, length, amount)
    local response = request({
        Url = "https://saladexecutor.pythonanywhere.com/v0/generate",
        Method = "POST",
        Headers = {
            ["Content-Type"] = "application/json"
        },
        Body = game:GetService("HttpService"):JSONEncode({
            admin = admin_key,
            length = length,
            amount = amount
        })
    })
    print(response.Body)
end

-- Usage
generate_keys("AdminSecretKey", "weekly", 5)
```

#### Response
```json
{
  "message": "Keys generated successfully!",
  "keys": "uuid1,uuid2,uuid3"
}
```

#### Status Codes
- 201: Created
- 401: Unauthorized (if admin key is incorrect)

### 2. Register Account
Register a new user account.

- **URL:** `/v0/register`
- **Method:** `POST`
- **Auth Required:** No

#### Request Body
```json
{
  "username": "string",
  "password": "string",
  "email": "string"
}
```

#### Lua Example
```lua
local function register_account(username, password, email)
    local hashed_password = crypt.hash(password, "sha512")
    local response = request({
        Url = "https://saladexecutor.pythonanywhere.com/v0/register",
        Method = "POST",
        Headers = {
            ["Content-Type"] = "application/json"
        },
        Body = game:GetService("HttpService"):JSONEncode({
            username = username,
            password = hashed_password,
            email = email
        })
    })
    print(response.Body)
end

-- Usage
register_account("NewUser", "SecurePassword123", "user@example.com")
```

#### Response
```json
{
  "message": "User registered successfully"
}
```

#### Status Codes
- 201: Created
- 409: Conflict (if username already exists)

### 3. Login
Authenticate a user.

- **URL:** `/v0/login`
- **Method:** `POST`
- **Auth Required:** No

#### Request Body
```json
{
  "username": "string",
  "password": "string"
}
```

#### Lua Example
```lua
local function login(username, password)
    local hashed_password = crypt.hash(password, "sha512")
    local response = request({
        Url = "https://saladexecutor.pythonanywhere.com/v0/login",
        Method = "POST",
        Headers = {
            ["Content-Type"] = "application/json"
        },
        Body = game:GetService("HttpService"):JSONEncode({
            username = username,
            password = hashed_password
        })
    })
    print(response.Body)
end

-- Usage
login("ExistingUser", "UserPassword123")
```

#### Response
```json
{
  "message": "Successfully logged in"
}
```

#### Status Codes
- 200: OK
- 401: Unauthorized (missing credentials)
- 403: Forbidden (incorrect credentials)

### 4. Redeem Key
Redeem a key to extend account expiry.

- **URL:** `/v0/redeem`
- **Method:** `POST`
- **Auth Required:** No

#### Request Body
```json
{
  "username": "string",
  "key": "uuid-string"
}
```

#### Lua Example
```lua
local function redeem_key(username, key)
    local response = request({
        Url = "https://saladexecutor.pythonanywhere.com/v0/redeem",
        Method = "POST",
        Headers = {
            ["Content-Type"] = "application/json"
        },
        Body = game:GetService("HttpService"):JSONEncode({
            username = username,
            key = key
        })
    })
    print(response.Body)
end

-- Usage
redeem_key("ExistingUser", "uuid-key-string")
```

#### Response
```json
{
  "message": "Key redeemed successfully"
}
```

#### Status Codes
- 200: OK
- 401: Unauthorized (invalid key)
- 422: Unprocessable Entity (missing key)

### 5. Access Script Hub
Fetch scripts from the script hub.

- **URL:** `/access/scripthub`
- **Method:** `GET`
- **Auth Required:** Yes

#### Query Parameters
- `username`: User's username
- `password`: User's password
- `q`: Search query (default: "script")

#### Lua Example
```lua
local function access_scripthub(username, password, query)
    local hashed_password = crypt.hash(password, "sha512")
    local response = request({
        Url = string.format(
            "https://saladexecutor.pythonanywhere.com/access/scripthub?username=%s&password=%s&q=%s",
            game:GetService("HttpService"):UrlEncode(username),
            game:GetService("HttpService"):UrlEncode(hashed_password),
            game:GetService("HttpService"):UrlEncode(query)
        ),
        Method = "GET"
    })
    print(response.Body)
end

-- Usage
access_scripthub("ExistingUser", "UserPassword123", "aimbot")
```

#### Response
Returns JSON data from scriptblox.com API.

#### Status Codes
- 200: OK
- 401: Unauthorized (missing credentials)
- 403: Forbidden (incorrect credentials)

### 6. Initialize
Fetch initialization data.

- **URL:** `/access/init`
- **Method:** `GET`
- **Auth Required:** Yes

#### Query Parameters
- `username`: User's username
- `password`: User's password

#### Lua Example
```lua
local function initialize(username, password)
    local hashed_password = crypt.hash(password, "sha512")
    local response = request({
        Url = string.format(
            "https://saladexecutor.pythonanywhere.com/access/init?username=%s&password=%s",
            game:GetService("HttpService"):UrlEncode(username),
            game:GetService("HttpService"):UrlEncode(hashed_password)
        ),
        Method = "GET"
    })
    loadstring(response.Body, "init")()
end

-- Usage
initialize("ExistingUser", "UserPassword123")
```

#### Response
Returns plain text content from a GitHub repository.

#### Status Codes
- 200: OK
- 401: Unauthorized (missing credentials)
- 403: Forbidden (incorrect credentials)

## Notes
- All endpoints return JSON responses unless otherwise specified.
- The `/v0/generate` endpoint requires a secret admin key for authentication. This key is not disclosed in the documentation for security reasons.
- User sessions are logged for login, key redemption, and access to scripthub and init endpoints.
- The API uses UUID for key generation.
- Account expiry times are managed in seconds since epoch.
- In the Lua examples, we use `game:GetService("HttpService"):JSONEncode()` to convert Lua tables to JSON strings, and `game:GetService("HttpService"):UrlEncode()` to properly encode URL parameters.
- Passwords are hashed using SHA-512 via `crypt.hash(password, "sha512")` before being sent to the server. This enhances security by ensuring that plain text passwords are never transmitted.
- Error handling is not included in these examples for brevity. In a production environment, you should add proper error handling and response parsing.
