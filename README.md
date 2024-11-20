import * as fs from "fs";
import * as path from "path";

interface ClientData {
  client_num: string;
  client_name: string;
}

export const updateFeatureFile = async (existingFeatureFilePath: string, outputFilePath: string) => {
  try {
    // Read the existing feature file
    const existingContent = fs.readFileSync(existingFeatureFilePath, "utf-8");

    // Extract the JSON file path from the "Examples" section
    const jsonPathMatch = existingContent.match(/Examples:\s*\|(.+)\|/);
    if (!jsonPathMatch) {
      throw new Error("JSON file path not found in the Examples section.");
    }

    // Trim whitespace and get the path
    const jsonFilePath = path.resolve(__dirname, jsonPathMatch[1].trim());

    // Read and parse the JSON file
    if (!fs.existsSync(jsonFilePath)) {
      throw new Error(`JSON file not found at path: ${jsonFilePath}`);
    }
    const jsonData = fs.readFileSync(jsonFilePath, "utf-8");
    const clients: ClientData[] = JSON.parse(jsonData);

    // Construct the new "Examples" section
    let examplesSection = "    Examples:\n";
    clients.forEach((client) => {
      examplesSection += `      | ${client.client_num} | ${client.client_name} |\n`;
    });

    // Replace the "Examples" section in the existing feature file
    const updatedContent = existingContent.replace(
      /Examples:\n([\s\S]*?)(\n\s*\n|$)/,
      `${examplesSection}\n`
    );

    // Write the updated content to the output file
    fs.writeFileSync(outputFilePath, updatedContent, "utf-8");
    console.log(`Feature file updated successfully at: ${outputFilePath}`);
  } catch (error) {
    console.error("Error updating feature file:", error.message);
  }
};





import { test } from "@playwright/test";
import { updateFeatureFile } from "./utils/updateFeatureFile";
import * as path from "path";

test.beforeAll(async () => {
  // Define file paths
  const existingFeatureFilePath = path.resolve(__dirname, "features/existing_feature.feature");
  const outputFilePath = path.resolve(__dirname, "features/updated_feature.feature");

  // Update the feature file
  await updateFeatureFile(existingFeatureFilePath, outputFilePath);
});

test("Validate feature file update", async ({}) => {
  console.log("Feature file updated. Proceed with further testing.");
});