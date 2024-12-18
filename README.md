import { Sequelize } from "sequelize";

export const sequelize = new Sequelize("your_database_name", "your_username", "your_password", {
  host: "your_host", // e.g., "localhost"
  dialect: "oracle",
  dialectOptions: {
    connectString: "your_host:your_port/your_service_name", // e.g., "localhost:1521/xe"
  },
  logging: false, // Disable SQL query logging for cleaner output
});

export async function testConnection(): Promise<void> {
  try {
    await sequelize.authenticate();
    console.log("Connection has been established successfully.");
  } catch (error) {
    console.error("Unable to connect to the database:", error);
  }
}