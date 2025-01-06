import * as XLSX from 'xlsx';
import * as fs from 'fs';
import * as path from 'path';

let runDirectory: string | null = null;

/**
 * Creates a timestamped directory for the test run.
 * This directory will hold all Excel files and screenshots for the current run.
 */
const createRunDirectory = (): string => {
    if (!runDirectory) {
        const timestamp = new Date().toISOString().replace(/[:.]/g, '-'); // Format timestamp
        runDirectory = path.resolve(__dirname, `../results/run-${timestamp}`);
        if (!fs.existsSync(runDirectory)) {
            fs.mkdirSync(runDirectory, { recursive: true });
        }
    }
    return runDirectory;
};

/**
 * Creates an Excel file for the scenario if it doesn't exist.
 * @param scenarioName Name of the scenario.
 */
export const createExcelForScenario = (scenarioName: string): string => {
    const runDir = createRunDirectory();
    const filePath = path.resolve(runDir, `${scenarioName}.xlsx`);
    if (!fs.existsSync(filePath)) {
        const workbook = XLSX.utils.book_new();
        const worksheet = XLSX.utils.aoa_to_sheet([["Step", "Screenshot Path"]]); // Only store paths
        XLSX.utils.book_append_sheet(workbook, worksheet, "Results");
        XLSX.writeFile(workbook, filePath);
    }
    return filePath;
};

/**
 * Adds a screenshot path to the Excel file for the given scenario and step.
 * @param scenarioName Name of the scenario.
 * @param stepName Name of the step.
 * @param screenshotPath Path to the screenshot file.
 */
export const addScreenshotToExcel = (scenarioName: string, stepName: string, screenshotPath: string) => {
    const filePath = createExcelForScenario(scenarioName);
    const workbook = XLSX.readFile(filePath);
    const worksheet = workbook.Sheets["Results"];
    const lastRow = XLSX.utils.sheet_to_json(worksheet, { header: 1 }).length;

    const newRow = [[stepName, screenshotPath]]; // Store the file path instead of Base64

    XLSX.utils.sheet_add_aoa(worksheet, newRow, { origin: `A${lastRow + 1}` });
    XLSX.writeFile(workbook, filePath);
};

/**
 * Returns the path of the current run directory.
 */
export const getRunDirectory = (): string => {
    return createRunDirectory();
};