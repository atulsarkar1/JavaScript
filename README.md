import { test, expect } from "@playwright/test";
import { sequelize } from "../db-config";
import { executeSelectQuery } from "../db-utils";

test.beforeAll(async () => {
  await sequelize.authenticate();
});

test.afterAll(async () => {
  await sequelize.close();
});

test("Validate SELECT query results from Oracle DB", async () => {
  const query = "SELECT column1, column2 FROM your_table WHERE column1 = :value";
  const params = ["expected_value"]; // Replace with your actual value

  const results = await executeSelectQuery(query, params);

  console.log("Query Results: ", results);

  // Perform validations
  expect(results.length).toBeGreaterThan(0); // Ensure data exists
  expect(results[0]).toHaveProperty("COLUMN1", "expected_value"); // Validate specific column value
});