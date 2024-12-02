import * as xlsx from 'xlsx';
import { promises as fs } from 'fs';

// Function to extract placeholders (columns) from the feature file
async function extractPlaceholders(featureFilePath: string): Promise<string[]> {
  const featureContent = await fs.readFile(featureFilePath, 'utf8');
  const placeholderMatches = featureContent.match(/<([^>]+)>/g) || [];
  
  // Remove < and > from the matched placeholders
  return placeholderMatches.map(match => match.replace(/[<>]/g, ''));
}

// Function to read and filter Excel data based on required columns
function readAndFilterExcelData(filePath: string, sheetName: string, requiredColumns: string[]): string[][] {
  const workbook = xlsx.readFile(filePath);
  const worksheet = workbook.Sheets[sheetName];
  const jsonData: any[] = xlsx.utils.sheet_to_json(worksheet, { header: 1 });

  const headerRow: string[] = jsonData[0];
  
  // Get indices of required columns
  const columnIndices = requiredColumns.map(col => headerRow.indexOf(col)).filter(index => index !== -1);

  // Filter and format rows
  const filteredData = jsonData.map(row => columnIndices.map(index => row[index]));
  
  return [requiredColumns, ...filteredData.slice(1)]; // Ensure header is the first row
}

// Convert array to pipe-delimited string for Examples
function formatExamples(data: string[][]): string {
  const rows = data.map(row => '|' + row.join('|') + '|').join('\n');
  return rows;
}

// Function to update the feature file with the new Examples section
async function updateFeatureFile(featureFilePath: string, examplesData: string) {
  const featureFileContent = await fs.readFile(featureFilePath, 'utf8');

  // Replace existing Examples section or insert if none exists
  const updatedContent = featureFileContent.replace(
    /Examples:\s*\|[^]*?\|[^]*?\n/g,
    `Examples:\n${examplesData}\n`
  );

  await fs.writeFile(featureFilePath, updatedContent);
}

// Main function to integrate everything
(async () => {
  const featureFilePath = 'path/to/featureFile.feature';
  const excelFilePath = 'path/to/data.xlsx';
  const sheetName = 'Sheet1';

  const requiredColumns = await extractPlaceholders(featureFilePath);
  const filteredData = readAndFilterExcelData(excelFilePath, sheetName, requiredColumns);
  const formattedExamples = formatExamples(filteredData);

  await updateFeatureFile(featureFilePath, formattedExamples);
})();