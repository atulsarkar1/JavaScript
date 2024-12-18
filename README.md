import { sequelize } from "./db-config";

export async function executeSelectQuery(query: string, replacements: any[] = []): Promise<any[]> {
  try {
    const [results] = await sequelize.query(query, {
      replacements, // Use for parameterized queries
      type: sequelize.QueryTypes.SELECT, // Ensures SELECT query is enforced
    });
    return results;
  } catch (error) {
    console.error("Error executing SELECT query:", error);
    throw error;
  }
}