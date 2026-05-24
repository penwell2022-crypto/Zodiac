<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>แบบประเมินคำนวณธาตุและจักรราศีกำเนิด/ปฏิสนธิ</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Sarabun', sans-serif;
        }
        body {
            background-color: #f0f4f8;
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .container {
            background-color: #ffffff;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.15);
            width: 100%;
            max-width: 600px;
        }
        h2 {
            color: #2c3e50;
            text-align: center;
            margin-bottom: 25px;
            border-bottom: 3px solid #dbb234;
            padding-bottom: 10px;
        }
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            font-weight: bold;
            margin-bottom: 8px;
            color: #34495e;
        }
        input[type="date"], input[type="time"], input[type="number"] {
            width: 100%;
            padding: 12px;
            border: 1px solid #ccc;
            border-radius: 6px;
            font-size: 16px;
        }
        .radio-group {
            display: flex;
            gap: 20px;
            margin-top: 5px;
        }
        .radio-item {
            display: flex;
            align-items: center;
            gap: 5px;
        }
        .row {
            display: flex;
            gap: 15px;
        }
        .col {
            flex: 1;
        }
        button {
            width: 100%;
            background-color: #dbb234;
            color: #2c3e50;
            border: none;
            padding: 14px;
            font-size: 18px;
            font-weight: bold;
            border-radius: 6px;
            cursor: pointer;
            transition: background-color 0.3s;
            margin-top: 10px;
        }
        button:hover {
            background-color: #c29d2b;
        }
        .result-box {
            margin-top: 25px;
            padding: 20px;
            background-color: #fbf7ec;
            border-left: 5px solid #dbb234;
            border-radius: 4px;
            display: none;
        }
        .result-box h3 {
            margin-top: 0;
            color: #2c3e50;
            border-bottom: 1px solid #ddd;
            padding-bottom: 5px;
        }
        .result-item {
            margin-bottom: 12px;
            font-size: 16px;
            color: #333;
            line-height: 1.5;
        }
        .highlight {
            font-weight: bold;
            color: #b37d14;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>แบบประเมินคำนวณจักรราศีและธาตุ</h2>
    
    <form id="quizForm">
        <div class="form-group">
            <label for="birthDate">A) วัน เดือน ปี เกิด (ค.ศ.):</label>
            <input type="date" id="birthDate" required>
        </div>

        <div class="form-group">
            <label>B) คุณเกิดก่อนเวลา 06:00 น. ใช่หรือไม่?</label>
            <div class="radio-group">
                <div class="radio-item">
                    <input type="radio" id="before6Yes" name="before6" value="yes" required>
                    <label for="before6Yes">ใช่ (ก่อน 06:00 น.)</label>
                </div>
                <div class="radio-item">
                    <input type="radio" id="before6No" name="before6" value="no" required>
                    <label for="before6No">ไม่ใช่ / ไม่แน่ใจ</label>
                </div>
            </div>
        </div>

        <div class="form-group">
            <label for="birthTime">C) เวลาเกิดของคุณ (โดยประมาณ):</label>
            <input type="time" id="birthTime">
        </div>

        <div class="form-group">
            <label>D) ระยะเวลาที่อยู่ในครรภ์มารดา:</label>
            <div class="row">
                <div class="col">
                    <label style="font-size:12px; font-weight:normal;">จำนวนเดือน:</label>
                    <input type="number" id="gestationMonths" placeholder="เช่น 9" min="0" max="12" required>
                </div>
                <div class="col">
                    <label style="font-size:12px; font-weight:normal;">จำนวนวัน (ถ้าไม่ระบุจะเท่ากับ 0):</label>
                    <input type="number" id="gestationDays" placeholder="เช่น 15" min="0" max="31">
                </div>
            </div>
        </div>

        <button type="button" onclick="processQuiz()">วิเคราะห์ผลลัพธ์</button>
    </form>

    <div class="result-box" id="resultBox">
        <h3>ผลการวิเคราะห์ทางโหราศาสตร์</h3>
        <div class="result-item" id="resBirthDate"></div>
        <div class="result-item" id="resLunarBirth"></div>
        <div class="result-item" id="resZodiacBirth"></div>
        <div class="result-item" id="resGestationDiff"></div>
        <div class="result-item" id="resZodiacConception"></div>
    </div>
</div>

<script>
const ZODIAC_RULES = [
    { sign: "ราศีมังกร", element: "ธาตุดิน", startMonth: 1, endMonth: 2 },
    { sign: "ราศีกุมภ์", element: "ธาตุดิน", startMonth: 1, endMonth: 2 },
    { sign: "ราซีมีน", element: "ธาตุดิน", startMonth: 2, endMonth: 3 },
    { sign: "ราศีเมษ", element: "ธาตุไฟ", startMonth: 3, endMonth: 4 },
    { sign: "ราซีพฤกษก", element: "ธาตุไฟ", startMonth: 4, endMonth: 5 },
    { sign: "ราศีเมถุน", element: "ธาตุไฟ", startMonth: 5, endMonth: 6 },
    { sign: "ราศีกรกฎ", element: "ธาตุลม", startMonth: 6, endMonth: 7 },
    { sign: "ราศีสิงห์", element: "ธาตุลม", startMonth: 7, endMonth: 8 },
    { sign: "ราศีกัน์", element: "ธาตุลม", startMonth: 8, endMonth: 9 },
    { sign: "ราศีตุล", element: "ธาตุน้ำ", startMonth: 9, endMonth: 10 },
    { sign: "ราศีพิจิก", element: "ธาตุน้ำ", startMonth: 10, endMonth: 11 },
    { sign: "ราศีธนู", element: "ธาตุน้ำ", startMonth: 11, endMonth: 12 }
];

