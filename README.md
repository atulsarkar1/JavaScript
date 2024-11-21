import * as fs from 'fs';

function findJSONExamples(directory: string) {
  fs.readdirSync(directory, { withFileTypes: true }).forEach(dirent => {
    const fullPath = `${directory}/${dirent.name}`;

    if (dirent.isDirectory()) {
      findJSONExamples(fullPath);
    } else if (dirent.isFile() && dirent.name.endsWith('.feature')) {
      const fileContent = fs.readFileSync(fullPath, 'utf8');

      const regex = /Examples:\s*\|(.*)\|/g;
      let match;

      while ((match = regex.exec(fileContent)) !== null) {
        const exampleLine = match[1];
        const exampleValues = exampleLine.split('|');

        for (const value of exampleValues) {
          if (value.trim().endsWith('.json')) {
            console.log(`JSON Example found in ${fullPath}: ${value}`);
          }
        }
      }
    }
  });
}

// Replace 'your_project_directory' with the actual path
findJSONExamples('your_project_directory');
