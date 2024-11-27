import * as fs from 'fs';
import * as path from 'path';

// Function to extract and save nested objects/arrays and replace them with references
function extractAndReplace(jsonData: any, outputDir: string): any {
  const extractedData: Record<string, string> = {};

  const processObject = (obj: any, parentKey: string = ''): any => {
    if (Array.isArray(obj)) {
      // Extract arrays and save to a file
      const arrayKey = `${parentKey}`;
      extractedData[arrayKey] = saveToFile(obj, arrayKey, outputDir);
      return `{{${arrayKey}}}`;
    } else if (typeof obj === 'object' && obj !== null) {
      // Recursively process each key in the object
      const newObj: any = {};
      for (const key in obj) {
        const compositeKey = parentKey ? `${parentKey}_${key}` : key;
        newObj[key] = processObject(obj[key], compositeKey);
      }
      return newObj;
    }
    return obj; // Return primitive values as is
  };

  // Save extracted data to separate files
  const saveToFile = (data: any, key: string, outputDir: string): string => {
    const fileName = `${key}.json`;
    const filePath = path.join(outputDir, fileName);
    fs.writeFileSync(filePath, JSON.stringify(data, null, 2));
    return fileName;
  };

  const processedData = processObject(jsonData);
  return { processedData, extractedData };
}

// Function to convert JSON structure to pipe-delimited string
function jsonToPipeDelimited(jsonData: any, extractedData: Record<string, string>): string {
  const keys: string[] = [];
  const exampleValues: string[] = [];

  const processObject = (obj: any, parentKey: string = '') => {
    for (const key in obj) {
      const value = obj[key];
      const compositeKey = parentKey ? `${parentKey}_${key}` : key;

      if (typeof value === 'string' && value.startsWith('{{')) {
        // Handle extracted references
        keys.push(compositeKey);
        exampleValues.push(value);
      } else if (typeof value === 'object' && value !== null) {
        // Handle nested objects recursively
        processObject(value, compositeKey);
      } else {
        // Handle primitive values
        keys.push(compositeKey);
        exampleValues.push(`example_${compositeKey}`);
      }
    }
  };

  processObject(jsonData);

  // Construct header and example rows
  const headerRow = `| ${keys.join(' | ')} |`;
  const exampleRow = `| ${exampleValues.join(' | ')} |`;

  return `${headerRow}\n${exampleRow}`;
}

// Example usage
const jsonFilePath = 'path/to/your/long.json';
const outputDir = 'path/to/output/dir'; // Ensure this directory exists

// Load JSON data from file
const jsonData = JSON.parse(fs.readFileSync(jsonFilePath, 'utf-8'));

// Extract and replace nested structures
const { processedData, extractedData } = extractAndReplace(jsonData, outputDir);

// Convert and print Scenario Outline format with references
const pipeDelimited = jsonToPipeDelimited(processedData, extractedData);
console.log('Processed Data:', JSON.stringify(processedData, null, 2));
console.log('Pipe Delimited:', pipeDelimited);
console.log('Extracted Files:', extractedData);