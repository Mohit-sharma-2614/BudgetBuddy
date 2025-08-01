# BudgetBuddy Backend: Detailed Plan

## 🚀 Overview
The **BudgetBuddy** backend, built with **Spring Boot**, powers the personal finance manager mobile app by handling user authentication, expense tracking, budget management, currency conversion, and financial insights. It uses **MySQL** for structured data storage, **Spring Security** with JWT for secure access, and integrates an external currency conversion API (e.g., ExchangeRate-API) for multi-currency support. This plan reuses your **CampusSyncBackend** expertise (JWT, REST APIs) and introduces new skills like handling financial data and external API integration.

## 🛠️ Architecture
### Components
- **Framework**: Spring Boot 3.x for rapid development and REST API creation.
- **Database**: MySQL for structured storage of users, expenses, and budgets.
- **Security**: Spring Security with JWT for authentication and authorization.
- **Networking**: RestTemplate or WebClient for integrating with a currency conversion API.
- **Validation**: Hibernate Validator for input validation (e.g., expense amounts, budget limits).
- **Error Handling**: Custom exceptions and global exception handling for consistent API responses.
- **Logging**: SLF4J with Logback for debugging and monitoring.
- **Testing**: JUnit and MockMVC for unit and integration tests.

### Data Flow
1. **Client Requests**: Mobile app sends HTTP requests (e.g., POST `/api/expenses`) with JWT in the Authorization header.
2. **Authentication**: Spring Security validates JWT and authorizes access based on user roles.
3. **Business Logic**: Controllers delegate to services, which interact with repositories (JPA) and external APIs (e.g., currency conversion).
4. **Response**: Backend returns JSON responses (e.g., expense data, budget status) to the mobile app.
5. **Storage**: MySQL stores user data, expenses, and budgets, with transactions ensuring data integrity.

## 📊 Database Schema
The MySQL database is designed for efficiency and scalability, with the following tables:

- **users**:
  - `id`: BIGINT, Primary Key, Auto-increment
  - `email`: VARCHAR(255), Unique, Not Null
  - `password`: VARCHAR(255), Not Null (BCrypt hashed)
  - `preferred_currency`: VARCHAR(3), Not Null (e.g., USD, EUR)
  - `created_at`: DATETIME, Not Null

- **expenses**:
  - `id`: BIGINT, Primary Key, Auto-increment
  - `user_id`: BIGINT, Foreign Key (references `users.id`)
  - `amount`: DECIMAL(10,2), Not Null
  - `currency`: VARCHAR(3), Not Null (e.g., USD)
  - `category`: VARCHAR(50), Not Null (e.g., groceries, transport)
  - `merchant`: VARCHAR(255), Nullable
  - `date`: DATE, Not Null
  - `created_at`: DATETIME, Not Null

- **budgets**:
  - `id`: BIGINT, Primary Key, Auto-increment
  - `user_id`: BIGINT, Foreign Key (references `users.id`)
  - `category`: VARCHAR(50), Not Null
  - `amount`: DECIMAL(10,2), Not Null
  - `month`: VARCHAR(7), Not Null (e.g., 2025-07)
  - `created_at`: DATETIME, Not Null

**Notes**:
- Use DECIMAL for financial data to avoid floating-point precision issues.
- Indexes on `user_id` and `category` for faster queries.
- `preferred_currency` in `users` supports multi-currency display.

## 🔒 Security
- **JWT Authentication**: Reuse your Campus Sync JWT setup.
  - Generate JWT on login (`/api/auth/login`).
  - Validate JWT for all protected endpoints using a `JwtAuthenticationFilter`.
  - Store JWT in Shared Preferences on the client (as in Campus Sync).
- **Role-Based Access**: Currently, only “USER” role is needed, but the schema supports future roles (e.g., ADMIN for analytics).
- **Password Hashing**: Use BCrypt for secure password storage.
- **HTTPS**: Configure Spring Boot to enforce HTTPS in production.

