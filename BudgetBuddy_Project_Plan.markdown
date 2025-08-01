# BudgetBuddy - Personal Finance Manager: Project Plan

## 🚀 Project Overview
**BudgetBuddy** is a mobile app designed to help users manage their personal finances by tracking expenses, setting budgets, and analyzing spending patterns. The app uses OCR to scan receipts for easy expense logging, supports multi-currency transactions, and provides visual insights through charts. It’s secured with JWT authentication, ensuring user data privacy. The backend, built with Spring Boot, handles data storage and currency conversion, while the Android frontend leverages Jetpack Compose and CameraX for a modern, seamless experience.

This project builds on your **Campus Sync** expertise (e.g., CameraX, Google ML Kit, Retrofit, Hilt, JWT, Spring Boot) and introduces new skills like OCR processing, currency conversion APIs, and data visualization with Chart.js.

## ✨ Key Features
1. **Receipt Scanning**: Use Google ML Kit’s OCR to extract expense details (e.g., amount, date, merchant) from receipt images.
2. **Budget Tracking**: Allow users to categorize expenses (e.g., groceries, transport) and set monthly budgets with overspending alerts.
3. **Financial Insights**: Visualize spending patterns with charts (e.g., pie chart for expense categories, line chart for monthly trends).
4. **Secure Storage**: Protect user data with JWT-based authentication and role-based access (e.g., user vs. admin for future extensions).
5. **Multi-Currency Support**: Handle transactions in different currencies, with real-time conversion rates fetched via an API.

## 🛠️ Architecture
### Frontend (Android)
- **UI Framework**: Jetpack Compose for a modern, reactive UI.
- **Navigation**: Single-activity architecture with Navigation Component for screen transitions (e.g., from budget overview to receipt scanner).
- **Camera**: CameraX for capturing receipt images.
- **ML**: Google ML Kit for OCR to extract text from receipts.
- **Networking**: Retrofit for API calls to fetch/store expenses, budgets, and currency rates.
- **Dependency Injection**: Hilt for managing dependencies (e.g., Retrofit, ViewModels).
- **State Management**: ViewModels with StateFlow/LiveData for reactive UI updates.
- **Local Storage**: Shared Preferences for storing JWT tokens and user settings.

### Backend (Spring Boot)
- **Framework**: Spring Boot with Spring Security for JWT authentication.
- **Database**: MySQL for structured data (users, expenses, budgets).
- **APIs**: REST endpoints for user management, expense logging, budget tracking, and currency conversion.
- **External APIs**: Integrate a currency conversion API (e.g., ExchangeRate-API).
- **Security**: JWT token verification for all endpoints, ensuring secure data access.

### Data Flow
1. User scans a receipt → CameraX captures image → Google ML Kit extracts text (amount, date) → App sends data to backend via Retrofit.
2. Backend validates JWT, processes expense data, converts currency if needed, and stores in MySQL.
3. App fetches expense/budget data, displays in Jetpack Compose UI, and visualizes with charts.
4. Push notifications (optional) alert users of overspending or budget updates.

## 🔑 Implementation Details
### 1. Receipt Scanning
- **Tech**: Use CameraX to capture receipt images, Google ML Kit’s Text Recognition API to extract text.
- **Workflow**:
  - Create a Compose screen with a CameraX preview for scanning receipts.
  - Process the image with ML Kit to extract amount, date, and merchant (use regex to parse structured data).
  - Send extracted data to the backend via a Retrofit POST request (e.g., `/api/expenses`).
- **Challenges**: Handle poor image quality or varied receipt formats. Use preprocessing (e.g., contrast adjustment) and regex to improve accuracy.
- **Your Skills**: Leverages your CameraX and ML Kit experience from Campus Sync’s QR scanning.

### 2. Budget Tracking
- **Tech**: Jetpack Compose for UI, Retrofit for API calls, MySQL for storage.
- **Workflow**:
  - Create a Compose screen to set monthly budgets per category (e.g., groceries: $200).
  - Use Retrofit to fetch and update budgets (`/api/budgets`).
  - Display expenses vs. budget in a Compose UI, with alerts (e.g., Snackbar) for overspending.
- **Challenges**: Ensure real-time updates when expenses are added. Use StateFlow for reactive UI.
- **Your Skills**: Builds on your state management and Retrofit skills from Campus Sync.

### 3. Financial Insights
- **Tech**: Chart.js for web-based dashboards (optional), Jetpack Compose with a custom canvas for mobile charts.
- **Workflow**:
  - Fetch expense data by category via Retrofit (`/api/expenses/categories`).
  - Display a pie chart for category distribution (e.g., 40% groceries, 30% transport).
  - Optionally, create a web dashboard with Chart.js for detailed analytics.
- **Challenges**: Optimize chart rendering for large datasets. Use pagination for API calls.
- **Your Skills**: Introduces data visualization, complementing your UI and API skills.

