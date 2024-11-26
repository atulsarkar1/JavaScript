// Function to convert JSON schema to pipe-delimited format with unique placeholders for nested structures
function jsonSchemaToPipeDelimited(schema: any, parentKey: string = ''): string {
  const properties = schema.properties;
  
  // Validate the schema structure
  if (!properties) {
    throw new Error("Invalid schema: 'properties' field is required.");
  }

  const keys: string[] = [];
  const exampleValues: string[] = [];

  // Traverse through each property
  for (const key in properties) {
    if (properties.hasOwnProperty(key)) {
      const prop = properties[key];
      const uniqueKey = parentKey ? `${parentKey}_${key}` : key;

      if (prop.type === 'object') {
        // Handle nested object by storing its reference key
        keys.push(key);
        exampleValues.push(`[Object: ${uniqueKey}]`);
      } else if (prop.type === 'array') {
        // Handle arrays, assuming a uniform type for simplicity
        keys.push(key);
        exampleValues.push(`[Array: ${uniqueKey}]`);
      } else {
        // Handle primitive types
        keys.push(key);
        exampleValues.push(`example_${key}`);
      }
    }
  }

  // Construct header and example rows
  const headerRow = `| ${keys.join(' | ')} |`;
  const exampleRow = `| ${exampleValues.join(' | ')} |`;

  return `${headerRow}\n${exampleRow}`;
}

// Example JSON schema with nested objects and arrays
const nestedSchema = {
  type: "object",
  properties: {
    name: { type: "string" },
    address: {
      type: "object",
      properties: {
        city: { type: "string" },
        zip: { type: "integer" }
      }
    },
    contacts: {
      type: "array",
      items: {
        type: "object",
        properties: {
          phone: { type: "string" },
          email: { type: "string" }
        }
      }
    },
    age: { type: "integer" }
  }
};

// Convert and print result
try {
  console.log(jsonSchemaToPipeDelimited(nestedSchema));
} catch (error) {
  console.error(error.message);
}