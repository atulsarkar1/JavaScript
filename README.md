import * as ExcelJS from 'exceljs';
import * as fs from 'fs';

async function getFundData(filePath: string, fundId: string, headerName: string): Promise<string | null> {
    if (!fs.existsSync(filePath)) {
        console.error('File not found:', filePath);
        return null;
    }

    const workbook = new ExcelJS.Workbook();
    await workbook.xlsx.readFile(filePath);
    const worksheet = workbook.worksheets[0]; // Assuming data is in the first sheet

    let headerIndex: number | null = null;
    let result: string | null = null;

    worksheet.eachRow((row, rowNumber) => {
        if (rowNumber === 1) {
            // Find the column index of the header
            row.eachCell((cell, colNumber) => {
                if (cell.value && cell.value.toString().trim().toLowerCase() === headerName.trim().toLowerCase()) {
                    headerIndex = colNumber;
                }
            });
        } else if (headerIndex !== null) {
            // Check if the Fund ID matches
            const fundIdCell = row.getCell(1).value; // Assuming Fund ID is in column 1 (A)
            if (fundIdCell && fundIdCell.toString().trim() === fundId) {
                result = row.getCell(headerIndex).value as string;
            }
        }
    });

    return result;
}

// Usage Example
(async () => {
    const filePath = '/mnt/data/file-6jhVgiFh98EQmy7uBwzrsM'; // Path to uploaded file
    const fundId = 'SGCI2';
    const headerName = 'Fund Name in System';

    const value = await getFundData(filePath, fundId, headerName);
    console.log(`Value for Fund ID ${fundId}:`, value);
})();