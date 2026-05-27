# Mi-Ciclo
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Visualizador de Ciclo Real - Darya</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: #f7f9fa;
            color: #333;
            padding: 15px;
            margin: 0;
            display: flex;
            justify-content: center;
        }
        .container {
            background: white;
            padding: 20px;
            border-radius: 16px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.05);
            max-width: 450px;
            width: 100%;
        }
        h2 {
            color: #2c3e50;
            text-align: center;
            margin-top: 0;
            font-size: 22px;
            margin-bottom: 20px;
        }
        .input-group {
            margin-bottom: 25px;
            background: #f8f9fa;
            padding: 15px;
            border-radius: 12px;
        }
        label {
            display: block;
            font-weight: bold;
            margin-bottom: 8px;
            color: #5c6b73;
            font-size: 14px;
        }
        input[type="range"] {
            width: 100%;
            margin-bottom: 5px;
        }
        .days-display {
            text-align: right;
            font-weight: bold;
            color: #d63031;
            font-size: 20px;
        }
        
        /* Contenedor del Gráfico Visual */
        .visual-chart {
            position: relative;
            margin: 20px 0 35px 0;
        }
        .timeline {
            display: flex;
            height: 35px;
            border-radius: 8px;
            overflow: hidden;
            box-shadow: inset 0 2px 4px rgba(0,0,0,0.1);
        }
        .phase {
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 12px;
            font-weight: bold;
            transition: width 0.3s ease;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.2);
        }
        .phase-menstruation { background-color: #ff7675; }
        .phase-follicular { background-color: #ffeaa7; color: #b33939 !important; }
        .phase-fertile { background-color: #55efc4; color: #019473 !important; }
        .phase-luteal { background-color: #74b9ff; }
        
        /* Regla de números inferior */
        .scale-labels {
            position: relative;
            height: 20px;
            margin-top: 8px;
            font-size: 11px;
            font-weight: bold;
            color: #7f8c8d;
        }
        .scale-point {
            position: absolute;
            transform: translateX(-50%);
            text-align: center;
            transition: left 0.3s ease;
        }
        .scale-point::before {
            content: '|';
            display: block;
            margin-bottom: -2px;
            color: #bdc3c7;
        }

        .info-card {
            background-color: #f1f2f6;
            padding: 15px;
            border-radius: 10px;
            margin-top: 10px;
            border-left: 5px solid #74b9ff;
        }
        .info-card p {
            margin: 6px 0;
            font-size: 15px;
        }
        .info-card strong {
            color: #2c3e50;
        }
        .legend {
            margin-top: 20px;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            font-size: 12px;
            border-top: 1px solid #eee;
            padding-top: 15px;
        }
        .legend-item {
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .color-box {
            width: 14px;
            height: 14px;
            border-radius: 4px;
            flex-shrink: 0;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>Mi Visualizador de Ciclo 🌸</h2>
    
    <div class="input-group">
        <label for="cycleLength">Mueve para cambiar la duración de tu ciclo:</label>
        <input type="range" id="cycleLength" min="28" max="45" value="38">
        <div class="days-display" id="cycleValue">38 días</div>
    </div>

    <!-- Bloque Gráfico Rehecho -->
    <div class="visual-chart">
        <div class="timeline">
            <div id="pMenstruation" class="phase phase-menstruation">Regla</div>
            <div id="pFollicular" class="phase phase-follicular">Espera</div>
            <div id="pFertile" class="phase phase-fertile">🎯 Fértil</div>
            <div id="pLuteal" class="phase phase-luteal">Lútea</div>
        </div>
        
        <!-- Los números guía colocados exactamente debajo de los cortes -->
        <div class="scale-labels">
            <div style="left: 0%;" class="scale-point">Día 1</div>
            <div id="markMenses" class="scale-point"></div>
            <div id="markFertileStart" class="scale-point"></div>
            <div id="markOvulation" class="scale-point"></div>
            <div id="markEnd" class="scale-point"></div>
        </div>
    </div>

    <div class="info-card">
        <p><strong>Día de Ovulación:</strong> <span id="ovulationDay" style="color:#019473; font-weight:bold;">Día 24</span></p>
        <p><strong>Ventana Fértil:</strong> <span id="fertileWindow" style="font-weight:bold;">Días 19 al 24</span></p>
        <p><strong>Fase Lútea Fija:</strong> Últimos 14 días del ciclo</p>
    </div>

    <div class="legend">
        <div class="legend-item"><div class="color-box phase-menstruation"></div> Menstruación (Días 1-5)</div>
        <div class="legend-item"><div class="color-box phase-follicular"></div> Fase de Espera (Tranquila)</div>
        <div class="legend-item"><div class="color-box phase-fertile"></div> Ventana Fértil (Clara de huevo)</div>
        <div class="legend-item"><div class="color-box phase-luteal"></div> Fase Lútea (Espera al test)</div>
    </div>
</div>

<script>
    const slider = document.getElementById('cycleLength');
    const cycleValue = document.getElementById('cycleValue');
    const ovulationDay = document.getElementById('ovulationDay');
    const fertileWindow = document.getElementById('fertileWindow');

    const pMenstruation = document.getElementById('pMenstruation');
    const pFollicular = document.getElementById('pFollicular');
    const pFertile = document.getElementById('pFertile');
    const pLuteal = document.getElementById('pLuteal');

    // Elementos de las etiquetas numéricas inferiores
    const markMenses = document.getElementById('markMenses');
    const markFertileStart = document.getElementById('markFertileStart');
    const markOvulation = document.getElementById('markOvulation');
    const markEnd = document.getElementById('markEnd');

    function updateCycle() {
        const totalDays = parseInt(slider.value);
        cycleValue.innerText = totalDays + " días";
        
        const mensesDays = 5;
        const ovDay = totalDays - 14;
        const fertileStart = ovDay - 5;
        const follicularDays = fertileStart - mensesDays - 1;
        const fertileDaysCount = 6; 
        const lutealDays = 14;

        // Modificar textos informativos
        ovulationDay.innerText = "Día " + ovDay;
        fertileWindow.innerText = "Días " + fertileStart + " al " + ovDay;

        // Calcular porcentajes de ancho para las fases
        const pctMenses = (mensesDays / totalDays * 100);
        const pctFollicular = (follicularDays / totalDays * 100);
        const pctFertile = (fertileDaysCount / totalDays * 100);
        const pctLuteal = (lutealDays / totalDays * 100);

        pMenstruation.style.width = pctMenses + "%";
        pFollicular.style.width = pctFollicular + "%";
        pFertile.style.width = pctFertile + "%";
        pLuteal.style.width = pctLuteal + "%";

        // Mover los números exactamente debajo de cada línea divisoria de la barra
        markMenses.style.left = pctMenses + "%";
        markMenses.innerText = "Día " + mensesDays;

        markFertileStart.style.left = (pctMenses + pctFollicular) + "%";
        markFertileStart.innerText = "Día " + fertileStart;

        markOvulation.style.left = (pctMenses + pctFollicular + pctFertile) + "%";
        markOvulation.innerText = "Ovulación (Día " + ovDay + ")";

        markEnd.style.left = "100%";
        markEnd.innerText = "Día " + totalDays;
    }

    slider.addEventListener('input', updateCycle);
    updateCycle(); 
</script>

</body>
</html>
