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
        const worksheet = XLSX.utils.aoa_to_sheet([["Step", "Screenshot (Base64)"]]); // Store Base64 data
        XLSX.utils.book_append_sheet(workbook, worksheet, "Results");
        XLSX.writeFile(workbook, filePath);
    }
    return filePath;
};

/**
 * Splits the Base64 string into chunks to avoid exceeding Excel's cell size limit (32767 characters).
 * @param base64 Base64 string representing the image.
 */
const splitBase64 = (base64: string): string[] => {
    const chunkSize = 32000; // Ensure it stays below Excel's limit
    const chunks = [];
    let i = 0;
    while (i < base64.length) {
        chunks.push(base64.substring(i, i + chunkSize));
        i += chunkSize;
    }
    return chunks;
};

/**
 * Adds the screenshot as Base64 chunks to the Excel file for the given scenario and step.
 * @param scenarioName Name of the scenario.
 * @param stepName Name of the step.
 * @param screenshotPath Path to the screenshot file.
 */
export const addScreenshotToExcel = (scenarioName: string, stepName: string, screenshotPath: string) => {
    const filePath = createExcelForScenario(scenarioName);
    const workbook = XLSX.readFile(filePath);
    const worksheet = workbook.Sheets["Results"];
    const lastRow = XLSX.utils.sheet_to_json(worksheet, { header: 1 }).length;

    const screenshotBase64 = fs.readFileSync(screenshotPath, { encoding: 'base64' });
    const base64Chunks = splitBase64(screenshotBase64); // Split the Base64 string into chunks

    // Add each chunk in a new row to avoid exceeding Excel's cell limit
    base64Chunks.forEach((chunk, index) => {
        const newRow = [[`${stepName} (Part ${index + 1})`, chunk]];
        XLSX.utils.sheet_add_aoa(worksheet, newRow, { origin: `A${lastRow + index + 1}` });
    });

    XLSX.writeFile(workbook, filePath);
};

/**
 * Returns the path of the current run directory.
 */
export const getRunDirectory = (): string => {
    return createRunDirectory();
};