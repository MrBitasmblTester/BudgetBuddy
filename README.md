# BudgetBuddy — Smart Personal Finance Tracker

BudgetBuddy helps users manage expenses, set savings goals, and receive AI-powered recommendations for smarter financial decisions. The application is a full-stack solution using PyTorch for the ML components, GraphQL as the API layer, ASP.NET Core for the back end, MongoDB for persistence, and Angular for the front end.

## Tech Stack

- PyTorch (AI/ML)
- GraphQL (API)
- ASP.NET Core (Back-End)
- MongoDB (Database)
- Angular (Front-End)

## Requirements

- Produce a complete README using the specified structure
- Refer explicitly to the selected stacks: PyTorch, GraphQL, ASP.NET Core, MongoDB, Angular
- Include stack-specific setup commands and environment variable instructions
- Provide implementation steps describing how components integrate
- Include API endpoints only where implied by the stack or requirements (GraphQL endpoint)
- Do not introduce unrelated technologies
- Do not generalize; always refer to the specific stack
- Do not skip required files normally used in the selected stacks

## Installation

Prerequisites (install these first):

- .NET SDK 6 or later
- Node.js 16 or later and npm
- Angular CLI (globally) — install with npm
- Python 3.8 or later and pip
- MongoDB server or MongoDB Atlas account

Example commands to prepare the environment:

1) Install .NET SDK (verify):


dotnet --version


2) Install Node and Angular CLI (verify):


node --version
npm --version
npm install -g @angular/cli
ng --version


3) Prepare Python environment and install PyTorch (CPU example):


python -m venv .venv
# Activate the venv (Linux/macOS)
source .venv/bin/activate
# Activate on Windows
# .venv\Scripts\activate
pip install --upgrade pip
pip install torch torchvision torchaudio
pip install -r ai/requirements.txt


Note: For GPU-enabled PyTorch builds, follow official PyTorch install instructions and use the correct wheel for CUDA.

4) MongoDB

- For local development: install MongoDB Community and run mongod
- For hosted DB: create a MongoDB Atlas cluster and copy the connection string

Environment variables (create a .env or set these in your deployment environment):

- MONGO_URI — MongoDB connection string (for example mongodb://localhost:27017/budgetbuddy)
- ASPNETCORE_ENVIRONMENT — Development or Production
- JWT_SECRET — secret string for token signing (if implementing auth)
- GRAPHQL_ENDPOINT — GraphQL endpoint (default: /graphql)
- PYTORCH_MODEL_PATH — path to saved PyTorch model file used for inference (ai/models/expenses_model.pt)
- ANGULAR_API_URL — base URL for the API used by the Angular front end (for example http://localhost:5000/graphql)

Project-specific installation steps (backend, frontend, ai):

Backend (ASP.NET Core + GraphQL):


cd backend
# restore packages
dotnet restore
# add GraphQL server and MongoDB driver packages (example packages)
# dotnet add package HotChocolate.AspNetCore
# dotnet add package MongoDB.Driver
# run
dotnet run --project BudgetBuddy.Api


Front end (Angular):


cd frontend
npm install
# install GraphQL client for Angular
npm install apollo-angular @apollo/client graphql
ng serve --configuration development
# app served at http://localhost:4200


AI (PyTorch):


cd ai
pip install -r requirements.txt
python train.py   # trains and outputs a model file to ai/models
python infer.py --model-path ai/models/expenses_model.pt --input sample_input.json


## Usage

Minimal dev workflow to run the full stack locally:

1) Ensure MongoDB is running and MONGO_URI is set
2) Start the ASP.NET Core backend which hosts the GraphQL API


cd backend
dotnet run


By default the GraphQL endpoint will be available at http://localhost:5000/graphql (adjust port in launchSettings or environment)

3) Start the PyTorch model training or place a pre-trained model at the path referenced by PYTORCH_MODEL_PATH


cd ai
python train.py
# resulting model saved to ai/models/expenses_model.pt


4) Start the Angular front end


cd frontend
ng serve
# open http://localhost:4200


5) Interact with the GraphQL API via Playground / Banana Cake Pop (if configured) at the GraphQL endpoint or through the Angular UI

Typical flows:

- Create an expense via a GraphQL mutation
- Query expenses and savings goals via GraphQL queries
- Request AI-driven recommendations via a GraphQL field that triggers a PyTorch-based inference
- Front end shows visualizations and allows goal management

## Implementation Steps

1) Create the repository skeleton
   - Top-level folders: backend, frontend, ai, docs
   - Add README, .gitignore, and LICENSE

