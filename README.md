<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App Bokashinho v2</title>
    <style>
        :root {
            --primary: #4CAF50;
            --secondary: #8BC34A;
            --background: #f4f7f6;
            --dark: #333;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--background);
            color: var(--dark);
            margin: 0;
            padding: 15px;
            display: flex;
            justify-content: center;
        }

        .app-container {
            max-width: 450px;
            width: 100%;
            background: white;
            border-radius: 20px;
            padding: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            box-sizing: border-box;
        }

        header {
            text-align: center;
            margin-bottom: 15px;
        }

        .setup-section {
            background: #e3f2fd;
            border-radius: 12px;
            padding: 15px;
            margin-bottom: 20px;
        }

        select {
            width: 100%;
            padding: 10px;
            border-radius: 8px;
            border: 1px solid #ccc;
            font-size: 14px;
            margin-top: 5px;
        }

        .mascote-section {
            background: linear-gradient(135deg, #e8f5e9, #c8e6c9);
            border-radius: 15px;
            padding: 20px;
            text-align: center;
            margin-bottom: 20px;
        }

        .avatar {
            font-size: 70px;
            margin: 10px 0;
            transition: transform 0.3s ease;
        }

        .avatar.growing {
            transform: scale(1.2);
        }

        .status-bar {
            background: #e0e0e0;
            border-radius: 10px;
            height: 20px;
            width: 100%;
            overflow: hidden;
            margin-top: 10px;
        }

        .progress {
            background: var(--primary);
            height: 100%;
            width: 20%;
            transition: width 0.5s ease;
        }

        .level-badge {
            background: var(--dark);
            color: white;
            padding: 5px 12px;
            border-radius: 20px;
            font-size: 14px;
            font-weight: bold;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 15px;
        }

        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
            font-size: 14px;
        }

        th {
            background-color: #f1f8e9;
            color: #33691e;
        }

        .checkbox-container {
            display: flex;
            gap: 10px;
        }

        input[type="checkbox"] {
            transform: scale(1.3);
            cursor: pointer;
            accent-color: var(--primary);
        }
    </style>
</head>
<body>

<div class="app-container">
    <header>
        <h1>🌱 Bokashinho</h1>
        <p>Crescendo com a sua planta real</p>
    </header>

    <!-- OPÇÃO: Seleção do Tipo de Planta -->
    <div class="setup-section">
        <label Over for="plant-select"><strong>Qual planta você vai cuidar?</strong></label>
        <select id="plant-select" onchange="changePlant()">
            <option value="🌿 Hortaliça / Geral">Horta (Alface, Temperos)</option>
            <option value="🌵 Suculenta / Cacto">Suculenta ou Cacto</option>
            <option value="🪴 Samambaia / Folhagem">Samambaia ou Folhagem</option>
        </select>
    </div>

    <!-- Seção do Mascote -->
    <div class="mascote-section">
        <span class="level-badge">Nível <span id="level">1</span></span>
        <div class="avatar" id="bokashi-avatar">🌱</div>
        <p>Acompanhando sua: <strong id="current-plant">🌿 Hortaliça / Geral</strong></p>
        <p><strong>Energia:</strong> <span id="energy-text">20%</span></p>
        <div class="status-bar">
            <div class="progress" id="energy-bar" style="width: 20%;"></div>
        </div>
    </div>

    <!-- Tabela de Tarefas -->
    <h3>Tabela de Manejo</h3>
    <table>
        <thead>
            <tr>
                <th>Tarefa</th>
                <th>Qtd / Dia</th>
                <th>Marcar</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><strong>Bokashi Mensal</strong></td>
                <td>1 colher</td>
                <td><input type="checkbox" id="task-bokashi" onchange="updateEnergy(50, 'bokashi', this)"></td>
            </tr>
            <tr>
                <td><strong>Rega da Semana</strong></td>
                <td><span id="rega-info">Terça e Sexta</span></td>
                <td class="checkbox-container">
                    <label><input type="checkbox" id="task-rega1" onchange="updateEnergy(15, 'rega1', this)"> 1º</label>
                    <label><input type="checkbox" id="task-rega2" onchange="updateEnergy(15, 'rega2', this)"> 2º</label>
                </td>
            </tr>
        </tbody>
    </table>
</div>

<script>
    // OPÇÃO: Banco de Dados Local (Carregar dados salvos ao abrir)
    let energy = parseInt(localStorage.getItem('b_energy')) || 20;
    let level = parseInt(localStorage.getItem('b_level')) || 1;
    let selectedPlant = localStorage.getItem('b_plant') || "🌿 Hortaliça / Geral";

    const avatars = { 1: "🌱", 2: "🌿", 3: "🌳", 4: "✨🌳✨" };

    // Regras dinâmicas por planta
    const plantRules = {
        "🌿 Hortaliça / Geral": "Terça e Sexta",
        "🌵 Suculenta / Cacto": "Apenas 1x por semana",
        "🪴 Samambaia / Folhagem": "Seg, Qua e Sex"
    };

    // Inicializar o App com as informações guardadas
    window.onload = function() {
        document.getElementById('plant-select').value = selectedPlant;
        document.getElementById('current-plant').innerText = selectedPlant;
        document.getElementById('rega-info').innerText = plantRules[selectedPlant] || "Regas padrão";
        
        // Recarrega os checkboxes salvos
        document.getElementById('task-bokashi').checked = localStorage.getItem('check_bokashi') === 'true';
        document.getElementById('task-rega1').checked = localStorage.getItem('check_rega1') === 'true';
        document.getElementById('task-rega2').checked = localStorage.getItem('check_rega2') === 'true';
        
        updateUI();
    };

    function changePlant() {
        selectedPlant = document.getElementById('plant-select').value;
        document.getElementById('current-plant').innerText = selectedPlant;
        document.getElementById('rega-info').innerText = plantRules[selectedPlant];
        localStorage.setItem('b_plant', selectedPlant);
    }

    function updateEnergy(value, id, checkbox) {
        if (checkbox.checked) {
            energy += value;
        } else {
            energy -= value;
        }

        // Sistema de níveis
        if (energy >= 100) {
            energy -= 100;
            level++;
            alert(`Parabéns! Seu Bokashinho evoluiu para o Nível ${level}! 🎉`);
        } else if (energy < 0) {
            if (level > 1) {
                level--;
                energy = 100 + energy;
            } else {
                energy = 0;
            }
        }

        // Salvar status atuais no "Banco de Dados" do navegador
        localStorage.setItem('b_energy', energy);
        localStorage.setItem('b_level', level);
        localStorage.setItem(`check_${id}`, checkbox.checked);

        updateUI();
    }

    function updateUI() {
        document.getElementById('energy-bar').style.width = energy + '%';
        document.getElementById('energy-text').innerText = energy + '%';
        document.getElementById('level').innerText = level;

        const avatarEl = document.getElementById('bokashi-avatar');
        avatarEl.classList.add('growing');
        setTimeout(() => avatarEl.classList.remove('growing'), 300);

        avatarEl.innerText = avatars[level] || "✨🌳✨";
    }
</script>

</body>
</html>
# Bokashinho2.0