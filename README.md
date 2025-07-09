# Simulador de Ionización de Aminoácidos
Simulador de Ionización de Aminoácidos 3
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador de pH vs. Estado de Ionización de Aminoácidos</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .charge-positive {
            color: #3b82f6; /* blue-500 */
        }
        .charge-negative {
            color: #ef4444; /* red-500 */
        }
        .charge-neutral {
            color: #374151; /* gray-700 */
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">
    <div class="w-full max-w-4xl bg-white rounded-2xl shadow-lg p-6 md:p-8">
        <header class="mb-6 text-center">
            <h1 class="text-2xl md:text-3xl font-bold text-gray-800">🧪 Simulador de Ionización de Aminoácidos</h1>
            <p class="text-gray-600 mt-2">Ajusta el pH y observa cómo cambia la carga neta de un aminoácido.</p>
        </header>

        <main class="grid grid-cols-1 lg:grid-cols-2 gap-8">
            <!-- Panel de Control -->
            <div class="bg-gray-50 rounded-xl p-6 border border-gray-200">
                <!-- Selección de Aminoácido -->
                <div class="mb-6">
                    <label for="amino-acid-select" class="block text-lg font-medium text-gray-700 mb-2">1. Elige un Aminoácido:</label>
                    <select id="amino-acid-select" class="w-full p-3 bg-white border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition">
                        <option value="glycine">Glicina</option>
                        <option value="alanine">Alanina</option>
                        <option value="aspartic_acid">Ácido Aspártico</option>
                        <option value="glutamic_acid">Ácido Glutámico</option>
                        <option value="lysine">Lisina</option>
                        <option value="arginine">Arginina</option>
                        <option value="histidine">Histidina</option>
                    </select>
                </div>

                <!-- Control de pH -->
                <div class="mb-6">
                    <label for="ph-slider" class="block text-lg font-medium text-gray-700 mb-2">2. Ajusta el pH:</label>
                    <input id="ph-slider" type="range" min="0" max="14" value="7.0" step="0.01" class="w-full h-3 bg-gray-200 rounded-lg appearance-none cursor-pointer range-lg">
                    <div class="text-center text-2xl font-bold text-blue-600 mt-2" id="ph-value">pH = 7.00</div>
                </div>

                <!-- Visualización de la Estructura -->
                <div class="mb-6">
                    <h3 class="text-lg font-medium text-gray-700 mb-3">3. Estado de Ionización:</h3>
                    <div class="bg-white p-4 rounded-lg border border-gray-200 text-center">
                        <div id="structure-display" class="text-xl md:text-2xl font-mono font-semibold tracking-tighter flex items-center justify-center space-x-2">
                            <!-- El contenido se genera con JS -->
                        </div>
                    </div>
                </div>
                
                <!-- Carga Neta -->
                <div>
                    <h3 class="text-lg font-medium text-gray-700 mb-2">4. Carga Neta Calculada:</h3>
                    <div id="net-charge-display" class="text-4xl font-bold text-center p-4 rounded-lg bg-blue-100 text-blue-800 border border-blue-200">
                        0
                    </div>
                </div>
            </div>

            <!-- Gráfico de Titulación -->
            <div class="bg-gray-50 rounded-xl p-6 border border-gray-200 flex flex-col">
                 <h3 class="text-lg font-medium text-gray-700 mb-4 text-center">Curva de Titulación</h3>
                <div class="relative flex-grow min-h-[300px]">
                    <canvas id="titration-chart"></canvas>
                </div>
            </div>
        </main>
    </div>

    <script>
        // --- DATA ---
        // Se añaden los nuevos aminoácidos: Ácido Glutámico, Arginina, Histidina
        const aminoAcids = {
            glycine: { 
                name: 'Glicina',
                pKa1: 2.34, // Carboxilo
                pKa2: 9.60, // Amino
                pKaR: null,
                R: 'H'
            },
            alanine: {
                name: 'Alanina',
                pKa1: 2.34,
                pKa2: 9.69,
                pKaR: null,
                R: 'CH₃'
            },
            aspartic_acid: {
                name: 'Ácido Aspártico',
                pKa1: 1.88,
                pKa2: 9.60,
                pKaR: 3.65, // Cadena lateral
                R: 'CH₂COOH'
            },
            glutamic_acid: {
                name: 'Ácido Glutámico',
                pKa1: 2.19,
                pKa2: 9.67,
                pKaR: 4.25, // Cadena lateral
                R: '(CH₂)₂COOH'
            },
            lysine: {
                name: 'Lisina',
                pKa1: 2.18,
                pKa2: 8.95,
                pKaR: 10.53, // Cadena lateral
                R: '(CH₂)₄NH₃⁺'
            },
            arginine: {
                name: 'Arginina',
                pKa1: 2.17,
                pKa2: 9.04,
                pKaR: 12.48, // Cadena lateral
                R: '(CH₂)₃-Guanidinio'
            },
            histidine: {
                name: 'Histidina',
                pKa1: 1.82,
                pKa2: 9.17,
                pKaR: 6.00, // Cadena lateral
                R: 'CH₂-Imidazol'
            }
        };

        // --- DOM ELEMENTS ---
        const selectElement = document.getElementById('amino-acid-select');
        const sliderElement = document.getElementById('ph-slider');
        const phValueDisplay = document.getElementById('ph-value');
        const structureDisplay = document.getElementById('structure-display');
        const netChargeDisplay = document.getElementById('net-charge-display');
        const chartCanvas = document.getElementById('titration-chart');

        let chart;
        let currentAAKey = 'glycine';

        // --- FUNCTIONS ---

        /**
         * Calcula la carga de cada grupo y la carga neta.
         * @param {number} pH - El pH actual.
         * @param {object} aaData - Los datos del aminoácido.
         * @returns {object} - Un objeto con las cargas individuales y la neta.
         */
        function calculateCharges(pH, aaData) {
            let chargeCarboxyl = (pH > aaData.pKa1) ? -1 : 0;
            let chargeAmino = (pH < aaData.pKa2) ? 1 : 0;
            let chargeR = 0;

            if (aaData.pKaR !== null) {
                // Se agrupan los aminoácidos por tipo de cadena lateral (ácida o básica)
                const isAcidicR = aaData.name === 'Ácido Aspártico' || aaData.name === 'Ácido Glutámico';
                const isBasicR = aaData.name === 'Lisina' || aaData.name === 'Arginina' || aaData.name === 'Histidina';

                if (isAcidicR) {
                    chargeR = (pH > aaData.pKaR) ? -1 : 0;
                } else if (isBasicR) {
                    chargeR = (pH < aaData.pKaR) ? 1 : 0;
                }
            }
            
            const netCharge = chargeCarboxyl + chargeAmino + chargeR;
            return { chargeCarboxyl, chargeAmino, chargeR, netCharge };
        }

        /**
         * Actualiza la visualización de la estructura del aminoácido.
         * @param {object} charges - Las cargas calculadas.
         * @param {object} aaData - Los datos del aminoácido.
         */
        function updateStructure(charges, aaData) {
            const aminoGroup = (charges.chargeAmino === 1) ? 'H₃N⁺' : 'H₂N';
            const carboxylGroup = (charges.chargeCarboxyl === -1) ? 'COO⁻' : 'COOH';
            let rGroup;

            // Se usa una estructura switch para manejar todos los casos de forma clara.
            switch (aaData.name) {
                case 'Glicina':
                    rGroup = 'H';
                    break;
                case 'Alanina':
                    rGroup = 'CH₃';
                    break;
                case 'Ácido Aspártico':
                    rGroup = (charges.chargeR === -1) ? 'CH₂COO⁻' : 'CH₂COOH';
                    break;
                case 'Ácido Glutámico':
                    rGroup = (charges.chargeR === -1) ? '(CH₂)₂COO⁻' : '(CH₂)₂COOH';
                    break;
                case 'Lisina':
                    rGroup = (charges.chargeR === 1) ? '(CH₂)₄NH₃⁺' : '(CH₂)₄NH₂';
                    break;
                case 'Arginina':
                    rGroup = (charges.chargeR === 1) ? '(CH₂)₃-Guanidinio⁺' : '(CH₂)₃-Guanidinio';
                    break;
                case 'Histidina':
                    rGroup = (charges.chargeR === 1) ? 'CH₂-Imidazol⁺' : 'CH₂-Imidazol';
                    break;
                default:
                    rGroup = aaData.R;
            }
            
            structureDisplay.innerHTML = `
                <span class="charge-positive">${aminoGroup}</span>
                <span>—</span>
                <div class="flex flex-col items-center">
                    <span>CH</span>
                    <span class="text-base charge-neutral">${rGroup}</span>
                </div>
                <span>—</span>
                <span class="charge-negative">${carboxylGroup}</span>
            `;
        }

        /**
         * Actualiza la pantalla de carga neta.
         * @param {number} netCharge - La carga neta.
         */
        function updateNetCharge(netCharge) {
            netChargeDisplay.textContent = netCharge > 0 ? `+${netCharge}` : netCharge.toFixed(0);
            netChargeDisplay.className = 'text-4xl font-bold text-center p-4 rounded-lg border';
            if (netCharge > 0) {
                netChargeDisplay.classList.add('bg-blue-100', 'text-blue-800', 'border-blue-200');
            } else if (netCharge < 0) {
                netChargeDisplay.classList.add('bg-red-100', 'text-red-800', 'border-red-200');
            } else {
                netChargeDisplay.classList.add('bg-gray-100', 'text-gray-800', 'border-gray-200');
            }
        }
        
        /**
         * Genera los puntos de datos para la curva de titulación.
         * @param {object} aaData - Los datos del aminoácido.
         * @returns {array} - Un array de objetos {x, y} para el gráfico.
         */
        function generateTitrationCurve(aaData) {
            const pKas = [aaData.pKa1, aaData.pKa2, aaData.pKaR].filter(pka => pka !== null).sort((a, b) => a - b);
            const curve = [];
            for (let ph = 0; ph <= 14; ph += 0.1) {
                let protonsLost = 0;
                pKas.forEach(pka => {
                    protonsLost += 1 / (1 + Math.pow(10, pka - ph));
                });
                curve.push({ x: protonsLost, y: ph });
            }
            return curve;
        }

        /**
         * Calcula la posición X (equivalentes) en el gráfico para un pH dado.
         * @param {number} pH - El pH actual.
         * @param {object} aaData - Los datos del aminoácido.
         * @returns {number} - El valor de equivalentes de OH-.
         */
        function getEquivalentsForPh(pH, aaData) {
            const pKas = [aaData.pKa1, aaData.pKa2, aaData.pKaR].filter(pka => pka !== null).sort((a, b) => a - b);
            let protonsLost = 0;
            pKas.forEach(pka => {
                protonsLost += 1 / (1 + Math.pow(10, pka - pH));
            });
            return protonsLost;
        }

        /**
         * Crea o actualiza el gráfico de titulación.
         * @param {object} aaData - Los datos del aminoácido.
         */
        function createOrUpdateChart(aaData) {
            const curveData = generateTitrationCurve(aaData);
            const pKas = [aaData.pKa1, aaData.pKa2, aaData.pKaR].filter(pka => pka !== null).sort((a, b) => a - b);

            if (chart) {
                chart.data.datasets[0].data = curveData;
                chart.data.datasets[1].data = [{ x: 0, y: 0 }]; // Reset current point
                chart.options.scales.x.max = pKas.length;
                chart.update();
            } else {
                const ctx = chartCanvas.getContext('2d');
                chart = new Chart(ctx, {
                    type: 'line',
                    data: {
                        datasets: [
                        {
                            label: 'Curva de Titulación',
                            data: curveData,
                            borderColor: 'rgb(59, 130, 246)',
                            backgroundColor: 'rgba(59, 130, 246, 0.5)',
                            tension: 0.4,
                            borderWidth: 3,
                            pointRadius: 0,
                        },
                        {
                            label: 'Punto Actual',
                            data: [{ x: 0, y: 0 }],
                            borderColor: 'rgb(239, 68, 68)',
                            backgroundColor: 'rgb(239, 68, 68)',
                            pointRadius: 8,
                            pointHoverRadius: 10,
                            type: 'scatter'
                        }
                        ]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        scales: {
                            y: {
                                beginAtZero: true,
                                max: 14,
                                title: { display: true, text: 'pH', font: { size: 14, weight: 'bold' } }
                            },
                            x: {
                                type: 'linear', position: 'bottom', min: 0, max: pKas.length,
                                title: { display: true, text: 'Equivalentes de OH⁻ agregados', font: { size: 14, weight: 'bold' } }
                            }
                        },
                        plugins: { legend: { display: false }, tooltip: { enabled: false } }
                    }
                });
            }
        }
        
        /**
         * La función principal que se ejecuta en cada actualización.
         */
        function updateSimulation() {
            const pH = parseFloat(sliderElement.value);
            const aaData = aminoAcids[currentAAKey];

            phValueDisplay.textContent = `pH = ${pH.toFixed(2)}`;
            const charges = calculateCharges(pH, aaData);
            updateStructure(charges, aaData);
            updateNetCharge(charges.netCharge);

            const equiv = getEquivalentsForPh(pH, aaData);
            chart.data.datasets[1].data = [{ x: equiv, y: pH }];
            chart.update();
        }
        
        /**
         * Inicializa la simulación para un aminoácido específico.
         */
        function initialize() {
            currentAAKey = selectElement.value;
            const aaData = aminoAcids[currentAAKey];
            
            // Lógica mejorada para calcular el pI y posicionar el slider
            const pKas = [aaData.pKa1, aaData.pKa2, aaData.pKaR].filter(pka => pka !== null).sort((a, b) => a - b);
            let pI = 7.0;
            const isAcidic = aaData.name === 'Ácido Aspártico' || aaData.name === 'Ácido Glutámico';
            const isBasic = aaData.name === 'Lisina' || aaData.name === 'Arginina' || aaData.name === 'Histidina';

            if (pKas.length === 2) { // Aminoácido neutro
                pI = (pKas[0] + pKas[1]) / 2;
            } else if (isAcidic) { // Aminoácido ácido
                pI = (pKas[0] + pKas[1]) / 2;
            } else if (isBasic) { // Aminoácido básico
                pI = (pKas[1] + pKas[2]) / 2;
            }
            sliderElement.value = pI;

            createOrUpdateChart(aaData);
            updateSimulation();
        }

        // --- EVENT LISTENERS ---
        sliderElement.addEventListener('input', updateSimulation);
        selectElement.addEventListener('change', initialize);

        // --- INITIAL RUN ---
        window.onload = initialize;

    </script>
</body>
</html>
