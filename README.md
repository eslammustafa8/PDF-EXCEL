# مشروع تصدير الجدول الدراسي 📚

مشروع React لتصدير الجدول الدراسي بصيغتي PDF و Excel مع تنسيق متقدم.

## الخطوات الكاملة + الكود 📝

### 1. إنشاء مشروع React
```bash
npx create-react-app schedule-export
cd schedule-export
```
 ### 2. تثبيت المكتبات المطلوبة

``` bash
npm install jspdf jspdf-autotable exceljs file-saver
```

### 3. ملف ScheduleExport.js (الكود الكامل)

```js
import React from 'react';
import jsPDF from 'jspdf';
import 'jspdf-autotable';
import ExcelJS from 'exceljs';
import { saveAs } from 'file-saver';

const ScheduleExport = () => {
  const scheduleData = [
    ['اليوم', '9-10', '10-11', '11-12'],
    ['الأحد', 'رياضة', 'فيزياء', 'حاسب'],
    ['الإثنين', 'كيمياء', 'حاسب', 'راحة'],
    ['الثلاثاء', 'راحة', 'رياضة', 'حاسب'],
  ];

  // 📄 Export as PDF
  const exportToPDF = () => {
    const doc = new jsPDF();

    doc.setFontSize(16);
    doc.text('الجدول الدراسي', 75, 15);

    doc.autoTable({
      startY: 25,
      head: [scheduleData[0]],
      body: scheduleData.slice(1),
      styles: {
        halign: 'center',
        fontSize: 12,
        cellPadding: 4,
        textColor: [0, 0, 0],
      },
      headStyles: {
        fillColor: [255, 102, 0],
        textColor: [255, 255, 255],
        fontStyle: 'bold',
      },
      alternateRowStyles: {
        fillColor: [240, 240, 240],
      },
    });

    doc.save('schedule.pdf');
  };

  // 📊 Export as Excel (with styles)
  const exportToExcel = async () => {
    const workbook = new ExcelJS.Workbook();
    const worksheet = workbook.addWorksheet('Schedule');

    // Add data + style
    scheduleData.forEach((row, index) => {
      const rowRef = worksheet.addRow(row);
      rowRef.eachCell((cell) => {
        cell.alignment = { horizontal: 'center' };
        cell.border = {
          top: { style: 'thin' },
          bottom: { style: 'thin' },
          left: { style: 'thin' },
          right: { style: 'thin' },
        };
      });

      // Style header
      if (index === 0) {
        rowRef.eachCell((cell) => {
          cell.font = { bold: true, color: { argb: 'FFFFFFFF' } };
          cell.fill = {
            type: 'pattern',
            pattern: 'solid',
            fgColor: { argb: 'FFFF6600' },
          };
        });
      }
    });

    // دمج خلايا مثال (دمج A5 لـ C5)
    worksheet.mergeCells('A5:C5');
    worksheet.getCell('A5').value = 'خلاصة الأسبوع';
    worksheet.getCell('A5').font = { bold: true };
    worksheet.getCell('A5').alignment = { horizontal: 'center' };

    const buffer = await workbook.xlsx.writeBuffer();
    const blob = new Blob([buffer], {
      type:
        'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
    });

    saveAs(blob, 'styled_schedule.xlsx');
  };

  return (
    <div style={{ padding: '20px', fontFamily: 'Arial' }}>
      <h2>📚 الجدول الدراسي</h2>

      <table border="1" cellPadding="10">
        {scheduleData.map((row, rowIndex) => (
          <tr key={rowIndex}>
            {row.map((cell, cellIndex) => (
              <td key={cellIndex} style={{ textAlign: 'center', fontWeight: rowIndex === 0 ? 'bold' : 'normal' }}>
                {cell}
              </td>
            ))}
          </tr>
        ))}
      </table>

      <div style={{ marginTop: '20px' }}>
        <button onClick={exportToPDF}>📄 تصدير كـ PDF</button>
        <button onClick={exportToExcel} style={{ marginLeft: '10px' }}>📊 تصدير كـ Excel</button>
      </div>
    </div>
  );
};

export default ScheduleExport;
```
#####
### 🔁 5) استدعِ الكومبوننت في App.js

```js
import React from 'react';
import ScheduleExport from './ScheduleExport';

function App() {
  return (
    <div className="App">
      <ScheduleExport />
    </div>
  );
}

export default App;
```

