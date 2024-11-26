function jsonToPipeDelimited(jsonData: any[]): string {
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

  return `${headerRow}\n${exampleRow}`;
}

// Example JSON data (replace this with your actual data)
const jsonData = [
  {
    id: "0001",
    type: "donut",
    name: "Cake",
    ppu: 0.55,
    batters: {
      batter: [
        { id: "1001", type: "Regular" },
        { id: "1002", type: "Chocolate" }
      ]
    },
    topping: [
      { id: "5001", type: "None" },
      { id: "5002", type: "Glazed" }
    ]
  }
];

// Convert and print result
console.log(jsonToPipeDelimited(jsonData));