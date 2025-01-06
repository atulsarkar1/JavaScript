import { test, expect } from '@playwright/test';
import * as fs from 'fs';
import * as xlsx from 'xlsx';

// Function to create an Excel workbook with test results
const createExcelReport = (testResults: any[]) => {
  const ws = xlsx.utils.json_to_sheet(testResults);
  const wb = xlsx.utils.book_new();
  xlsx.utils.book_append_sheet(wb, ws, 'Test Results');
  xlsx.writeFile(wb, 'test_results.xlsx');
};

test('Login to Google', async ({ page, testInfo }) => {
  // Navigate to Google
  await page.goto('https://www.google.com');

  // Click the sign-in button (adjust the selector as needed)
  await page.click('a[href*="/accounts/"]'); 

  // Enter your email address
  await page.fill('input[type="email"]', 'your_email@gmail.com');
  await page.keyboard.press('Enter');

  // Enter your password
  await page.fill('input[type="password"]', 'your_password');
  await page.keyboard.press('Enter');

  // Take a screenshot
  const screenshotPath = `screenshots/${testInfo.title.replace(/ /g, '_')}.png`;
  await page.screenshot({ path: screenshotPath });

  // Prepare test result data for Excel
  const testResult = {
    TestName: testInfo.title,
    Status: testInfo.status,
    ScreenshotPath: screenshotPath,
  };

  // Store test result in an array
  const testResults: any[] = []; // Initialize an array to store test results
  testResults.push(testResult);

  // Assert that the user is logged in 
  // (e.g., check for the user's profile picture or name)
  const usernameElement = await page.$('//span[contains(text(), "Your Name")]'); 
  await expect(usernameElement).toBeVisible(); 

});

// Create Excel report after all tests have finished
// (This should be in an 'afterAll' hook in your test configuration)
// createExcelReport(testResults); 

// Note: 
// 1. Replace 'your_email@gmail.com' and 'your_password' with your actual credentials.
// 2. Adjust the selectors ('a[href*="/accounts/"]', 'input[type="email"]', etc.) 
//    if the Google login page elements have changed.
// 3. The 'afterAll' hook and the `createExcelReport` call should be placed 
//    in your test configuration file (e.g., playwright.config.ts).

