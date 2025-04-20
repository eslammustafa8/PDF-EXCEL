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
// ✅ المتطلبات: npm install exceljs jspdf jspdf-autotable file-saver

import React from 'react';
import { saveAs } from 'file-saver';
import ExcelJS from 'exceljs';
import jsPDF from 'jspdf';
import autoTable from 'jspdf-autotable';
import scheduleData from './test_schedule.json'; // ← تم التعديل هنا لقراءة ملف JSON الذي رفعته

const timeSlots = [
  '8 - 9', '9 - 10', '10 - 11', '11 - 12', '12 - 1', '1 - 2', '2 - 3', '3 - 4', '4 - 5', '5 - 6'
];

const days = ['السبت', 'الأحد', 'الاثنين', 'الثلاثاء', 'الأربعاء', 'الخميس'];

const colors = {
  'Structure Programming': 'FF9900',
  'Electronics': '999999',
  'Mathematics in computer': '00B050',
  'introduction to IT': '00B0F0',
  'قاعة أو معمل': 'CCCCCC',
  'default': 'FFFFFF'
};

const getColor = (courseName) => {
  for (let key in colors) {
    if (courseName.includes(key)) return colors[key];
  }
  return colors['default'];
};

const formatCellText = (session) => {
  return `${session.course}\nد/ ${session.staff}\n${session.room}`;
};

const ScheduleExportStyled = () => {
  const buildGrid = () => {
    const grid = {};
    for (let day of days) {
      grid[day] = {};
      for (let time of timeSlots) {
        grid[day][time] = null;
      }
    }
    scheduleData.forEach((session) => {
      const { day, time_slot } = session;
      grid[day][time_slot] = session;
    });
    return grid;
  };

  const exportExcel = async () => {
    const workbook = new ExcelJS.Workbook();
    const sheet = workbook.addWorksheet('الجدول الدراسي');

    sheet.views = [{ rightToLeft: true }];

    sheet.mergeCells('A1', 'B1');
    sheet.getCell('A1').value = 'الجدول الدراسي';
    sheet.getCell('A1').alignment = { vertical: 'middle', horizontal: 'center' };
    sheet.getRow(1).height = 30;

    const headerRow = ['الفترة الزمنية', ...days];
    sheet.addRow(headerRow);

    const grid = buildGrid();

    timeSlots.forEach((slot) => {
      const row = [slot];
      for (let day of days) {
        const cellData = grid[day][slot];
        if (cellData) {
          row.push(formatCellText(cellData));
        } else {
          row.push('');
        }
      }
      sheet.addRow(row);
    });

    sheet.columns.forEach((col, i) => {
      col.width = i === 0 ? 12 : 30;
    });

    sheet.eachRow((row, rowNumber) => {
      row.eachCell((cell, colNumber) => {
        cell.alignment = { vertical: 'middle', horizontal: 'center', wrapText: true };
        if (rowNumber > 2 && colNumber > 1) {
          const session = grid[days[colNumber - 2]][timeSlots[rowNumber - 3]];
          if (session) {
            const color = getColor(session.course);
            cell.fill = {
              type: 'pattern',
              pattern: 'solid',
              fgColor: { argb: color },
            };
            cell.border = {
              top: { style: 'thin' },
              left: { style: 'thin' },
              bottom: { style: 'thin' },
              right: { style: 'thin' },
            };
          }
        }
      });
    });

    const buffer = await workbook.xlsx.writeBuffer();
    saveAs(new Blob([buffer]), 'schedule.xlsx');
  };

  const exportPDF = () => {
    const doc = new jsPDF({ orientation: 'landscape' });

    const grid = buildGrid();
    const body = timeSlots.map((slot) => {
      const row = [slot];
      for (let day of days) {
        const cellData = grid[day][slot];
        if (cellData) {
          row.push(formatCellText(cellData));
        } else {
          row.push('');
        }
      }
      return row;
    });

    autoTable(doc, {
      head: [['الفترة الزمنية', ...days]],
      body: body,
      styles: {
        font: 'helvetica',
        fontSize: 10,
        halign: 'center',
        valign: 'middle',
        cellWidth: 'wrap',
        cellPadding: 3,
      },
      headStyles: { fillColor: [41, 128, 185], textColor: 255 },
      theme: 'grid',
      margin: { top: 20 },
    });

    doc.save('schedule.pdf');
  };

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">جدول المستوى الأول</h2>
      <div className="flex gap-4">
        <button onClick={exportExcel} className="bg-green-600 text-white px-4 py-2 rounded">تصدير Excel</button>
        <button onClick={exportPDF} className="bg-red-600 text-white px-4 py-2 rounded">تصدير PDF</button>
      </div>
    </div>
  );
};

export default ScheduleExportStyled;

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