2) Backend: initialize ASP.NET Core project
   - Create an ASP.NET Core Web API project (BudgetBuddy.Api)
   - Include necessary files: Program.cs, appsettings.json, BudgetBuddy.Api.csproj
   - Add package references for GraphQL server and MongoDB driver

3) GraphQL schema and server setup
   - Add a GraphQL schema file or schema-first definitions (schema.graphql) describing types: Expense, Goal, Recommendation, User
   - Implement resolvers that map to repository operations
   - Expose the GraphQL endpoint at /graphql
   - Configure GraphQL playground for development

4) MongoDB integration
   - Add a data access layer using MongoDB.Driver
   - Create collections: expenses, goals, users, recommendations
   - Add repository classes (for example ExpensesRepository) and register them in the DI container
   - Ensure appsettings.json and environment variables contain MONGO_URI and database name

5) Domain models and DTOs
   - Add C# models matching the GraphQL types and MongoDB documents
   - Add mapping between domain models and GraphQL types if necessary

6) AI (PyTorch) model and integration
   - Create an ai folder with model definition (ai/model.py), training script (ai/train.py), and inference helper (ai/infer.py)
   - Training script outputs a serialized model to ai/models/expenses_model.pt
   - Inference helper loads the model and exposes a function to produce recommendation objects given user and expense data
   - Add requirements.txt listing torch and any numeric libs (numpy, pandas)

7) Connect GraphQL resolvers to AI inference
   - Add a RecommendationResolver or a field on the User type that executes the PyTorch inference
   - For synchronous performance, load the model once at application startup and reuse it for inference
   - Ensure model path is read from PYTORCH_MODEL_PATH env var or config

8) Authentication & Authorization (optional but recommended)
   - Add simple JWT-based authentication using ASP.NET Core middleware
   - Protect mutations that mutate user data
   - Provide JWT_SECRET in environment variables

9) Front end (Angular)
   - Initialize Angular app in frontend folder using Angular CLI
   - Add GraphQL client using apollo-angular and configure with ANGULAR_API_URL
   - Create services for GraphQL queries and mutations: expenses.service.ts, goals.service.ts, recommendations.service.ts
   - Build components for dashboards, expense list, add expense, manage goals, and recommendations
   - Add environment files in src/environments with apiURL and other config

10) End-to-end data flow and testing
   - Add integration tests for backend GraphQL queries and resolvers
   - Add unit tests for ai/train and inference scripts
   - Test full flow: create user, add expenses, set goals, request recommendations, see UI updates

11) Add required files for each stack
   - backend: BudgetBuddy.Api.csproj, Program.cs, appsettings.Development.json, schema.graphql
   - frontend: package.json, angular.json, src/environments/environment.ts, src/main.ts
   - ai: requirements.txt, train.py, infer.py, model.py, models/expenses_model.pt (gitignored large artifacts)
   - Add .gitignore entries for node_modules, bin/obj, .venv, ai/models (or store models in a release artifact)

12) Documentation and example data
   - Add example GraphQL queries and mutations in docs/graphql-queries.md
   - Provide sample CSV/JSON for expense imports

## API Endpoints

Because the project uses GraphQL, the primary endpoint is a single GraphQL endpoint rather than REST endpoints.

- GraphQL endpoint (POST/GET for queries and mutations)


/graphql   # POST queries and mutations here


If subscriptions are enabled (optional):


/ws/graphql   # WebSocket endpoint for GraphQL subscriptions (if configured)


Example GraphQL queries and mutations:

- Create an expense (mutation):


mutation CreateExpense {
  createExpense(input: {
    userId: "user123",
    amount: 12.50,
    category: Food,
    date: "2025-01-10",
    note: "Lunch"
  }) {
    id
    amount
    category
    date
  }
}


- Query expenses (query):


query GetExpenses {
  expenses(userId: "user123", limit: 50) {
    id
    amount
    category
    date
    note
  }
}


- Request AI recommendations (query or field on User):


query GetRecommendations {
  recommendations(userId: "user123") {
    suggestion
    impactEstimate
    confidence
  }
}


- Create a savings goal (mutation):


mutation CreateGoal {
  createGoal(input: {
    userId: "user123",
    name: "Emergency Fund",
    targetAmount: 3000,
    monthlyContribution: 200
  }) {
    id
    name
    targetAmount
    monthlyContribution
  }
}


Notes:
- GraphQL schema should include types for Expense, Goal, Recommendation, and User.
- Resolvers for Recommendations should call into the AI inference component which uses PyTorch to compute personalized advice.

---

If you want, I can generate starter files for each folder (backend Program.cs and GraphQL setup, frontend Angular service examples, and a basic PyTorch model/training script) so you can scaffold the repository quickly.