import { test, expect } from '@playwright/test';
import * as fs from 'fs';
import * as xlsx from 'xlsx';
import * as path from 'path';

// Function to create an Excel workbook with test results
const createExcelReport = (testResults: any[]) => {
  const workbook = xlsx.utils.book_new();
  const worksheet = xlsx.utils.json_to_sheet(testResults);
  xlsx.utils.book_append_sheet(workbook, worksheet, 'Test Results');

  // Add images to the worksheet
  testResults.forEach((result, index) => {
    if (result.ScreenshotPath) {
      const imageData = fs.readFileSync(result.ScreenshotPath);
      const image = xlsx.utils.decode_file(result.ScreenshotPath); 
      const imageCell = `C${index + 2}`; // Place image in column C starting from row 2
      xlsx.utils.sheet_add_image(worksheet, {
        image: image,
        type: 'png', 
        position: {
          type: 'oneCell',
          ref: imageCell,
        },
      });
    }
  });

  xlsx.writeFile(workbook, 'test_results.xlsx');
};

test('Login to Google', async ({ page, testInfo }) => {
  // ... (Your Google login logic here) ...

  // Take a screenshot
  const screenshotPath = path.join(__dirname, 'screenshots', `${testInfo.title.replace(/ /g, '_')}.png`);
  await page.screenshot({ path: screenshotPath });

  // Prepare test result data for Excel
  const testResult = {
    TestName: testInfo.title,
    Status: testInfo.status,
    ScreenshotPath: screenshotPath,
  };

  // Store test result in an array
  const testResults: any[] = [];
  testResults.push(testResult);

  // ... (Your assertions here) ... 
});

// Create Excel report after all tests have finished
// (This should be in your test configuration file - playwright.config.ts)
// createExcelReport(testResults); 