const ZODIAC_ORDER = [
    "ราศีมังกร", "ราศีกุมภ์", "ราซีมีน", "ราศีเมษ", "ราซีพฤกษก", "ราศีเมถุน",
    "ราศีกรกฎ", "ราศีสิงห์", "ราศีกัน์", "ราศีตุล", "ราศีพิจิก", "ราศีธนู"
];

const ELEMENT_MAP = {
    "ราศีมังกร": "ธาตุดิน", "ราศีกุมภ์": "ธาตุดิน", "ราซีมีน": "ธาตุดิน",
    "ราศีเมษ": "ธาตุไฟ", "ราซีพฤกษก": "ธาตุไฟ", "ราศีเมถุน": "ธาตุไฟ",
    "ราศีกรกฎ": "ธาตุลม", "ราศีสิงห์": "ธาตุลม", "ราศีกัน์": "ธาตุลม",
    "ราศีตุล": "ธาตุน้ำ", "ราศีพิจิก": "ธาตุน้ำ", "ราศีธนู": "ธาตุน้ำ"
};

function getThaiLunarDate(date) {
    const epoch = new Date(1970, 0, 1);
    const diffTime = Math.abs(date - epoch);
    const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
    
    const lunarCycle = 29.53059;
    const currentLunarDay = Math.floor((diffDays % lunarCycle));
    
    let phase = "";
    let kham = 0;
    if (currentLunarDay < 15) {
        phase = "ขึ้น";
        kham = currentLunarDay + 1;
    } else {
        phase = "แรม";
        kham = (currentLunarDay - 14);
    }
    
    let solarMonth = date.getMonth() + 1; 
    let thaiMonth = (solarMonth + 1) % 12;
    if (thaiMonth === 0) thaiMonth = 12;
    
    return { phase: phase, kham: kham, month: thaiMonth };
}

function processQuiz() {
    const birthDateInput = document.getElementById('birthDate').value;
    const before6 = document.querySelector('input[name="before6"]:checked')?.value;
    let gMonths = parseInt(document.getElementById('gestationMonths').value) || 0;
    let gDays = parseInt(document.getElementById('gestationDays').value) || 0;

    if (!birthDateInput || !before6) {
        alert("กรุณาระบุวันเกิด และเลือกเงื่อนไขเวลาเกิดก่อน 06:00 น.");
        return;
    }

    let birthDate = new Date(birthDateInput);
    if (before6 === 'yes') {
        birthDate.setDate(birthDate.getDate() - 1);
    }

    const lunarInfo = getThaiLunarDate(birthDate);
    
    let birthZodiac = "ราศีเมษ";
    let birthElement = "ธาตุไฟ";
    
    for (let rule of ZODIAC_RULES) {
        if (lunarInfo.month === rule.startMonth || lunarInfo.month === rule.endMonth) {
            birthZodiac = rule.sign;
            birthElement = rule.element;
            break;
        }
    }

    let diffMonths = 12 - gMonths;
    let currentIdx = ZODIAC_ORDER.indexOf(birthZodiac);
    let conceptionIdx = (currentIdx + diffMonths) % 12;
    let conceptionZodiac = ZODIAC_ORDER[conceptionIdx];
    let conceptionElement = ELEMENT_MAP[conceptionZodiac];

    document.getElementById('resultBox').style.display = 'block';
    document.getElementById('resBirthDate').innerHTML = `📅 วันเกิดทางโหราศาสตร์: <span class="highlight">${birthDate.toLocaleDateString('th-TH', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })}</span>`;
    document.getElementById('resLunarBirth').innerHTML = `🌙 คำนวณวันจันทรคติไทย: <span class="highlight">${lunarInfo.phase} ${lunarInfo.kham} ค่ำ เดือน ${lunarInfo.month}</span>`;
    document.getElementById('resZodiacBirth').innerHTML = `♈ จักรราศีและธาตูกำเนิด: <span class="highlight">${birthZodiac} (${birthElement})</span>`;
    document.getElementById('resGestationDiff').innerHTML = `⏳ ส่วนต่างเวลาปฏิสนธิ (12 เดือน - ${gMonths} เดือน): <span class="highlight">นับไปข้างหน้า ${diffMonths} เดือน ${gDays} วัน</span>`;
    document.getElementById('resZodiacConception').innerHTML = `🧬 จักรราศีและธาตุปฏิสนธิ: <span class="highlight">${conceptionZodiac} (${conceptionElement})</span>`;
}
</script>
</body>
</html>
