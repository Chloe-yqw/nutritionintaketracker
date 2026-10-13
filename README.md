<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PHRM 801: Calcium Intake Tracker</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, sans-serif; line-height: 1.6; padding: 20px; max-width: 700px; margin: auto; background-color: #f4f7f9; }
        .card { background: white; padding: 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.08); margin-bottom: 20px; }
        h2, h3 { color: #0056b3; margin-top: 0; }
        .instruction { background: #e7f3ff; border-left: 5px solid #0056b3; padding: 15px; font-size: 0.9em; margin-bottom: 20px; border-radius: 0 8px 8px 0; }
        .input-row { display: flex; gap: 10px; margin-bottom: 15px; flex-wrap: wrap; align-items: flex-end; }
        .input-group { flex: 1; min-width: 140px; }
        label { display: block; font-weight: 600; font-size: 0.85em; margin-bottom: 5px; color: #555; }
        input, select { width: 100%; padding: 10px; border: 1px solid #ced4da; border-radius: 6px; font-size: 14px; }
        button.add-btn { background-color: #007bff; color: white; border: none; padding: 12px; border-radius: 6px; cursor: pointer; width: 100%; font-weight: bold; }
        .summary-box { background: #fff3cd; border: 1px solid #ffeeba; padding: 20px; border-radius: 10px; text-align: center; margin-top: 20px; }
        .avg-value { font-size: 2em; color: #856404; font-weight: bold; display: block; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; background: #fff; }
        th, td { padding: 12px; border-bottom: 1px solid #dee2e6; text-align: left; }
        th { background: #f8f9fa; color: #666; font-weight: 600; }
        .checkbox-cell { width: 40px; text-align: center; }
        .del-btn { color: #dc3545; cursor: pointer; border: 1px solid #dc3545; background: none; padding: 2px 8px; border-radius: 4px; font-size: 12px; }
        .del-btn:hover { background: #dc3545; color: white; }
        .divider { height: 2px; background: #eee; margin: 30px 0; }
    </style>
</head>
<body>

<div class="card">
    <h2>Calcium Intake Tracker</h2>
    <div class="instruction">
        <strong>Instructions:</strong> For the next few days, record the amount of calcium you consume from various food, drinks, and snacks[cite: 1, 2]. <br><br>
        <strong>Calculation Rule:</strong> To find the mg of calcium, simply <strong>add a zero</strong> to the % daily value on the nutritional label (e.g., 20% = 200 mg)[cite: 6, 7].
    </div>

    <div class="input-row">
        <div class="input-group"><label>Date</label><input type="date" id="date"></div>
        <div class="input-group">
            <label>Meal/Snack</label>
            <select id="meal">
                <option>Breakfast</option><option>Morning Snack</option><option>Lunch</option>
                <option>Afternoon Snack</option><option>Dinner</option><option>Bedtime Snack</option><option>Other</option>
            </select>
        </div>
        <div class="input-group"><label>Calcium (mg)</label><input type="number" id="mg" placeholder="0"></div>
    </div>
    <button class="add-btn" onclick="addEntry()">Add Entry to Diary</button>
</div>

<div class="card">
    <h3>Diary Summary</h3>
    <p style="font-size: 0.85em; color: #666;">Track your intake daily. On the day of class, use the <strong>"Include in Report"</strong> checkboxes to select your entries for calculation.</p>
    
    <table>
        <thead>
            <tr>
                <th class="checkbox-cell">Include</th>
                <th>Date</th>
                <th>Meal</th>
                <th>Amount</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody id="logBody"></tbody>
    </table>

    <div class="divider"></div>

    <div class="summary-box">
        <span style="font-size: 0.9em; text-transform: uppercase; letter-spacing: 1px; color: #856404;">Class Report Calculation</span>
        <span class="avg-value"><span id="avgDisplay">0</span> mg/day</span>
        <p style="font-size: 0.85em; margin-bottom: 0;">(Calculated as: Total of selected entries ÷ 3 days for the class average)</p>
    </div>
    
    <button onclick="clearAll()" style="background:#6c757d; color:white; border:none; padding:8px; margin-top:20px; border-radius:4px; cursor:pointer; font-size:0.8em;">Clear Entire Diary</button>
</div>

<script>
    // LocalStorage for persistence
    let logs = JSON.parse(localStorage.getItem('calcium_v5')) || [];

    function updateUI() {
        const body = document.getElementById('logBody');
        body.innerHTML = '';
        let selectedTotal = 0;

        logs.forEach((item, index) => {
            if (item.selected) selectedTotal += parseInt(item.mg);
            
            body.innerHTML += `
                <tr>
                    <td class="checkbox-cell"><input type="checkbox" ${item.selected ? 'checked' : ''} onchange="toggleSelect(${index})"></td>
                    <td>${item.date}</td>
                    <td>${item.meal}</td>
                    <td>${item.mg} mg</td>
                    <td><button class="del-btn" onclick="deleteItem(${index})">Delete</button></td>
                </tr>`;
        });

        // Per Dr. Chen's requirement for a 3-day average report
        const average = Math.round(selectedTotal / 3);
        document.getElementById('avgDisplay').innerText = average;
        
        localStorage.setItem('calcium_v5', JSON.stringify(logs));
    }

    function addEntry() {
        const d = document.getElementById('date').value;
        const m = document.getElementById('mg').value;
        if(!d || !m) return alert("Please enter both date and amount.");
        
        logs.push({date: d, meal: document.getElementById('meal').value, mg: m, selected: true});
        updateUI();
        document.getElementById('mg').value = '';
    }

    function toggleSelect(index) {
        logs[index].selected = !logs[index].selected;
        updateUI();
    }

    function deleteItem(index) {
        logs.splice(index, 1);
        updateUI();
    }

    function clearAll() {
        if(confirm("Are you sure you want to delete all entries?")) {
            logs = [];
            updateUI();
        }
    }

    document.getElementById('date').valueAsDate = new Date();
    updateUI();
</script>
</body>
</html>