### 4. Secure Storage
- **Tech**: Spring Security with JWT, Shared Preferences for token storage.
- **Workflow**:
  - Authenticate users via `/api/auth/login`, store JWT in Shared Preferences.
  - Include JWT in API headers for secure requests (e.g., `/api/expenses`).
  - Backend verifies JWT for all protected endpoints.
- **Challenges**: Handle token expiration and refresh. Implement logout functionality.
- **Your Skills**: Directly reuses your JWT authentication from Campus Sync.

### 5. Multi-Currency Support
- **Tech**: ExchangeRate-API or similar for real-time rates, Spring Boot for backend processing.
- **Workflow**:
  - Allow users to select a currency for expenses (e.g., USD, EUR).
  - Backend fetches conversion rates via API and stores converted amounts in MySQL.
  - Display expenses in the user’s preferred currency in the UI.
- **Challenges**: Cache rates to reduce API calls. Handle rate fluctuations.
- **New Skill**: Currency conversion APIs, building on your Retrofit experience.

## ⚙️ Getting Started
### Prerequisites
- **Android Studio**: Latest version.
- **JDK**: 11 or higher.
- **MySQL**: For backend database.
- **API Key**: Sign up for a free currency conversion API (e.g., [ExchangeRate-API](https://www.exchangerate-api.com/)).
- **Android Device/Emulator**: API level 24+.

### Setup Instructions
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/YourUsername/BudgetBuddy.git
   cd BudgetBuddy
   ```
2. **Backend Setup**:
   - Create a new Spring Boot project or extend **CampusSyncBackend**.
   - Set up MySQL and configure `application.properties` with database credentials.
   - Implement REST endpoints (e.g., `/api/expenses`, `/api/budgets`, `/api/auth`).
   - Integrate ExchangeRate-API for currency conversion.
   - Run the backend: `mvn spring-boot:run`.
3. **Frontend Setup**:
   - Open the project in Android Studio.
   - Update Retrofit’s `BASE_URL` in the network module to point to your backend (e.g., `http://192.168.1.5:8080/`).
   - Add Google ML Kit and CameraX dependencies in `build.gradle`.
   - Sync and run the app on a device/emulator.
4. **Testing**:
   - Test receipt scanning with sample receipts.
   - Verify budget alerts and currency conversion.
   - Ensure JWT authentication secures all API calls.

## 📊 Sample Chart: Expense Categories
Here’s a sample pie chart to visualize expense categories, using hypothetical data (e.g., monthly spending). This can be implemented in a web dashboard with Chart.js or adapted for mobile with Compose.

```chartjs
{
  "type": "pie",
  "data": {
    "labels": ["Groceries", "Transport", "Dining", "Entertainment", "Utilities"],
    "datasets": [{
      "label": "Monthly Expenses",
      "data": [200, 150, 100, 80, 70],
      "backgroundColor": ["#FF6F61", "#6B7280", "#FBBF24", "#34D399", "#3B82F6"],
      "borderColor": ["#FFFFFF", "#FFFFFF", "#FFFFFF", "#FFFFFF", "#FFFFFF"],
      "borderWidth": 1
    }]
  },
  "options": {
    "responsive": true,
    "plugins": {
      "legend": {
        "position": "top",
        "labels": {
          "color": "#1F2937"
        }
      },
      "title": {
        "display": true,
        "text": "Expense Categories (USD)",
        "color": "#1F2937"
      }
    }
  }
}
```

**Note**: This chart assumes sample data. Let me know if you have specific data or want a different chart type (e.g., bar, line).

## 📈 Learning Outcomes
- **Reused Skills**: CameraX, Google ML Kit, Retrofit, Hilt, JWT, Spring Boot, state management.
- **New Skills**:
  - **OCR Processing**: Extract structured data from receipts using ML Kit.
  - **Currency Conversion**: Integrate and handle real-time API data.
  - **Data Visualization**: Create charts for financial insights (Chart.js or Compose).
- **Portfolio Impact**: A practical app with real-world utility, showcasing AI, security, and modern UI/UX.

## 🚀 Next Steps
1. **Define Scope**: Decide if you want to include optional features (e.g., push notifications, web dashboard).
2. **Database Schema**:
   - **Users**: `id`, `email`, `password`, `preferred_currency`.
   - **Expenses**: `id`, `user_id`, `amount`, `currency`, `category`, `date`, `merchant`.
   - **Budgets**: `id`, `user_id`, `category`, `amount`, `month`.
3. **API Endpoints**:
   - `POST /api/auth/login`: Authenticate user, return JWT.
   - `POST /api/expenses`: Log expense from receipt data.
   - `GET /api/expenses/categories`: Fetch expense breakdown for charts.
   - `POST /api/budgets`: Set/update budget.
4. **Prototype**: Start with receipt scanning and budget tracking, then add visualizations and currency support.
5. **Feedback**: Share your progress, and I can help refine features, debug issues, or provide code snippets.

What do you think of this plan? Want to dive deeper into any feature (e.g., OCR, charts) or adjust the scope? Let me know if you want the chart modified or additional resources!