import * as fs from 'fs';
import * as path from 'path';

// Function to retrieve data from a nested array or object within the JSON structure
function getNestedData(jsonData: any[], nestedPath: string): any[] {
  const results: any[] = [];

  // Traverse each object in the JSON array
  jsonData.forEach(item => {
    const value = nestedPath.split('.').reduce((acc, key) => {
      // If the current key exists in the object, navigate deeper
      if (acc && acc[key] !== undefined) {
        return acc[key];
      }
      return null;
    }, item);

    // If it's an array, push the contents of the array to results
    if (Array.isArray(value)) {
      results.push(...value);
    } else if (value !== null) {
      // If it's a primitive or an object, include the value
      results.push(value);
    }
  });

  return results;
}

// Function to load JSON data from a file
function loadJsonData(filePath: string): any[] {
  const fileContent = fs.readFileSync(filePath, 'utf-8');
  return JSON.parse(fileContent);
}

// Function to convert JSON structure to pipe-delimited string format (like for Scenario Outline)
function jsonToPipeDelimited(jsonData: any[], fileName: string): string {
  const keys: string[] = [];
  const exampleValues: string[] = [];

  // Traverse the first item in the array to extract keys
  const processObject = (obj: any, parentKey: string = '') => {
    for (const key in obj) {
      const value = obj[key];
      const compositeKey = parentKey ? `${parentKey}_${key}` : key;

      if (Array.isArray(value)) {
        // Handle arrays
        keys.push(compositeKey);
        exampleValues.push(`[Array: ${compositeKey}]`);
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

  processObject(jsonData[0]); // Process the first object to extract headers

  // Construct header and example rows
  const headerRow = `| ${keys.join(' | ')} |`;
  const exampleRow = `| ${exampleValues.join(' | ')} |`;

  // Include fileName as part of the result
  const formattedFileName = `${fileName},${exampleValues.join(' | ')}`;
  
  return `${headerRow}\n${exampleRow}\nFile: ${formattedFileName}`;
}

// Example usage:

// Path to your long.json file
const jsonFilePath = 'path/to/your/long.json';

// Get the file name without extension
const fileName = path.basename(jsonFilePath, '.json');

// Load JSON data from file
const jsonData = loadJsonData(jsonFilePath);

// Retrieve and print data from the 'batters.batter' array
const batterData = getNestedData(jsonData, 'batters.batter');
console.log('Batter Data:', batterData);

// Convert and print Scenario Outline format with file name
const pipeDelimited = jsonToPipeDelimited(jsonData, fileName);
console.log('Pipe Delimited:', pipeDelimited);