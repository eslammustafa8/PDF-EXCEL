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
iimport React, { useEffect, useState } from 'react';
import jsPDF from 'jspdf';
import 'jspdf-autotable';
import ExcelJS from 'exceljs';
import { saveAs } from 'file-saver';
import scheduleData from './test_schedule.json';

const days = ['sunday', 'monday', 'tuesday', 'wednesday', 'thursday'];
const timeSlots = ['09:00-11:00', '11:00-13:00', '13:00-15:00', '15:00-17:00'];

const ScheduleExport = () => {
  const [tableData, setTableData] = useState({});

  useEffect(() => {
    const result = {};
    days.forEach(day => {
      result[day] = {};
      timeSlots.forEach(slot => {
        result[day][slot] = [];
      });
    });

    scheduleData.schedule.forEach(session => {
      const day = session.time_slot.day.toLowerCase();
      const start = session.time_slot.start_time;
      const end = session.time_slot.end_time;
      const slotKey = `${start}-${end}`;

      if (result[day] && result[day][slotKey]) {
        const course = session.course.name;
        const teacher = session.staff.name;
        const room = session.hall?.name || session.room?.name || '-';
        const type = session.session_type === 'lab' ? 'معمل' : 'محاضرة';

        result[day][slotKey].push(`${course} (${type})\n${teacher} - ${room}`);
      }
    });

    setTableData(result);
  }, []);

  const exportPDF = () => {
    const doc = new jsPDF();
    doc.setFontSize(14);
    doc.text('الجدول الدراسي', 80, 10);

    const head = ['اليوم', ...timeSlots];
    const body = days.map(day => {
      const row = [day];
      timeSlots.forEach(slot => {
        const cell = tableData[day]?.[slot]?.join('\n') || '';
        row.push(cell);
      });
      return row;
    });

    doc.autoTable({
      startY: 20,
      head: [head],
      body,
      styles: { halign: 'center', fontSize: 8 },
      headStyles: { fillColor: [255, 102, 0], textColor: [255, 255, 255] },
      alternateRowStyles: { fillColor: [245, 245, 245] },
    });

    doc.save('schedule.pdf');
  };

  const exportExcel = async () => {
    const workbook = new ExcelJS.Workbook();
    const sheet = workbook.addWorksheet('Schedule');

    const header = ['اليوم', ...timeSlots];
    sheet.addRow(header);

    days.forEach(day => {
      const row = [day];
      timeSlots.forEach(slot => {
        row.push(tableData[day]?.[slot]?.join('\n') || '');
      });
      sheet.addRow(row);
    });

    sheet.columns.forEach(col => {
      col.width = 30;
    });

    sheet.getRow(1).font = { bold: true, color: { argb: 'FFFFFFFF' } };
    sheet.getRow(1).fill = {
      type: 'pattern',
      pattern: 'solid',
      fgColor: { argb: 'FFFF6600' }
    };

    const buffer = await workbook.xlsx.writeBuffer();
    const blob = new Blob([buffer], {
      type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
    });
    saveAs(blob, 'schedule.xlsx');
  };

  return (
    <div style={{ padding: 20 }}>
      <h2>📚 الجدول الدراسي</h2>
      <table border="1" cellPadding="5" style={{ borderCollapse: 'collapse', width: '100%' }}>
        <thead>
          <tr>
            <th>اليوم</th>
            {timeSlots.map((slot, i) => <th key={i}>{slot}</th>)}
          </tr>
        </thead>
        <tbody>
          {days.map((day, i) => (
            <tr key={i}>
              <td><strong>{day}</strong></td>
              {timeSlots.map((slot, j) => (
                <td key={j} style={{ whiteSpace: 'pre-wrap', textAlign: 'center' }}>
                  {tableData[day]?.[slot]?.join('\n') || '-'}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>

      <div style={{ marginTop: 20 }}>
        <button onClick={exportPDF}>📄 تصدير PDF</button>
        <button onClick={exportExcel} style={{ marginLeft: 10 }}>📊 تصدير Excel</button>
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