## 🌐 API Endpoints
Below are the key REST endpoints, designed to support the mobile app’s features.

### Authentication
- **POST /api/auth/register**
  - Request: `{ "email": "user@example.com", "password": "pass123", "preferredCurrency": "USD" }`
  - Response: `{ "message": "User registered successfully" }`
  - Description: Registers a new user, hashes password, stores in MySQL.
- **POST /api/auth/login**
  - Request: `{ "email": "user@example.com", "password": "pass123" }`
  - Response: `{ "token": "jwt-token", "userId": 1, "preferredCurrency": "USD" }`
  - Description: Authenticates user, returns JWT.

### Expenses
- **POST /api/expenses**
  - Request: `{ "amount": 25.50, "currency": "USD", "category": "groceries", "merchant": "Walmart", "date": "2025-07-17" }`
  - Response: `{ "id": 1, "amount": 25.50, "currency": "USD", ... }`
  - Description: Logs an expense (from receipt scan or manual entry), converts amount if needed.
- **GET /api/expenses**
  - Query Params: `?startDate=2025-07-01&endDate=2025-07-31&category=groceries`
  - Response: `[{ "id": 1, "amount": 25.50, "currency": "USD", ... }, ...]`
  - Description: Fetches user’s expenses, filtered by date or category.
- **GET /api/expenses/categories**
  - Response: `{ "groceries": 200.50, "transport": 150.00, "dining": 100.00, ... }`
  - Description: Returns expense totals by category for charting.

### Budgets
- **POST /api/budgets**
  - Request: `{ "category": "groceries", "amount": 200.00, "month": "2025-07" }`
  - Response: `{ "id": 1, "category": "groceries", "amount": 200.00, ... }`
  - Description: Sets or updates a monthly budget.
- **GET /api/budgets**
  - Query Params: `?month=2025-07`
  - Response: `[{ "category": "groceries", "amount": 200.00, "spent": 150.50 }, ...]`
  - Description: Fetches budgets with spent amounts for the specified month.

### Currency Conversion
- **GET /api/currency/convert**
  - Query Params: `?amount=100&from=USD&to=EUR`
  - Response: `{ "convertedAmount": 92.50, "rate": 0.925, "date": "2025-07-17" }`
  - Description: Converts an amount using an external API, caches result for 24 hours.

## 🧑‍💻 Implementation Details
### 1. Project Setup
- **Dependencies** (in `pom.xml`):
  ```xml
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt</artifactId>
      <version>0.9.1</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.33</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
  </dependencies>
  ```
- **Configuration** (`application.properties`):
  ```properties
  spring.datasource.url=jdbc:mysql://localhost:3306/budgetbuddy
  spring.datasource.username=root
  spring.datasource.password=your-password
  spring.jpa.hibernate.ddl-auto=update
  spring.jpa.show-sql=true
  jwt.secret=your-secret-key
  jwt.expiration=86400000
  currency.api.url=https://api.exchangerate-api.com/v4/latest/
  currency.api.key=your-api-key
  ```

### 2. Security Configuration
- Reuse your Campus Sync JWT setup, adapting it for BudgetBuddy:
  ```java
  @Configuration
  @EnableWebSecurity
  public class SecurityConfig {
      @Bean
      public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
          http
              .csrf().disable()
              .authorizeRequests()
              .antMatchers("/api/auth/**").permitAll()
              .anyRequest().authenticated()
              .and()
              .addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
          return http.build();
      }

      @Bean
      public JwtAuthenticationFilter jwtAuthenticationFilter() {
          return new JwtAuthenticationFilter();
      }
  }
  ```
- Implement `JwtAuthenticationFilter` to validate JWT tokens, similar to Campus Sync.

