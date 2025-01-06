import * as ExcelJS from 'exceljs';
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
        const workbook = new ExcelJS.Workbook();
        const worksheet = workbook.addWorksheet('Results');
        worksheet.columns = [
            { header: 'Step', key: 'step', width: 30 },
            { header: 'Screenshot', key: 'screenshot', width: 50 },
        ];
        workbook.xlsx.writeFile(filePath);
    }
    return filePath;
};

/**
 * Adds the screenshot as an image to the Excel file for the given scenario and step.
 * @param scenarioName Name of the scenario.
 * @param stepName Name of the step.
 * @param screenshotPath Path to the screenshot file.
 */
export const addScreenshotToExcel = async (scenarioName: string, stepName: string, screenshotPath: string) => {
    const filePath = createExcelForScenario(scenarioName);
    const workbook = new ExcelJS.Workbook();
    await workbook.xlsx.readFile(filePath);

    const worksheet = workbook.getWorksheet('Results');
    const lastRow = worksheet.lastRow ? worksheet.lastRow.number : 1;

    // Add step description in the first column
    worksheet.addRow({ step: stepName });

    // Read the screenshot image and add it as an embedded image
    const imageBuffer = fs.readFileSync(screenshotPath);
    const imageId = workbook.addImage({
        buffer: imageBuffer,
        extension: 'png',
    });

    // Add the image to the second column of the row
    worksheet.getRow(lastRow + 1).getCell(2).value = { text: 'Embedded Image' }; // Optional text
    worksheet.getRow(lastRow + 1).getCell(2).style = { alignment: { vertical: 'middle', horizontal: 'center' } };
    worksheet.getRow(lastRow + 1).getCell(2).addImage(imageId, {
        tl: { col: 1, row: lastRow }, // Position image
        ext: { width: 200, height: 150 }, // Set image size
    });

    await workbook.xlsx.writeFile(filePath);
};

/**
 * Returns the path of the current run directory.
 */
export const getRunDirectory = (): string => {
    return createRunDirectory();
};