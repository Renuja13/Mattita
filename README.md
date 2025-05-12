# Mattitaname: Node.js CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}

    - name: Cache Node Modules
      uses: actions/cache@v4
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-

    - name: Install dependencies
      run: npm ci

    - name: Run Linter
      run: npm run lint

    - name: Run Tests
      run: npm test

    - name: Build Project
      run: npm run build

    - name: Upload Build Artifacts
      uses: actions/upload-artifact@v4
      with:
        name: production-build
        path: dist/

    - name: Comment on PR with Build Info
      if: github.event_name == 'pull_request'
      uses: marocchino/sticky-pull-request-comment@v2
      with:
        message: |
          ✅ Build completed successfully for Node.js version ${{ matrix.node-version }}!
          Linting, testing, and build passed. Artifacts are attached.
          "scripts": {
  "lint": "eslint .",
  "test": "jest",
  "build": "tsc"
}"scripts": {
  "lint": "eslint . --ext .js,.ts,.tsx",
  "lint:fix": "eslint . --ext .js,.ts,.tsx --fix",
  "test": "jest --coverage",
  "test:watch": "jest --watch",
  "test:ci": "jest --ci --coverage",
  "build": "tsc",
  "build:watch": "tsc --watch",
  "clean": "rm -rf dist coverage node_modules",
  "prepare": "husky install",
  "prettify": "prettier --write .",
  "check:types": "tsc --noEmit",
  "start": "node dist/index.js",
  "dev": "ts-node-dev src/index.ts"
}name: Node.js CI & Bot Automation

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}

    - name: Cache Node Modules
      uses: actions/cache@v4
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-

    - name: Install dependencies
      run: npm ci

    - name: Run Linter
      run: npm run lint

    - name: Run Tests
      run: npm run test:ci

    - name: Build Project
      run: npm run build

    - name: Upload Build Artifacts
      uses: actions/upload-artifact@v4
      with:
        name: production-build
        path: dist/

    - name: Comment on PR with Build Info
      if: github.event_name == 'pull_request'
      uses: marocchino/sticky-pull-request-comment@v2
      with:
        message: |
          ✅ Build completed on Node.js ${{ matrix.node-version }}.
          - Lint, Test, and Build succeeded.
          - Artifacts are uploaded.

    - name: Assign Reviewers
      uses: kentaro-m/auto-assign-action@v1.2.1

    - name: Auto Label PR
      uses: actions/labeler@v5

    - name: Check Conventional Commits
      uses: amannn/action-semantic-pull-request@v5
      with:
        types: |
          feat
          fix
          chore
          docs
          style
          refactor
          perf
          test
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

    - name: Auto Approve Dependabot PRs
      if: ${{ github.actor == 'dependabot[bot]' }}
      uses: hmarr/auto-approve-action@v4
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
