To handle nested JSON schemas containing objects or arrays, we'll ensure that each nested structure is stored and referenced uniquely, providing a placeholder or identifier for arrays or objects in the pipe-delimited string. Here's how to approach it:

Steps:

1. Traverse the JSON schema recursively.


2. Detect nested objects or arrays.


3. Replace nested structures with unique identifiers or placeholders.


4. Generate the pipe-delimited string using top-level properties.




---

Updated TypeScript Function:

// Function to convert JSON schema to pipe-delimited format with unique placeholders for nested structures
function jsonSchemaToPipeDelimited(schema: any, parentKey: string = ''): string {
  const properties = schema.properties;
  const keys: string[] = [];
  const exampleValues: string[] = [];

  for (const key in properties) {
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
console.log(jsonSchemaToPipeDelimited(nestedSchema));


---

Sample Output:

| name | address | contacts | age |
| example_name | [Object: address] | [Array: contacts] | example_age |


---

Explanation:

Objects and Arrays: Replaced with placeholders ([Object: key] or [Array: key]) to indicate the nested structure without expanding it in the table.

Unique Identification: Nested properties are identified by their unique paths (address, contacts).



---

Integration into Cucumber Scenario Outline:

Scenario Outline: Validate nested user input fields
  Given the user provides input
  When the input is processed
  Then the input should be validated successfully

  Examples:
    | name | address | contacts | age |
    | John Doe | [Object: address] | [Array: contacts] | 30 |

This way, nested structures are represented and can be expanded or tested separately if needed. Would you like further customization, such as fully expanding arrays or nested objects?

