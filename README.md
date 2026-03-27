<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Orchideen Bestimmungstool</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f4;
            display: flex;
            justify-content: center;
            padding: 40px 20px;
            color: #333;
        }
        .container {
            background: #ffffff;
            padding: 40px;
            border-radius: 12px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            max-width: 750px;
            width: 100%;
            text-align: center;
        }
        h1 { color: #2e7d32; margin-bottom: 30px; }
        .question-text { font-size: 1.1em; margin-bottom: 25px; min-height: 50px; font-weight: 500; }
        
        .choice-box {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-bottom: 30px;
        }
        .btn-choice {
            background-color: #fff;
            border: 2px solid #2e7d32;
            color: #2e7d32;
            padding: 20px;
            border-radius: 10px;
            cursor: pointer;
            font-size: 16px;
            transition: all 0.2s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            min-height: 160px;
            line-height: 1.4;
        }
        .btn-choice:hover {
            background-color: #2e7d32;
            color: #fff;
        }
        
        #result-screen {
            display: none;
            padding: 40px;
            border: 3px solid #2e7d32;
            border-radius: 15px;
            background-color: #e8f5e9;
        }
        .result-name { font-size: 32px; font-weight: bold; color: #1b5e20; margin: 20px 0; }
        
        .nav-bar {
            display: flex;
            justify-content: space-between;
            border-top: 1px solid #eee;
            padding-top: 20px;
            margin-top: 20px;
        }
        .btn-nav {
            background: #f0f0f0;
            border: 1px solid #ccc;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            color: #555;
            flex: 0 1 45%;
        }
        .btn-nav:hover { background: #e0e0e0; }
        .hidden { visibility: hidden; }

        @media (max-width: 600px) {
            .choice-box { grid-template-columns: 1fr; }
            .btn-choice { min-height: auto; }
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Orchideen-Gattungen</h1>

    <div id="quiz-ui">
        <p class="question-text">nach Kretzschmar 2018</p>
        <div class="choice-box">
            <button class="btn-choice" id="btnA" onclick="makeChoice('A')"></button>
            <button class="btn-choice" id="btnB" onclick="makeChoice('B')"></button>
        </div>
        <div class="nav-bar">
            <button class="btn-nav" id="backBtnQuiz" onclick="goBack()">zurück</button>
            <button class="btn-nav" onclick="reset()">Neustart ↻</button>
        </div>
    </div>

    <div id="result-screen">
        <p>Ergebnis der Bestimmung:</p>
        <div class="result-name" id="resultName"></div>
        <div class="nav-bar">
            <button class="btn-nav" id="backBtnResult" onclick="goBack()">zurück</button>
            <button class="btn-nav" onclick="reset()">Neu bestimmen ↻</button>
        </div>
    </div>
</div>

<script>
    // Daten exakt aus Gentes.csv übernommen [cite: 1, 2, 3, 4, 5]
    const rawData = [
        [0, "Wenige große Blüten, > 8 cm mit schuhförmiger Lippe", "RES: Cypripedium", "Blüten anders und viel kleiner", "1"],
        [1, "Pflanze mit grünen Blättern", "5", "Pflanze ohne grüne Blätter", "2"],
        [2, "ganze Pflanze violett überlaufen, Blüten violett", "RES: Limodorum", "Stängel gelb, bräunlich, weiß oder grün", "3"],
        [3, "Stängel grün, Blüten weiß, Lippe mit roten Punkten", "RES: Corallorhiza", "Stängel gelb, bräunlich oder (selten) weiß", "4"],
        [4, "wenige Blüten, weiße Lippe nach oben stehend", "RES: Epipogium", "Blüten in kompakter Ähre, alle Blüten in gleicher Farb wie Stängel, Lippe ohne Rippen", "RES: Neottia"],
        [5, "Lippe geteilt in mehr oder weniger voneinander abgesetzte Hinter- und Vorderlippe, letztere zungen- oder herzförmig", "6", "Lippe anders", "7"],
        [6, "Vorderteil der Lippe mit Längsrippen, Sepalen und Petalen weiß oder rosarot", "RES: Cephalanthera", "Lippe anders, Vorderteil der Lippe mit warzigen Strukturen, nicht mit Längsrippen, Hinterteil schüsselförmig und nektarführend", "RES: Epipactis"],
        [7, "Lippe gespornt", "15", "Lippe ungespornt", "8"],
        [8, "Lippe ähnlich einem Insekt gestaltet, das auf grüner oder farbiger Blüte sitzt", "RES: Ophrys", "Lippe anders", "9"],
        [9, "Zwei gegenständige Laubblätter deutlich oberhalb des Grundes angeordnet", "RES: Neottia (Listera)", "Blätter am Grund angeordnet", "10"],
        [10, "Lippe 3-lappig", "RES: Herminium", "Lippe ganzrandig", "11"],
        [11, "Sepalen und Petalen bilden einen Helm", "12", "Sepalen abstehend, nicht an Helmbildung beteiligt", "14"],
        [12, "Lippe grün oder grün-bräunlich", "RES: Chamorchis", "Lippe schneeweiß", "13"],
        [13, "Blütenstand gedreht, Blätter ohne Zeichnung", "RES: Spiranthes", "Blütenstand ungegdreht, Blätter mit Netzzeichnung", "RES: Goodyera"],
        [14, "Lippe nach unten gebogen, länger als 4 mm", "RES: Liparis", "Lippe winzig, nach oben stehend, unter 2,5 mm", "RES: Malaxis"],
        [15, "Sepalen und Petalen mit dünnen, spatelförmigen Spitzen", "RES: Traunsteinera", "Sepalen und Petalen anders", "16"],
        [16, "Sepalen und Petalen weiß oder grünlich-weiß, Sporn mehr als doppelt so lang wie der Fruchknoten", "RES: Platanthera", "Sepalen und Petaleni.d.R. anders gefärbt, Sporn viel kürzer", "17"],
        [17, "Blätter rosettig am Stängelgrund angeordnet", "18", "Blätter am Stängel verteilt, aufrecht", "24"],
        [18, "Nur 2 (selten 3) derbe, schmal eiförmige Blätter am Grund", "RES: Neottianthe", "Mehr Blätter und diese anders gestaltet", "19"],
        [19, "Blätter schmal, mindestens 5x so lang wie breit, vom Stängelgrund nach oben abstehend", "20", "Rosettige Laubblätter höchstens 5x so lang wie breit, dem Boden mehr oder weniger flach aufliegend", "21"],
        [20, "Lippen nach oben stehend, Blüten nicht resupiniert", "RES: Nigritella", "Lippen nach unten stehend, normal resupiniert", "RES: Gymnadenia"],
        [21, "Lippe mit über 4 cm langem, sprialig gedrehtem Mittellappen, Pflanze sehr robust", "RES: Himantoglossum", "Blüten anders", "22"],
        [22, "Lippe dreilappig, mit ungeteiltem Mittellappen, am Grund vor dem Sporneingang mit zwei erhabenen Längsleisten, Tragblätter grün, halb so lang bis deutlich länger als die Fruchtknoten, Tragblätter werden vor den Knospen entwickelt", "RES: Anacamptis", "Mittellappen an der Spitze mehr oder weniger geteilt, Lippe ohne Erhebungen an der Basis am Sporneingang (wenn mit, dann ohne Sporn), Tragblätter häutig", "23"],
        [23, "Griffelsäule sehr kurz, Lippe ohne Papillenhaare, Rosettenblätter bläulich grün, Narbenhöhle herzförmig mit den Ohren nach oben, die Arten bilden echte Winterrosetten aus", "RES: Neotinea", "Griffelsäule länglich, Narbenhöhle flächig, rundlich oder schwach trapezförmig, Blätter grün, es werden keine echten Winterrosetten gebildet (mit Ausnahme von O. anthropophora)", "RES: Orchis"],
        [24, "Blätter steif aufrecht, rinnig gefaltet, lang reitend am Stängelgrund ansetzend, dadurch am Stängel mehr oder weniger verteilt erscheinend", "RES: Anacamptis", "Blätter gefleckt oder ungefleckt, weit nach oben am Stängel verteilt", "RES: Dactylorhiza"]
    ];

    let history = [];
    let currentId = 0;

    function renderStep(id) {
        const row = rawData.find(item => item[0] === id);
        if (!row) return;

        currentId = id;
        document.getElementById('quiz-ui').style.display = 'block';
        document.getElementById('result-screen').style.display = 'none';
        document.getElementById('btnA').innerText = row[1];
        document.getElementById('btnB').innerText = row[3];
        
        const backBtnQuiz = document.getElementById('backBtnQuiz');
        backBtnQuiz.classList.toggle('hidden', history.length === 0);
    }

    function makeChoice(option) {
        const row = rawData.find(item => item[0] === currentId);
        const target = option === 'A' ? row[2] : row[4];

        if (typeof target === 'string' && target.startsWith("RES:")) {
            history.push(currentId);
            showResult(target.replace("RES:", "").trim());
        } else {
            history.push(currentId);
            renderStep(parseInt(target));
        }
    }

    function showResult(name) {
        document.getElementById('quiz-ui').style.display = 'none';
        document.getElementById('result-screen').style.display = 'block';
        document.getElementById('resultName').innerText = name;
    }

    function goBack() {
        if (history.length > 0) {
            currentId = history.pop();
            renderStep(currentId);
        }
    }

    function reset() {
        history = [];
        renderStep(0);
    }

    renderStep(0);
</script>

</body>
</html>