### 3. Expense Service
- Example service to handle expense creation with currency conversion:
  ```java
  @Service
  public class ExpenseService {
      @Autowired
      private ExpenseRepository expenseRepository;
      @Autowired
      private CurrencyConverter currencyConverter;

      public Expense createExpense(ExpenseDTO dto, Long userId) {
          BigDecimal amount = new BigDecimal(dto.getAmount());
          String userCurrency = getUserPreferredCurrency(userId);
          if (!dto.getCurrency().equals(userCurrency)) {
              amount = currencyConverter.convert(dto.getAmount(), dto.getCurrency(), userCurrency);
          }
          Expense expense = new Expense();
          expense.setUserId(userId);
          expense.setAmount(amount);
          expense.setCurrency(userCurrency);
          expense.setCategory(dto.getCategory());
          expense.setMerchant(dto.getMerchant());
          expense.setDate(LocalDate.parse(dto.getDate()));
          return expenseRepository.save(expense);
      }
  }
  ```

### 4. Currency Conversion
- Use RestTemplate to fetch rates from ExchangeRate-API:
  ```java
  @Service
  public class CurrencyConverter {
      @Value("${currency.api.url}")
      private String apiUrl;
      @Value("${currency.api.key}")
      private String apiKey;
      private final RestTemplate restTemplate = new RestTemplate();

      public BigDecimal convert(double amount, String from, String to) {
          String url = apiUrl + from + "?apiKey=" + apiKey;
          ResponseEntity<CurrencyResponse> response = restTemplate.getForEntity(url, CurrencyResponse.class);
          BigDecimal rate = response.getBody().getRates().get(to);
          return BigDecimal.valueOf(amount).multiply(rate).setScale(2, RoundingMode.HALF_UP);
      }
  }
  ```

### 5. Error Handling
- Use a global exception handler for consistent responses:
  ```java
  @RestControllerAdvice
  public class GlobalExceptionHandler {
      @ExceptionHandler(Exception.class)
      public ResponseEntity<ErrorResponse> handleException(Exception ex) {
          ErrorResponse error = new ErrorResponse("Internal Server Error", ex.getMessage());
          return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
      }
  }
  ```

## ⚙️ Setup Instructions
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/YourUsername/BudgetBuddyBackend.git
   cd BudgetBuddyBackend
   ```
2. **Set Up MySQL**:
   - Create a database: `CREATE DATABASE budgetbuddy;`
   - Update `application.properties` with your MySQL credentials.
3. **Get API Key**:
   - Sign up at [ExchangeRate-API](https://www.exchangerate-api.com/) for a free API key.
   - Add the key to `application.properties`.
4. **Run the Backend**:
   ```bash
   mvn clean install
   mvn spring-boot:run
   ```
5. **Test Endpoints**:
   - Use Postman or curl to test `/api/auth/login` and `/api/expenses`.
   - Example: `curl -H "Authorization: Bearer <jwt-token>" http://localhost:8080/api/expenses`

## 📈 Testing Strategy
- **Unit Tests**: Test services (e.g., `ExpenseService`) with JUnit and Mockito.
- **Integration Tests**: Use MockMVC to test API endpoints.
- **Database Tests**: Use `@DataJpaTest` to verify repository queries.
- **Security Tests**: Ensure unauthorized access is blocked (e.g., missing JWT).

## 🚀 Next Steps
1. **Start Small**: Implement authentication and expense endpoints first.
2. **Add Currency Conversion**: Integrate ExchangeRate-API and cache rates (e.g., using Redis).
3. **Test with Mobile App**: Ensure the backend works with your BudgetBuddy frontend (update Retrofit’s BASE_URL).
4. **Optimize**: Add indexes to MySQL for faster queries, implement caching for currency rates.
5. **Extend**: Consider adding push notifications (Firebase) or a web dashboard for analytics.

**Feedback Needed**:
- Do you want sample code for specific endpoints (e.g., `/api/expenses`)?
- Should I provide a detailed database migration script (e.g., Flyway)?
- Any specific feature (e.g., currency conversion, error handling) you’d like to dive deeper into?

This backend plan leverages your Spring Boot and JWT expertise from Campus Sync while introducing financial data management and external API integration. Let me know how you’d like to proceed!