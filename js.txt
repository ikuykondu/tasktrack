let classData = [
    { id: '1', day: 'Mon', time: '09:00 - 11:00', name: 'เคมี ม.5', topic: 'อะตอมและตารางธาตุ', duration: 2, completed: false },
    { id: '2', day: 'Mon', time: '13:00 - 15:00', name: 'ภาษาอังกฤษ ม.5', topic: 'Advanced Grammar', duration: 2, completed: false },
    { id: '3', day: 'Tue', time: '10:00 - 12:00', name: 'ชีววิทยา ม.5', topic: 'ระบบย่อยอาหาร', duration: 2, completed: false },
    { id: '4', day: 'Tue', time: '13:00 - 15:00', name: 'คอมพิวเตอร์ ม.5', topic: 'Python Basic', duration: 2, completed: false },
    { id: '5', day: 'Wed', time: '09:00 - 11:00', name: 'คณิตศาสตร์ ม.5', topic: 'แคลคูลัสเบื้องต้น', duration: 2, completed: false },
    { id: '6', day: 'Wed', time: '13:00 - 15:00', name: 'ฟิสิกส์ ม.5', topic: 'กลศาสตร์', duration: 2, completed: false },
    { id: '7', day: 'Thu', time: '10:00 - 12:00', name: 'ประวัติศาสตร์ ม.5', topic: 'สงครามเย็น', duration: 2, completed: false },
    { id: '8', day: 'Thu', time: '13:00 - 15:00', name: 'สถิติ ม.5', topic: 'Probability', duration: 2, completed: false },
    { id: '9', day: 'Fri', time: '09:00 - 11:00', name: 'ภาษาไทย ม.5', topic: 'การใช้ภาษา', duration: 2, completed: false },
    { id: '10', day: 'Fri', time: '13:00 - 15:00', name: 'สังคมศึกษา ม.5', topic: 'หน้าที่พลเมือง', duration: 2, completed: false },
    { id: '11', day: 'Sat', time: '10:00 - 12:00', name: 'กิจกรรมพัฒนาผู้เรียน ม.5', topic: 'Club & Activity', duration: 2, completed: false },
    { id: '12', day: 'Sun', time: '13:00 - 15:00', name: 'ทบทวนบทเรียน ม.5', topic: 'เคลียร์การบ้านสัปดาห์นี้', duration: 2, completed: false }
];

let scriptUrl = localStorage.getItem('google_apps_script_url') || '';

document.addEventListener('DOMContentLoaded', () => {
    if (scriptUrl) {
        document.getElementById('scriptUrlInput').value = scriptUrl;
        loadDataFromGoogleSheets();
    } else {
        renderTimetable();
        updateSummary();
    }

    document.getElementById('saveUrlBtn').addEventListener('click', () => {
        scriptUrl = document.getElementById('scriptUrlInput').value.trim();
        localStorage.setItem('google_apps_script_url', scriptUrl);
        if (scriptUrl) {
            loadDataFromGoogleSheets();
        } else {
            alert('กรุณากรอก URL ของ Google Apps Script Web App');
        }
    });

    document.getElementById('classForm').addEventListener('submit', (e) => {
        e.preventDefault();
        saveClassData();
    });

    document.getElementById('cancelEditBtn').addEventListener('click', () => {
        resetForm();
    });
});

function loadDataFromGoogleSheets() {
    if (!scriptUrl) return;
    document.getElementById('statusText').innerText = 'กำลังซิงค์ข้อมูลจาก Google Sheet...';
    
    fetch(scriptUrl + '?action=get')
        .then(response => response.json())
        .then(data => {
            if (data && Array.isArray(data.classes) && data.classes.length > 0) {
                classData = data.classes;
            }
            renderTimetable();
            updateSummary();
            document.getElementById('statusText').innerText = 'เชื่อมต่อ Google Sheet สำเร็จ (ม.5)';
        })
        .catch(err => {
            console.error('Error loading data:', err);
            document.getElementById('statusText').innerText = 'ใช้ข้อมูลออฟไลน์ (เชื่อมต่อชีทไม่สำเร็จ)';
            renderTimetable();
            updateSummary();
        });
}

function syncToGoogleSheets() {
    if (!scriptUrl) return;
    document.getElementById('statusText').innerText = 'กำลังบันทึกลง Google Sheet...';

    fetch(scriptUrl, {
        method: 'POST',
        mode: 'no-cors',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ action: 'save', classes: classData })
    }).then(() => {
        document.getElementById('statusText').innerText = 'บันทึกลง Google Sheet แล้ว';
    }).catch(err => {
        console.error('Error saving data:', err);
        document.getElementById('statusText').innerText = 'บันทึกออฟไลน์ (เชื่อมต่อชีทไม่สำเร็จ)';
    });
}

