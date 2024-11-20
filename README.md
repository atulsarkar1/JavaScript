# JavaScript
A code repo. for Java Script 
To ensure the JSON file path is dynamically extracted from the Scenario Outline Examples section of the feature file, we need to enhance the generated login.runner.ts file to read and parse the file path directly from the feature file. Here’s the updated complete framework structure and implementation:


---

Updated Framework Structure

project-root/
├── data/
│   └── data.json             # Test data in JSON format
├── features/
│   └── login.feature         # Feature file with JSON reference
├── generated/
│   └── login.runner.ts       # Auto-generated runner file (via bddgen)
├── utils/
│   └── test-data.ts          # Utility for loading JSON data
├── playwright.config.ts      # Playwright configuration file
├── package.json              # Project dependencies and scripts
├── tsconfig.json             # TypeScript configuration
└── README.md                 # Documentation


---

Updated Feature File

features/login.feature

This file remains unchanged and includes the path to the JSON file in the Examples section.

Feature: Login functionality

  Scenario Outline: Login with valid credentials
    Given I am on the login page
    When I login with "<username>" and "<password>"
    Then I should be redirected to the dashboard

    Examples:
      | /data/data.json |


---

Generated Runner File

generated/login.runner.ts

The runner file dynamically reads the JSON file path from the Examples section of the feature file and loads the test data accordingly.

import { test, expect } from '@playwright/test';
import * as fs from 'fs';
import * as path from 'path';

// Function to extract JSON file path from the feature file
function getJsonFilePathFromFeature(featurePath: string): string {
  const featureContent = fs.readFileSync(featurePath, 'utf-8');
  const exampleLine = featureContent
    .split('\n')
    .find((line) => line.trim().startsWith('|') && line.includes('.json'));

  if (!exampleLine) {
    throw new Error('No valid JSON file path found in the Examples section of the feature file.');
  }

  const jsonFilePath = exampleLine.match(/\|(.*)\|/)?.[1]?.trim();
  if (!jsonFilePath) {
    throw new Error('Invalid JSON file path format in the feature file.');
  }

  return jsonFilePath;
}

// Function to load test data from the JSON file
function loadTestData(jsonFilePath: string): { username: string; password: string }[] {
  const absolutePath = path.resolve(__dirname, `..${jsonFilePath}`);
  if (!fs.existsSync(absolutePath)) {
    throw new Error(`JSON file not found at path: ${absolutePath}`);
  }
  return JSON.parse(fs.readFileSync(absolutePath, 'utf-8'));
}

// Runner script
test.describe('Login functionality', () => {
  const featurePath = path.resolve(__dirname, '../features/login.feature'); // Path to the feature file
  const jsonFilePath = getJsonFilePathFromFeature(featurePath); // Extract JSON path from feature file
  const testData = loadTestData(jsonFilePath); // Load test data from JSON file

  testData.forEach(({ username, password }) => {
    test(`Login with username: ${username} and password: ${password}`, async ({ page }) => {
      // Given I am on the login page
      await page.goto('https://example.com/login');

      // When I login with "<username>" and "<password>"
      await page.fill('input[name="username"]', username);
      await page.fill('input[name="password"]', password);
      await page.click('button[type="submit"]');

      // Then I should be redirected to the dashboard
      await expect(page).toHaveURL(/dashboard/);
    });
  });
});


---

Updated Utility Function (Optional)

utils/test-data.ts

You can include this utility for reusability if multiple tests need to load JSON data.

import * as fs from 'fs';
import * as path from 'path';

export const loadTestData = (relativePath: string) => {
  const absolutePath = path.resolve(__dirname, `..${relativePath}`);
  return JSON.parse(fs.readFileSync(absolutePath, 'utf-8'));
};


---

Key Changes

1. Dynamic Path Extraction:

The path to the JSON file is extracted directly from the Examples section of the feature file.



2. Feature File Parsing:

The getJsonFilePathFromFeature function reads the feature file and parses the JSON file path from the Examples table.



3. Error Handling:

The script validates that the JSON file path exists and throws errors if the path is invalid or the file does not exist.





---

Workflow

Step 1: Add a Feature File

Write Gherkin-style feature files in the features/ folder.

Step 2: Generate the Runner File

Run the following command to generate the Playwright runner file:

npm run bddgen

Step 3: Run the Tests

Run the tests using Playwright:

npm test

Step 4: View Reports

Generate and view HTML reports for test execution:

npx playwright show-report


---

Framework Commands

package.json

Update the scripts section to integrate bddgen and Playwright:

{
  "scripts": {
    "bddgen": "npx bddgen generate features/ -o generated/",
    "test": "npm run bddgen && npx playwright test"
  }
}


---

Advantages

The JSON file path is fully dynamic and driven by the feature file content.

The framework automatically adapts to new JSON paths without requiring manual changes to the runner file.

The modular design supports easy extension for additional test cases and scenarios.


This updated framework is highly dynamic, flexible, and robust for managing feature-driven tests in Playwright.


