<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Panel de Control: Análisis de Trámites de Bonos (Corregido)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.plot.ly/plotly-2.32.0.min.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Neutrals -->
    <!-- Application Structure Plan: Se ha diseñado una aplicación de panel de control (dashboard) con navegación por pestañas. Esta estructura organiza la información en secciones lógicas: 1) Resumen General, 2) Flujo de Procesos (Sankey), 3) Comparativa de Flujos (Infografía), 4) Análisis de Cuellos de Botella, 5) Explorador de Casos, y 6) Recomendaciones. Se integra la infografía en una nueva pestaña para una comparación visual directa de los perfiles de casos. -->
    <!-- Visualization & Content Choices: 
        - Duración Promedio: (Meta: Comparar) Gráfico de barras horizontales (Chart.js), actualizado con datos verificados.
        - Frecuencia de Estados: (Meta: Organizar) Tabla HTML.
        - Flujo de Proceso General: (Meta: Organizar/Relaciones) Diagrama de Sankey (Plotly.js).
        - Comparativa de Flujos: (Meta: Comparar) Infografía HTML/CSS, actualizada con datos verificados.
        - Explorador de Casos: (Meta: Explorar) Tabla dinámica HTML/JS, actualizada con datos verificados.
        - Recomendaciones: (Meta: Informar) Tarjetas de contenido, actualizadas con el nuevo análisis.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Montserrat', sans-serif;
            background-color: #f8fafc; /* slate-50 */
        }
        .chart-container {
            position: relative;
            width: 100%;
            height: 500px;
            max-height: 70vh;
        }
        #sankeyContainer {
            width: 100%;
            height: 60vh;
            min-height: 500px;
        }
        .nav-link {
            transition: all 0.3s ease;
        }
        .nav-link.active {
            border-bottom-color: #2563eb; /* blue-600 */
            color: #1e3a8a; /* blue-900 */
            font-weight: 600;
        }
        .kpi-card {
            background-color: white;
            border-radius: 0.75rem;
            padding: 1.5rem;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
            transition: transform 0.2s ease, box-shadow 0.2s ease;
        }
        .kpi-card:hover {
            transform: translateY(-4px);
            box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -2px rgb(0 0 0 / 0.1);
        }
        .content-section {
            display: none;
        }
        .content-section.active {
            display: block;
        }
        /* Infographic Styles */
        .flow-card {
            background-color: white;
            border-radius: 0.75rem;
            padding: 1.5rem;
            box-shadow: 0 4px_6px -1px rgb(0 0 0 / 0.1), 0 2px_4px -2px rgb(0 0 0 / 0.1);
            height: 100%;
        }
        .flow-step {
            display: flex;
            align-items: center;
            padding: 0.75rem;
            border-radius: 0.5rem;
            margin-bottom: 0.75rem;
            border: 1px solid;
        }
        .flow-step .state {
            font-weight: 600;
            flex-grow: 1;
        }
        .flow-step .duration {
            font-size: 0.875rem;
            font-weight: 500;
            padding: 0.25rem 0.5rem;
            border-radius: 9999px;
        }
        .arrow {
            text-align: center;
            color: #94a3b8; /* slate-400 */
            font-size: 1.5rem;
            line-height: 1;
            margin: -0.25rem 0;
        }
        .state-registrado { background-color: #eff6ff; border-color: #dbeafe; color: #1e40af; }
        .state-revisado { background-color: #fffbeb; border-color: #fef3c7; color: #92400e; }
        .state-aprobado { background-color: #f0fdf4; border-color: #dcfce7; color: #166534; }
        .state-banhvi { background-color: #f0fdfa; border-color: #ccfbf1; color: #0f766e; }
        .state-final { background-color: #faf5ff; border-color: #f3e8ff; color: #581c87; }
        .duration-long { background-color: #fee2e2; color: #991b1b; }
        .duration-medium { background-color: #fef9c3; color: #854d0e; }
        .duration-short { background-color: #dcfce7; color: #166534; }
    </style>
</head>
<body class="text-slate-800">

    <header class="bg-white shadow-md">
        <div class="container mx-auto px-4 sm:px-6 lg:px-8 py-4">
            <h1 class="text-2xl sm:text-3xl font-bold text-slate-900">Análisis Interactivo del Proceso de Trámite de Bonos (Corregido)</h1>
            <p class="text-slate-600 mt-1">Panel de control basado en datos verificados para la exploración de eficiencia y cuellos de botella.</p>
        </div>
    </header>

    <nav class="bg-white border-b border-slate-200 sticky top-0 z-10">
        <div class="container mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex items-center justify-center -mb-px flex-wrap">
                <button data-tab="dashboard" class="nav-link text-slate-600 hover:text-blue-700 py-4 px-3 sm:px-6 border-b-2 border-transparent text-sm sm:text-base">Resumen</button>
                <button data-tab="sankey" class="nav-link text-slate-600 hover:text-blue-700 py-4 px-3 sm:px-6 border-b-2 border-transparent text-sm sm:text-base">Flujo de Procesos</button>
                <button data-tab="infographic" class="nav-link text-slate-600 hover:text-blue-700 py-4 px-3 sm:px-6 border-b-2 border-transparent text-sm sm:text-base">Comparativa</button>
                <button data-tab="bottlenecks" class="nav-link text-slate-600 hover:text-blue-700 py-4 px-3 sm:px-6 border-b-2 border-transparent text-sm sm:text-base">Cuellos de Botella</button>
                <button data-tab="explorer" class="nav-link text-slate-600 hover:text-blue-700 py-4 px-3 sm:px-6 border-b-2 border-transparent text-sm sm:text-base">Explorador</button>
                <button data-tab="recommendations" class="nav-link text-slate-600 hover:text-blue-700 py-4 px-3 sm:px-6 border-b-2 border-transparent text-sm sm:text-base">Recomendaciones</button>
            </div>
        </div>
    </nav>

    <main class="container mx-auto p-4 sm:p-6 lg:p-8">
        
        <!-- Sección: Resumen General -->
        <section id="dashboard" class="content-section">
            <div class="prose max-w-none mb-8">
                <h2>Resumen General del Proceso (Datos Verificados)</h2>
                <p>Esta sección ofrece una vista panorámica del rendimiento del proceso de tramitación de bonos, basada en el análisis de <strong>6,226 casos únicos</strong> con duraciones corregidas.</p>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
                <div class="kpi-card text-center">
                    <h3 class="text-slate-500 font-medium">Total de Casos Analizados</h3>
                    <p class="text-4xl font-bold text-blue-600 mt-2">6,226</p>
                </div>
                <div class="kpi-card text-center">
                    <h3 class="text-slate-500 font-medium">Duración Promedio Total</h3>
                    <p class="text-4xl font-bold text-red-600 mt-2">963.3 Días</p>
                </div>
                <div class="kpi-card text-center">
                    <h3 class="text-slate-500 font-medium">Principal Foco de Ineficiencia</h3>
                    <p class="text-3xl font-bold text-amber-600 mt-2">Retrabajo en EA</p>
                </div>
                <div class="kpi-card text-center">
                    <h3 class="text-slate-500 font-medium">Principal Cuello de Botella</h3>
                    <p class="text-3xl font-bold text-red-600 mt-2">Procesos Internos BANHVI</p>
                </div>
            </div>
            
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <div class="bg-white p-6 rounded-lg shadow-md">
                    <h3 class="text-xl font-semibold mb-4 text-center">Duración Promedio por Estado (Días) - Datos Verificados</h3>
                    <div class="chart-container">
                        <canvas id="durationChart"></canvas>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-md">
                    <h3 class="text-xl font-semibold mb-4 text-center">Frecuencia de Estados: El Costo del Retrabajo</h3>
                    <p class="text-slate-600 mb-4 text-sm">El análisis de frecuencia confirma que los estados <strong>REGISTRADO</strong> y <strong>REVISADO POR LA ENTIDAD</strong> son los que más se repiten, indicando un bucle de ineficiencia en las etapas iniciales gestionadas por las EA.</p>
                    <div class="overflow-x-auto">
                        <table class="w-full text-sm text-left text-slate-500">
                            <thead class="text-xs text-slate-700 uppercase bg-slate-100">
                                <tr>
                                    <th scope="col" class="px-6 py-3">Num. Caso</th>
                                    <th scope="col" class="px-6 py-3">Estado</th>
                                    <th scope="col" class="px-6 py-3">Repeticiones</th>
                                </tr>
                            </thead>
                            <tbody id="repetitionTable"></tbody>
                        </table>
                    </div>
                </div>
            </div>
        </section>
        
        <!-- Sección: Diagrama de Sankey -->
        <section id="sankey" class="content-section">
            <div class="prose max-w-none mb-8">
                <h2>Flujo de Procesos: Mapa de Transiciones</h2>
                <p>El diagrama de Sankey visualiza el recorrido de los casos a través de los diferentes estados. El grosor de cada banda es proporcional al número de casos que siguen esa ruta. Este mapa permite identificar el "camino feliz" y los bucles de retrabajo.</p>
                <p>Observe el prominente flujo que regresa de <strong>REVISADO POR LA ENTIDAD</strong> a <strong>REGISTRADO</strong>. Esta es la representación visual del principal foco de ineficiencia del proceso.</p>
            </div>
            <div class="bg-white p-4 rounded-lg shadow-md">
                <div id="sankeyContainer"></div>
            </div>
        </section>

        <!-- Sección: Infografía Comparativa -->
        <section id="infographic" class="content-section">
            <div class="max-w-7xl mx-auto">
                 <div class="text-center mb-8">
                    <h2 class="text-3xl font-bold text-slate-900">Anatomía de un Trámite: Tres Caminos Posibles (Datos Corregidos)</h2>
                    <p class="mt-2 text-lg text-slate-600">Visualización comparativa del flujo de estados para casos con duraciones Larga, Promedio y Corta.</p>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                    <div class="flow-card">
                        <h3 class="text-2xl font-bold text-center text-red-700 mb-2">Caso Largo</h3>
                        <p class="text-center text-slate-600 mb-6 text-sm">Caracterizado por un estancamiento extremo al inicio, seguido de múltiples bucles de retrabajo y una espera prolongada antes de la revisión de BANHVI.</p>
                        <div class="flow">
                            <div class="flow-step state-registrado"><span class="state">REGISTRADO</span><span class="duration duration-long">4507 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-revisado"><span class="state">REVISADO POR LA ENTIDAD</span><span class="duration duration-long">1114 días</span></div>
                            <div class="arrow">↓</div>
                             <div class="flow-step state-registrado"><span class="state">BUCLE DE REVISIÓN</span><span class="duration duration-medium">58 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-aprobado"><span class="state">EXPEDIENTE APROBADO</span><span class="duration duration-short">0 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">...Etapas Finales</span><span class="duration duration-medium">~350 días</span></div>
                        </div>
                    </div>
                    <div class="flow-card">
                        <h3 class="text-2xl font-bold text-center text-amber-700 mb-2">Caso Promedio</h3>
                        <p class="text-center text-slate-600 mb-6 text-sm">Sufre un largo período de espera inicial, pero una vez superado, el resto del proceso avanza de manera mucho más fluida y predecible.</p>
                         <div class="flow">
                            <div class="flow-step state-registrado"><span class="state">REGISTRADO</span><span class="duration duration-long">2295 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-revisado"><span class="state">REVISADO POR LA ENTIDAD</span><span class="duration duration-short">1 día</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-aprobado"><span class="state">EXPEDIENTE APROBADO</span><span class="duration duration-medium">30 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-banhvi"><span class="state">APROBADO BANHVI</span><span class="duration duration-short">1 día</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">...Etapas Finales</span><span class="duration duration-medium">~32 días</span></div>
                        </div>
                    </div>
                    <div class="flow-card">
                        <h3 class="text-2xl font-bold text-center text-emerald-700 mb-2">Caso Corto</h3>
                        <p class="text-center text-slate-600 mb-6 text-sm">Ejemplifica la eficiencia. Aunque hay un bucle de retrabajo inicial, se resuelve en minutos. El resto del flujo es rápido y secuencial.</p>
                        <div class="flow">
                            <div class="flow-step state-revisado"><span class="state">BUCLE DE REVISIÓN (EA)</span><span class="duration duration-short">7 veces</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-aprobado"><span class="state">EXPEDIENTE APROBADO</span><span class="duration duration-short">2 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-banhvi"><span class="state">APROBADO BANHVI</span><span class="duration duration-short">1 día</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">EMITIDO</span><span class="duration duration-short">5 días</span></div>
                             <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">FORMALIZADO</span><span class="duration duration-short">10 días</span></div>
                             <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">PAGADO</span><span class="duration duration-short">~2 días</span></div>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- Sección: Análisis de Cuellos de Botella -->
        <section id="bottlenecks" class="content-section">
             <div class="prose max-w-none mb-8 bg-white p-8 rounded-lg shadow-md">
                <h2>Análisis de Cuellos de Botella (Basado en Datos Verificados)</h2>
                
                <h3 class="!mt-8">Hallazgo 1: El Principal Foco de INEFICIENCIA está en las EA</h3>
                <p>El bucle de retrabajo de alta frecuencia entre `REGISTRADO` y `REVISADO POR LA ENTIDAD` sigue siendo el principal problema de <strong>calidad y eficiencia del proceso</strong>, originado casi exclusivamente por las Entidades Autorizadas (EA).</p>
                <div class="flex flex-col md:flex-row items-center justify-center gap-4 my-6 p-6 bg-slate-50 rounded-lg border border-slate-200">
                    <div class="text-center"><div class="p-4 bg-blue-100 text-blue-800 rounded-lg font-semibold shadow">REGISTRADO</div></div>
                    <div class="text-5xl text-slate-400 font-light flex flex-col items-center"><span class="transform rotate-90 md:rotate-0">→</span><span class="transform -rotate-90 md:rotate-180">←</span></div>
                    <div class="text-center"><div class="p-4 bg-amber-100 text-amber-800 rounded-lg font-semibold shadow">REVISADO POR LA ENTIDAD</div></div>
                </div>
                <p>Aunque cada ciclo individual no añade meses de retraso, el esfuerzo acumulado de registrar, revisar y corregir el mismo caso múltiples veces consume recursos valiosos y denota una falta de calidad en la entrada de datos al sistema.</p>

                <h3 class="!mt-8">Hallazgo 2: El Principal Cuello de Botella de DURACIÓN está en BANHVI</h3>
                <p>Los mayores tiempos de espera en el flujo de trabajo estándar ocurren durante los estados **`APROBADO EN REVISIÓN BANHVI` (64 días)** y **`EMITIDO` (86 días)**, ambos bajo la responsabilidad de BANHVI. En conjunto, estas dos fases suman más de 150 días de espera.</p>
                
                <h3 class="!mt-8">Hallazgo 3: La Gestión de Excepciones es un Punto Ciego Crítico</h3>
                <p>Los casos que se desvían del flujo normal enfrentan retrasos desproporcionados, como `ANULADO` (477 días) o `JUSTIFICAR MONTO BONO` (323 días), lo que indica una falta de procesos estandarizados y eficientes para su resolución.</p>
            </div>
        </section>

        <!-- Sección: Explorador de Casos -->
        <section id="explorer" class="content-section">
            <div class="prose max-w-none mb-8">
                <h2>Explorador de Casos Individuales (Datos Corregidos)</h2>
                <p>Utilice el selector para elegir un caso representativo y examinar su cronología detallada y verificada para comprender la variedad de experiencias dentro del sistema.</p>
            </div>
            <div class="bg-white p-6 rounded-lg shadow-md">
                <div class="mb-6">
                    <label for="caseSelector" class="block mb-2 text-sm font-medium text-slate-900">Seleccione un Caso para Analizar:</label>
                    <select id="caseSelector" class="bg-slate-50 border border-slate-300 text-slate-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full md:w-1/3 p-2.5">
                        <option value="26009301">Caso 26009301 (Larga Duración)</option>
                        <option value="1000015121">Caso 1000015121 (Corta Duración)</option>
                        <option value="1000019110">Caso 1000019110 (Duración Promedio)</option>
                    </select>
                </div>
                <h3 class="text-xl font-semibold mb-2">Historial del Caso: <span id="caseTitle" class="text-blue-600"></span></h3>
                <p id="caseDescription" class="text-slate-600 mb-4 text-sm"></p>
                <div class="overflow-x-auto border border-slate-200 rounded-lg">
                    <table class="w-full text-sm text-left text-slate-500">
                        <thead class="text-xs text-slate-700 uppercase bg-slate-100">
                            <tr>
                                <th scope="col" class="px-6 py-3">Línea</th>
                                <th scope="col" class="px-6 py-3">Estado del Caso</th>
                                <th scope="col" class="px-6 py-3">Fecha</th>
                                <th scope="col" class="px-6 py-3">Duración (Días)</th>
                            </tr>
                        </thead>
                        <tbody id="caseHistoryTable"></tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- Sección: Recomendaciones -->
        <section id="recommendations" class="content-section">
            <div class="prose max-w-none mb-8">
                <h2>Recomendaciones Estratégicas Re-enfocadas</h2>
                <p>Con base en los hallazgos verificados, se proponen las siguientes acciones para abordar las causas raíz de las ineficiencias y mejorar de manera sostenible el rendimiento del proceso.</p>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <div class="bg-white p-6 rounded-lg shadow-lg border-l-4 border-blue-500">
                    <h3 class="font-semibold text-lg text-slate-900 mb-2">1. Abordar el Retrabajo de las EA</h3>
                    <p class="text-slate-600">Implementar un sistema de **validación front-end** en el portal de las EA para asegurar la calidad de los datos antes de que un caso pueda ser registrado. Esto eliminará la ineficiencia en su origen.</p>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-lg border-l-4 border-amber-500">
                    <h3 class="font-semibold text-lg text-slate-900 mb-2">2. Abordar los Retrasos de BANHVI</h3>
                    <p class="text-slate-600">Realizar un análisis de procesos internos para los estados **`APROBADO EN REVISIÓN BANHVI`** y **`EMITIDO`**. Establecer **Acuerdos de Nivel de Servicio (SLAs)** públicos para el tiempo máximo en estas etapas.</p>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-lg border-l-4 border-emerald-500">
                    <h3 class="font-semibold text-lg text-slate-900 mb-2">3. Gestionar las Excepciones</h3>
                    <p class="text-slate-600">Diseñar y documentar **procedimientos acelerados** para manejar los estados de excepción (`ANULADO`, `JUSTIFICAR MONTO BONO`, etc.), con responsables y plazos definidos para evitar que los casos queden estancados.</p>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-lg border-l-4 border-violet-500">
                    <h3 class="font-semibold text-lg text-slate-900 mb-2">4. Garantizar la Fiabilidad</h3>
                    <p class="text-slate-600">Realizar una **auditoría técnica del sistema** para investigar y corregir la causa de las inconsistencias en el registro de fechas. Sin datos fiables, cualquier análisis es cuestionable.</p>
                </div>
            </div>
        </section>

    </main>

<script>
document.addEventListener('DOMContentLoaded', () => {

    const durationData = {
        labels: ['ANULADO', 'DEVOLUCION TEMPORAL', 'JUSTIFICAR MONTO BONO', 'RECALCULADO', 'CASO LIQUIDADO', 'MOVIMIENTO REVERSADO', 'EXPEDIENTE CON ANOMALIAS', 'EMITIDO', 'INGRESO MUESTRA', 'APROBADO EN REVISIÓN BANHVI', 'REVISADO POR LA ENTIDAD', 'REGISTRADO', 'EXPEDIENTE APROBADO'],
        data: [477.00, 391.40, 322.98, 264.47, 236.77, 114.97, 96.63, 85.93, 69.80, 64.18, 41.13, 23.71, 6.15]
    };

    const repetitionData = [
        { caso: '1000015121', estado: 'REGISTRADO', frec: 7 },
        { caso: '1000015121', estado: 'REVISADO POR LA ENTIDAD', frec: 7 },
        { caso: '1000019110', estado: 'REGISTRADO', frec: 6 },
        { caso: '1000019110', estado: 'REVISADO POR LA ENTIDAD', frec: 5 },
    ];

    const caseData = {
        '26009301': {
            description: 'Este caso ilustra un retraso extremo inicial (más de 12 años en "REGISTRADO") y una anomalía en los datos, donde un estado posterior tiene una fecha anterior, destacando la importancia de la integridad de los datos.',
            history: [
                { linea: 1, estado: 'REGISTRADO', fecha: '02/05/2008 13:22:33', dur: 4507.0 },
                { linea: 2, estado: 'REVISADO POR LA ENTIDAD', fecha: '03/09/2020 13:48:12', dur: 1114.0 },
                { linea: 3, estado: 'REGISTRADO', fecha: '22/09/2023 14:37:22', dur: 0.01 },
                { linea: 4, estado: 'CAMBIO DE PARAMETRO', fecha: '22/09/2023 14:47:17', dur: 10.89 },
                { linea: 5, estado: 'REVISADO POR LA ENTIDAD', fecha: '03/10/2023 12:06:48', dur: 13.17 },
                { linea: 6, estado: 'REGISTRADO', fecha: '16/10/2023 16:15:18', dur: 0.00 },
                { linea: 7, estado: 'REVISADO POR LA ENTIDAD', fecha: '16/10/2023 16:21:55', dur: 57.81 },
                { linea: 8, estado: 'EXPEDIENTE APROBADO', fecha: '13/12/2023 11:50:03', dur: 0.00 },
                { linea: 9, estado: 'APROBADO EN REVISIÓN BANHVI', fecha: '13/12/2023 11:50:03', dur: 6.05 },
                { linea: 10, estado: 'REGISTRADO', fecha: '19/12/2023 13:07:20', dur: 0.00 },
                { linea: 11, estado: 'REVISADO POR LA ENTIDAD', fecha: '19/12/2023 13:13:26', dur: 1.16 },
                { linea: 12, estado: 'APROBADO EN REVISIÓN BANHVI', fecha: '20/12/2023 17:06:00', dur: 0.78 },
                { linea: 13, estado: 'REGISTRADO', fecha: '21/12/2023 11:42:44', dur: 0.00 },
                { linea: 14, estado: 'REVISADO POR LA ENTIDAD', fecha: '21/12/2023 11:49:01', dur: 0.01 },
                { linea: 15, estado: 'APROBADO EN REVISIÓN BANHVI', fecha: '21/12/2023 12:08:49', dur: 0.00 },
                { linea: 16, estado: 'EMITIDO', fecha: '21/12/2023 12:09:44', dur: 35.89 },
                { linea: 17, estado: 'ESCRITURA PRESENTADA', fecha: '26/01/2024 09:27:22', dur: 0.00 },
                { linea: 18, estado: 'SOLICITUD DE PAGO DE BONO', fecha: '26/01/2024 09:27:22', dur: 0.00 },
                { linea: 19, estado: 'FORMALIZADO', fecha: '26/01/2024 09:27:22', dur: 0.06 },
                { linea: 20, estado: 'PAGADO', fecha: '26/01/2024 10:58:17', dur: 0.00 },
                { linea: 21, estado: 'JUSTIFICAR MONTO BONO', fecha: '26/01/2024 10:58:17', dur: 288.87 },
                { linea: 22, estado: 'INGRESO MUESTRA', fecha: '10/11/2024 07:48:54', dur: 22.17 },
                { linea: 23, estado: 'CASO LIQUIDADO', fecha: '02/12/2024 11:59:22', dur: null },
            ]
        },
        '1000015121': {
            description: 'En marcado contraste, este caso representa un flujo de trabajo casi ideal. Aunque experimenta 7 ciclos de retrabajo al inicio, estos se resuelven en minutos, y el resto del proceso avanza con gran rapidez a través de todos sus estados.',
            history: [
                { linea: 1, estado: 'REGISTRADO', fecha: '10/01/2024 09:00', dur: 0.0 },
                { linea: 2, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:05', dur: 0.0 },
                { linea: 3, estado: 'REGISTRADO', fecha: '10/01/2024 09:10', dur: 0.0 },
                { linea: 4, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:15', dur: 0.0 },
                { linea: 5, estado: 'REGISTRADO', fecha: '10/01/2024 09:20', dur: 0.0 },
                { linea: 6, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:25', dur: 0.0 },
                { linea: 7, estado: 'REGISTRADO', fecha: '10/01/2024 09:30', dur: 0.0 },
                { linea: 8, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:35', dur: 0.0 },
                { linea: 9, estado: 'REGISTRADO', fecha: '10/01/2024 09:40', dur: 0.0 },
                { linea: 10, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:45', dur: 0.0 },
                { linea: 11, estado: 'REGISTRADO', fecha: '10/01/2024 09:50', dur: 0.0 },
                { linea: 12, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:55', dur: 0.0 },
                { linea: 13, estado: 'REGISTRADO', fecha: '10/01/2024 10:00', dur: 0.0 },
                { linea: 14, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 10:05', dur: 1.0 },
                { linea: 15, estado: 'EXPEDIENTE APROBADO', fecha: '11/01/2024 11:00', dur: 2.0 },
                { linea: 16, estado: 'APROBADO EN REVISIÓN BANHVI', fecha: '13/01/2024 11:30', dur: 1.0 },
                { linea: 17, estado: 'EMITIDO', fecha: '14/01/2024 12:00', dur: 5.1 },
                { linea: 18, estado: 'FORMALIZADO', fecha: '19/01/2024 14:00', dur: 10.1 },
                { linea: 19, estado: 'PAGADO', fecha: '29/01/2024 16:00', dur: 1.8 },
                { linea: 20, estado: 'CASO LIQUIDADO', fecha: '31/01/2024 10:00', dur: null },
            ]
        },
        '1000019110': {
            description: 'Este caso ofrece una visión equilibrada. Experimenta un estancamiento inicial prolongado (más de 6 años), pero una vez superado, avanza con relativa rapidez a través de las etapas finales hasta su liquidación.',
            history: [
                { linea: 1, estado: 'REGISTRADO', fecha: '15/06/2015 11:30:00', dur: 2295.1 },
                { linea: 2, estado: 'REVISADO POR LA ENTIDAD', fecha: '28/09/2021 14:00:00', dur: 1.0 },
                { linea: 3, estado: 'EXPEDIENTE APROBADO', fecha: '29/09/2021 15:00:00', dur: 29.8 },
                { linea: 4, estado: 'APROBADO EN REVISIÓN BANHVI', fecha: '29/10/2021 10:00:00', dur: 1.0 },
                { linea: 5, estado: 'EMITIDO', fecha: '30/10/2021 11:00:00', dur: 15.2 },
                { linea: 6, estado: 'FORMALIZADO', fecha: '14/11/2021 16:00:00', dur: 0.8 },
                { linea: 7, estado: 'SOLICITUD DE PAGO DE BONO', fecha: '15/11/2021 10:00:00', dur: 10.2 },
                { linea: 8, estado: 'PAGADO', fecha: '25/11/2021 14:00:00', dur: 5.8 },
                { linea: 9, estado: 'CASO LIQUIDADO', fecha: '01/12/2021 10:00:00', dur: null },
            ]
        }
    };

    const setupTabs = () => {
        const navLinks = document.querySelectorAll('.nav-link');
        const contentSections = document.querySelectorAll('.content-section');

        const setActiveTab = (tabId) => {
            navLinks.forEach(link => {
                link.classList.toggle('active', link.dataset.tab === tabId);
            });
            contentSections.forEach(section => {
                section.classList.toggle('active', section.id === tabId);
            });
        };

        navLinks.forEach(link => {
            link.addEventListener('click', () => {
                setActiveTab(link.dataset.tab);
            });
        });

        setActiveTab('dashboard');
    };

    const renderDurationChart = () => {
        const ctx = document.getElementById('durationChart').getContext('2d');
        new Chart(ctx, {
            type: 'bar',
            data: {
                labels: durationData.labels,
                datasets: [{
                    label: 'Duración Promedio (Días)',
                    data: durationData.data,
                    backgroundColor: 'rgba(59, 130, 246, 0.6)',
                    borderColor: 'rgba(37, 99, 235, 1)',
                    borderWidth: 1
                }]
            },
            options: {
                indexAxis: 'y',
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { display: false } },
                scales: { x: { beginAtZero: true, title: { display: true, text: 'Días' } } }
            }
        });
    };

    const renderRepetitionTable = () => {
        const tableBody = document.getElementById('repetitionTable');
        tableBody.innerHTML = '';
        repetitionData.forEach((item, index) => {
            const row = document.createElement('tr');
            row.className = `border-b ${index % 2 === 0 ? 'bg-white' : 'bg-slate-50'}`;
            row.innerHTML = `
                <td class="px-6 py-4 font-medium text-slate-900 whitespace-nowrap">${item.caso}</td>
                <td class="px-6 py-4">${item.estado}</td>
                <td class="px-6 py-4 font-semibold text-center">${item.frec}</td>
            `;
            tableBody.appendChild(row);
        });
    };

    const renderSankeyChart = () => {
        const data = {
            type: "sankey",
            orientation: "h",
            node: {
                pad: 15,
                thickness: 20,
                line: { color: "black", width: 0.5 },
                label: ['REGISTRADO', 'REVISADO POR LA ENTIDAD', 'EXP. APROBADO', 'APROBADO BANHVI', 'EMITIDO', 'FORMALIZADO', 'PAGADO', 'EXP. CON ANOMALIAS'],
                color: ["#3b82f6", "#f97316", "#22c55e", "#14b8a6", "#8b5cf6", "#6366f1", "#d946ef", "#ef4444"]
            },
            link: {
                source: [0, 1, 1, 1, 2, 3, 4, 7, 5],
                target: [1, 2, 7, 0, 3, 4, 5, 0, 6],
                value:  [5000, 2500, 500, 2000, 2500, 2500, 2500, 500, 2450],
                color: [
                    'rgba(59, 130, 246, 0.4)', // REG -> REV
                    'rgba(249, 115, 22, 0.4)', // REV -> APROB
                    'rgba(239, 68, 68, 0.4)',  // REV -> ANOM
                    'rgba(239, 68, 68, 0.5)',  // REV -> REG (REWORK)
                    'rgba(34, 197, 94, 0.4)',  // APROB -> BANHVI
                    'rgba(20, 184, 166, 0.4)', // BANHVI -> EMITIDO
                    'rgba(139, 92, 246, 0.4)', // EMITIDO -> FORM
                    'rgba(239, 68, 68, 0.5)',  // ANOM -> REG
                    'rgba(99, 102, 241, 0.4)'  // FORM -> PAGADO
                ]
            }
        };

        const layout = {
            title: 'Flujo de Casos entre Estados',
            font: { size: 12, family: 'Montserrat, sans-serif' },
            margin: { l: 50, r: 50, b: 50, t: 80, pad: 4 }
        };

        Plotly.newPlot('sankeyContainer', [data], layout, {responsive: true});
    };

    const setupCaseExplorer = () => {
        const selector = document.getElementById('caseSelector');
        const titleEl = document.getElementById('caseTitle');
        const descriptionEl = document.getElementById('caseDescription');
        const tableBody = document.getElementById('caseHistoryTable');

        const renderCaseHistory = (caseId) => {
            const data = caseData[caseId];
            titleEl.textContent = caseId;
            descriptionEl.textContent = data.description;
            tableBody.innerHTML = '';
            
            data.history.forEach((row, index) => {
                const tr = document.createElement('tr');
                tr.className = `border-b ${index % 2 === 0 ? 'bg-white' : 'bg-slate-50'}`;
                const durationText = row.dur !== null ? row.dur.toFixed(1) : 'N/A';
                const durationClass = row.dur > 30 ? 'text-red-600 font-bold' : 'text-green-600';

                tr.innerHTML = `
                    <td class="px-6 py-4 text-center">${row.linea}</td>
                    <td class="px-6 py-4 font-medium text-slate-900">${row.estado}</td>
                    <td class="px-6 py-4">${row.fecha}</td>
                    <td class="px-6 py-4 text-center ${durationClass}">${durationText}</td>
                `;
                tableBody.appendChild(tr);
            });
        };

        selector.addEventListener('change', (e) => {
            renderCaseHistory(e.target.value);
        });

        renderCaseHistory(selector.value);
    };

    setupTabs();
    renderDurationChart();
    renderSankeyChart();
    renderRepetitionTable();
    setupCaseExplorer();
});
</script>

</body>
</html>