function renderTimetable() {
    const tbody = document.getElementById('timetableBody');
    tbody.innerHTML = '';

    const timeSlots = [
        '08:00 - 10:00',
        '10:00 - 12:00',
        '12:00 - 13:00',
        '13:00 - 15:00',
        '15:00 - 17:00',
        '17:00 - 19:00'
    ];

    const days = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];

    timeSlots.forEach(slot => {
        const tr = document.createElement('tr');
        
        const tdTime = document.createElement('td');
        tdTime.className = 'time-cell';
        tdTime.innerText = slot;
        tr.appendChild(tdTime);

        days.forEach(day => {
            const td = document.createElement('td');
            const matchedClasses = classData.filter(c => c.day === day && c.time.trim() === slot);

            if (matchedClasses.length > 0) {
                matchedClasses.forEach(cls => {
                    const card = document.createElement('div');
                    card.className = `class-card ${cls.completed ? 'completed' : ''}`;
                    card.innerHTML = `
                        <div class="card-actions">
                            <button class="edit-btn" onclick="editClass('${cls.id}')" title="แก้ไข">✏️</button>
                            <button class="del-btn" onclick="deleteClass('${cls.id}')" title="ลบ">🗑️</button>
                        </div>
                        <label>
                            <input type="checkbox" ${cls.completed ? 'checked' : ''} onchange="toggleComplete('${cls.id}')">
                            ${cls.name}
                        </label>
                        <div class="topic">${cls.topic || ''}</div>
                        <div class="dur">⏱ ${cls.duration} ชม.</div>
                    `;
                    td.appendChild(card);
                });
            } else {
                td.innerHTML = `<span class="empty-cell">-</span>`;
            }
            tr.appendChild(td);
        });

        tbody.appendChild(tr);
    });

    updateSummary();
}

function toggleComplete(id) {
    const cls = classData.find(c => c.id === id);
    if (cls) {
        cls.completed = !cls.completed;
        renderTimetable();
        syncToGoogleSheets();
    }
}

function saveClassData() {
    const editId = document.getElementById('editClassId').value;
    const day = document.getElementById('classDay').value;
    const time = document.getElementById('classTime').value;
    const name = document.getElementById('className').value;
    const topic = document.getElementById('classTopic').value;
    const duration = parseFloat(document.getElementById('classDuration').value) || 1;

    if (editId) {
        const cls = classData.find(c => c.id === editId);
        if (cls) {
            cls.day = day;
            cls.time = time;
            cls.name = name;
            cls.topic = topic;
            cls.duration = duration;
        }
    } else {
        const newCls = {
            id: Date.now().toString(),
            day,
            time,
            name,
            topic,
            duration,
            completed: false
        };
        classData.push(newCls);
    }

    resetForm();
    renderTimetable();
    syncToGoogleSheets();
}

function editClass(id) {
    const cls = classData.find(c => c.id === id);
    if (cls) {
        document.getElementById('editClassId').value = cls.id;
        document.getElementById('classDay').value = cls.day;
        document.getElementById('classTime').value = cls.time;
        document.getElementById('className').value = cls.name;
        document.getElementById('classTopic').value = cls.topic;
        document.getElementById('classDuration').value = cls.duration;
        
        document.getElementById('formTitle').innerText = '✏️ แก้ไขวิชาเรียน ม.5';
        document.getElementById('submitBtn').innerText = 'บันทึกการแก้ไข';
        document.getElementById('cancelEditBtn').style.display = 'block';
    }
}

function deleteClass(id) {
    if (confirm('คุณต้องการลบวิชานี้ใช่หรือไม่?')) {
        classData = classData.filter(c => c.id !== id);
        renderTimetable();
        syncToGoogleSheets();
    }
}

function resetForm() {
    document.getElementById('classForm').reset();
    document.getElementById('editClassId').value = '';
    document.getElementById('formTitle').innerText = '➕ เพิ่มวิชา / กิจกรรม ม.5';
    document.getElementById('submitBtn').innerText = 'บันทึกวิชาเรียน';
    document.getElementById('cancelEditBtn').style.display = 'none';
}

function updateSummary() {
    const target = 24;
    const completed = classData
        .filter(c => c.completed)
        .reduce((sum, c) => sum + (c.duration || 0), 0);

    document.getElementById('completedHours').innerText = completed.toFixed(1) + ' ชม.';
    document.getElementById('targetHours').innerText = target + ' ชม.';
    
    let percent = (completed / target) * 100;
    if (percent > 100) percent = 100;
    document.getElementById('progressBar').style.width = percent + '%';
}