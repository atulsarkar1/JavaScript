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

test('My Test', async ({ page, testInfo }) => {
  // ... Your test logic here ...

  // Take a screenshot
  const screenshotPath = `screenshots/${testInfo.title.replace(/ /g, '_')}.png`;
  await page.screenshot({ path: screenshotPath });

  // Prepare test result data for Excel
  const testResult = {
    TestName: testInfo.title,
    Status: testInfo.status,
    ScreenshotPath: screenshotPath,
  };

  // Store test result in an array (replace with your preferred storage mechanism)
  const testResults: any[] = []; // Initialize an array to store test results
  testResults.push(testResult);

  // Create Excel report after all tests have finished
  // (You might want to do this in an 'afterAll' hook)
  createExcelReport(testResults);
});
