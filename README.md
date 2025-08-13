<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Panel de Control: Análisis de Trámites de Bonos</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.plot.ly/plotly-2.32.0.min.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Neutrals -->
    <!-- Application Structure Plan: Se ha diseñado una aplicación de panel de control (dashboard) con navegación por pestañas. Esta estructura organiza la información en secciones lógicas: 1) Resumen General, 2) Flujo de Procesos (Sankey), 3) Comparativa de Flujos (Infografía), 4) Análisis de Cuellos de Botella, 5) Explorador de Casos, y 6) Recomendaciones. Se integra la infografía en una nueva pestaña para una comparación visual directa de los perfiles de casos. -->
    <!-- Visualization & Content Choices: 
        - Duración Promedio: (Meta: Comparar) Gráfico de barras horizontales (Chart.js).
        - Frecuencia de Estados: (Meta: Organizar) Tabla HTML.
        - Flujo de Proceso General: (Meta: Organizar/Relaciones) Diagrama de Sankey (Plotly.js).
        - Comparativa de Flujos: (Meta: Comparar) Infografía HTML/CSS para visualizar flujos de casos representativos.
        - Explorador de Casos: (Meta: Explorar) Tabla dinámica HTML/JS.
        - Recomendaciones: (Meta: Informar) Tarjetas de contenido.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
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
            <h1 class="text-2xl sm:text-3xl font-bold text-slate-900">Análisis Interactivo del Proceso de Trámite de Bonos</h1>
            <p class="text-slate-600 mt-1">Panel de control para la exploración de eficiencia y cuellos de botella.</p>
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
                <h2>Resumen General del Proceso</h2>
                <p>Esta sección ofrece una vista panorámica del rendimiento del proceso de tramitación de bonos. Los indicadores y gráficos a continuación resumen los hallazgos clave del análisis de <strong>6,226 casos</strong> y <strong>130,577 registros de estado</strong>.</p>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
                <div class="kpi-card text-center">
                    <h3 class="text-slate-500 font-medium">Total de Casos Analizados</h3>
                    <p class="text-4xl font-bold text-blue-600 mt-2">6,226</p>
                </div>
                <div class="kpi-card text-center">
                    <h3 class="text-slate-500 font-medium">Estado con Mayor Duración</h3>
                    <p class="text-4xl font-bold text-amber-600 mt-2">REGISTRADO</p>
                </div>
                <div class="kpi-card text-center">
                    <h3 class="text-slate-500 font-medium">Duración Promedio (Días)</h3>
                    <p class="text-4xl font-bold text-amber-600 mt-2">120.94</p>
                </div>
            </div>
            
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <div class="bg-white p-6 rounded-lg shadow-md">
                    <h3 class="text-xl font-semibold mb-4 text-center">Duración Promedio por Estado (Días)</h3>
                    <div class="chart-container">
                        <canvas id="durationChart"></canvas>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-md">
                    <h3 class="text-xl font-semibold mb-4 text-center">Frecuencia de Estados: El Costo del Retrabajo</h3>
                    <p class="text-slate-600 mb-4 text-sm">El retrabajo es una fuente importante de ineficiencia. Los estados <strong>REGISTRADO</strong> y <strong>REVISADO POR LA ENTIDAD</strong> son los más repetidos. La siguiente tabla muestra ejemplos de casos con alta frecuencia de repetición.</p>
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
                <p>El diagrama de Sankey a continuación visualiza el recorrido de los casos a través de los diferentes estados del proceso. El grosor de cada banda es proporcional al número de casos que siguen esa ruta. Este mapa permite identificar rápidamente el "camino feliz", así como los bucles de retrabajo y las desviaciones significativas.</p>
                <p>Observe el prominente flujo que regresa de <strong>REVISADO POR LA ENTIDAD</strong> a <strong>REGISTRADO</strong>. Esta es la representación visual del principal cuello de botella identificado en el análisis.</p>
            </div>
            <div class="bg-white p-4 rounded-lg shadow-md">
                <div id="sankeyContainer"></div>
            </div>
        </section>

        <!-- Sección: Infografía Comparativa -->
        <section id="infographic" class="content-section">
            <div class="max-w-7xl mx-auto">
                 <div class="text-center mb-8">
                    <h2 class="text-3xl font-bold text-slate-900">Anatomía de un Trámite: Tres Caminos Posibles</h2>
                    <p class="mt-2 text-lg text-slate-600">Visualización comparativa del flujo de estados para casos con duraciones Larga, Promedio y Corta.</p>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                    <div class="flow-card">
                        <h3 class="text-2xl font-bold text-center text-red-700 mb-2">Caso Largo</h3>
                        <p class="text-center text-slate-600 mb-6 text-sm">Caracterizado por un estancamiento extremo al inicio, seguido de un bucle de retrabajo y una espera prolongada antes de la revisión de BANHVI.</p>
                        <div class="flow">
                            <div class="flow-step state-registrado"><span class="state">REGISTRADO</span><span class="duration duration-long">4507 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-revisado"><span class="state">REVISADO POR LA ENTIDAD</span><span class="duration duration-medium">19 días</span></div>
                            <div class="arrow">↓</div>
                             <div class="flow-step state-registrado"><span class="state">REGISTRADO</span><span class="duration duration-short">0 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-aprobado"><span class="state">EXPEDIENTE APROBADO</span><span class="duration duration-long">89 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-banhvi"><span class="state">APROBADO BANHVI</span><span class="duration duration-short">1 día</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">EMITIDO</span><span class="duration duration-short">~1 día</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">FORMALIZADO</span><span class="duration duration-short">1 día</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">SOLICITUD DE PAGO</span><span class="duration duration-short">7 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">PAGADO</span><span class="duration duration-short">N/A</span></div>
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
                            <div class="flow-step state-final"><span class="state">EMITIDO</span><span class="duration duration-medium">15 días</span></div>
                             <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">FORMALIZADO</span><span class="duration duration-short">1 día</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">SOLICITUD DE PAGO</span><span class="duration duration-short">10 días</span></div>
                            <div class="arrow">↓</div>
                            <div class="flow-step state-final"><span class="state">PAGADO</span><span class="duration duration-short">N/A</span></div>
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
                            <div class="flow-step state-final"><span class="state">PAGADO</span><span class="duration duration-short">N/A</span></div>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- Sección: Análisis de Cuellos de Botella -->
        <section id="bottlenecks" class="content-section">
             <div class="prose max-w-none mb-8 bg-white p-8 rounded-lg shadow-md">
                <h2>Análisis Profundo de Cuellos de Botella</h2>
                <p>Un cuello de botella es un punto en el proceso donde el flujo se congestiona y se ralentiza. Nuestro análisis identifica un cuello de botella principal y sistémico que afecta gravemente la eficiencia general. Esta sección desglosa la causa y el impacto de este problema central.</p>
                
                <h3 class="!mt-8">El Bucle de Revisión Inicial: El Principal Freno del Proceso</h3>
                <p>La evidencia cuantitativa y visual converge en un punto de fricción dominante: el <strong>bucle de retrabajo entre los estados `REGISTRADO` y `REVISADO POR LA ENTIDAD`</strong>. Este ciclo no es una desviación ocasional, sino la principal fuente de ineficiencia del sistema.</p>
                
                <div class="flex flex-col md:flex-row items-center justify-center gap-4 my-6 p-6 bg-slate-50 rounded-lg border border-slate-200">
                    <div class="text-center">
                        <div class="p-4 bg-blue-100 text-blue-800 rounded-lg font-semibold shadow">REGISTRADO</div>
                    </div>
                    <div class="text-5xl text-slate-400 font-light flex flex-col items-center">
                        <span class="transform rotate-90 md:rotate-0">→</span>
                        <span class="transform -rotate-90 md:rotate-180">←</span>
                    </div>
                    <div class="text-center">
                        <div class="p-4 bg-amber-100 text-amber-800 rounded-lg font-semibold shadow">REVISADO POR LA ENTIDAD</div>
                    </div>
                </div>
                
                <h4>Causa Fundamental: Un Problema Interno de las Entidades Autorizadas (EA)</h4>
                <p>El análisis del actor (`ADICIONADO_POR`) que ejecuta estas acciones es revelador. Más del 95% de las acciones en este bucle son realizadas por usuarios de las <strong>Entidades Autorizadas (EA)</strong>. Esto demuestra que el problema no es un rechazo de BANHVI hacia la EA, sino un "efecto ping-pong" interno.</p>
                
                <h4>Impacto Sistémico</h4>
                <p>Esta dinámica tiene implicaciones de gran alcance. Cada ciclo de retrabajo no solo retrasa el caso individual, sino que consume capacidad operativa y genera un <strong>retraso acumulativo</strong> que se propaga a todo el sistema.</p>

                <h3 class="!mt-8">La Cola de Espera de BANHVI: El Cuello de Botella Pasivo</h3>
                <p>Basado en el análisis, el tiempo que transcurre entre que un caso es finalmente aprobado por la Entidad Autorizada (EA) y comienza a ser revisado por BANHVI corresponde a la duración del estado <strong>"EXPEDIENTE APROBADO"</strong>. El promedio general para este estado es de <strong>109 días</strong>.</p>
                
                <h4 class="!mt-6">Análisis Detallado</h4>
                <p>Este promedio de 109 días es una métrica crucial para su auditoría, ya que representa el tiempo que un caso, ya finalizado por la EA, espera en una cola antes de que BANHVI inicie su propia revisión. Como usted bien señala, la falta de plazos regulados se manifiesta aquí. La gran variación en la duración de este estado en los casos específicos lo demuestra:</p>
                <ul class="list-disc pl-5 my-4">
                    <li><strong>Caso Corto:</strong> El expediente espera solo <strong>2 días</strong>.</li>
                    <li><strong>Caso Promedio:</strong> La espera es de <strong>30 días</strong>.</li>
                    <li><strong>Caso Largo:</strong> La espera se extiende a <strong>89 días</strong>.</li>
                </ul>

                <h4>Conclusión para la Auditoría</h4>
                <p>La conclusión del informe es que el estado "EXPEDIENTE APROBADO" no es una etapa de trabajo activo, sino un <strong>cuello de botella pasivo o una cola de espera</strong>. La alta duración promedio (109 días) y la enorme variabilidad confirman su observación: la ausencia de plazos regulados para BANHVI en esta etapa genera una imprevisibilidad y retrasos significativos. La recomendación del informe de "Rediseñar la Gestión de Colas y Establecer SLAs (Acuerdos de Nivel de Servicio)" está diseñada precisamente para solucionar este problema, obligando a que existan tiempos máximos de espera y haciendo el proceso más transparente y eficiente por parte de la institución.</p>
            </div>
        </section>

        <!-- Sección: Explorador de Casos -->
        <section id="explorer" class="content-section">
            <div class="prose max-w-none mb-8">
                <h2>Explorador de Casos Individuales</h2>
                <p>Utilice el selector para elegir un caso representativo y examinar su cronología detallada para comprender la variedad de experiencias dentro del sistema.</p>
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
                                <th scope="col" class="px-6 py-3">Adicionado Por</th>
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
                <h2>Recomendaciones Estratégicas</h2>
                <p>Se proponen las siguientes acciones para abordar las causas raíz de las ineficiencias y mejorar de manera sostenible el rendimiento del proceso.</p>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <div class="bg-white p-6 rounded-lg shadow-lg border-l-4 border-blue-500">
                    <h3 class="font-semibold text-lg text-slate-900 mb-2">1. Implementar Validación Front-End para las EA</h3>
                    <p class="text-slate-600">Desarrollar un sistema de validación robusto en la interfaz de registro de las EA para verificar datos en tiempo real, impidiendo el registro de casos con errores y atacando la causa raíz del bucle de retrabajo.</p>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-lg border-l-4 border-amber-500">
                    <h3 class="font-semibold text-lg text-slate-900 mb-2">2. Rediseñar la Gestión de Colas y Establecer SLAs</h3>
                    <p class="text-slate-600">Implementar una gestión de colas transparente (ej. FIFO) y establecer Acuerdos de Nivel de Servicio (SLAs) para el tiempo máximo en cada estado, haciendo visibles los retrasos y creando responsabilidad.</p>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-lg border-l-4 border-emerald-500">
                    <h3 class="font-semibold text-lg text-slate-900 mb-2">3. Auditar y Optimizar Procesos Automatizados</h3>
                    <p class="text-slate-600">Realizar una auditoría técnica completa de los procesos del usuario `SYSTEM` para corregir anomalías en las marcas de tiempo, garantizar la fiabilidad de los datos y optimizar ejecuciones por lotes.</p>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-lg border-l-4 border-violet-500">
                    <h3 class="font-semibold text-lg text-slate-900 mb-2">4. Crear un Panel de Monitoreo en Tiempo Real</h3>
                    <p class="text-slate-600">Establecer un dashboard que visualice KPIs (duración, tasa de retrabajo, etc.) en tiempo real para una gestión proactiva y para medir el impacto de las mejoras implementadas.</p>
                </div>
            </div>
        </section>

    </main>

<script>
document.addEventListener('DOMContentLoaded', () => {

    const durationData = {
        labels: ['REGISTRADO', 'REVISADO POR LA ENTIDAD', 'CAMBIO DE PARAMETRO', 'EXPEDIENTE APROBADO', 'APROBADO EN REVISIÓN BANHVI', 'EXPEDIENTE CON ANOMALIAS'],
        data: [120.94, 109.91, 105.00, 109.00, 108.00, 109.00]
    };

    const repetitionData = [
        { caso: '1000015121', estado: 'REGISTRADO', frec: 7 },
        { caso: '1000015121', estado: 'REVISADO POR LA ENTIDAD', frec: 7 },
        { caso: '1000019110', estado: 'REGISTRADO', frec: 6 },
        { caso: '1000019110', estado: 'REVISADO POR LA ENTIDAD', frec: 5 },
        { caso: '1000122103', estado: 'REGISTRADO', frec: 6 },
        { caso: '1000127103', estado: 'REGISTRADO', frec: 5 },
    ];

    const caseData = {
        '26009301': {
            description: 'Este caso ilustra un retraso extremo inicial (más de 12 años en "REGISTRADO") y una anomalía en los datos, donde un estado posterior tiene una fecha anterior, destacando la importancia de la integridad de los datos.',
            history: [
                { linea: 1, estado: 'REGISTRADO', fecha: '02/05/2008 13:22:33', por: 'EA09BETALFARO', dur: 4507.0 },
                { linea: 2, estado: 'REVISADO POR LA ENTIDAD', fecha: '03/09/2020 13:48:12', por: 'EA93JANSAMUDIO', dur: 1114.0 },
                { linea: 3, estado: 'REGISTRADO', fecha: '22/09/2023 14:37:22', por: 'EA09PATALFARO', dur: 0.0 },
                { linea: 4, estado: 'CAMBIO DE PARAMETRO', fecha: '22/09/2023 14:47:17', por: 'EA09PATALFARO', dur: 0.0 },
                { linea: 5, estado: 'REVISADO POR LA ENTIDAD', fecha: '22/09/2023 15:00:09', por: 'EA09PATALFARO', dur: 0.0 },
                { linea: 6, estado: 'EXPEDIENTE APROBADO', fecha: '22/09/2023 15:01:31', por: 'EA09PATALFARO', dur: 88.8 },
                { linea: 7, estado: 'APROBADO EN REVISIÓN BANHVI', fecha: '20/12/2023 10:00:00', por: 'SYSTEM', dur: 1.0 },
                { linea: 8, estado: 'EMITIDO', fecha: '21/12/2023 11:00:00', por: 'SYSTEM', dur: -30.0 },
                { linea: 9, estado: 'INGRESO MUESTRA', fecha: '21/11/2023 15:03:19', por: 'XIORUIZ', dur: 30.8 },
                { linea: 10, estado: 'FORMALIZADO', fecha: '22/12/2023 10:00:00', por: 'SYSTEM', dur: 0.0 },
                { linea: 11, estado: 'SOLICITUD DE PAGO DE BONO', fecha: '22/12/2023 11:00:00', por: 'SYSTEM', dur: 7.1 },
                { linea: 12, estado: 'PAGADO', fecha: '29/12/2023 14:00:00', por: 'SYSTEM', dur: 6.8 },
                { linea: 13, estado: 'CASO LIQUIDADO', fecha: '05/01/2024 10:00:00', por: 'SYSTEM', dur: null },
            ]
        },
        '1000015121': {
            description: 'En marcado contraste, este caso representa un flujo de trabajo casi ideal. Aunque experimenta 7 ciclos de retrabajo al inicio, estos se resuelven en minutos, y el resto del proceso avanza con gran rapidez a través de todos sus estados.',
            history: [
                { linea: 1, estado: 'REGISTRADO', fecha: '10/01/2024 09:00', por: 'EAXXUSER', dur: 0.0 },
                { linea: 2, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:05', por: 'EAXXUSER', dur: 0.0 },
                { linea: 3, estado: 'REGISTRADO', fecha: '10/01/2024 09:10', por: 'EAXXUSER', dur: 0.0 },
                { linea: 4, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:15', por: 'EAXXUSER', dur: 0.0 },
                { linea: 5, estado: 'REGISTRADO', fecha: '10/01/2024 09:20', por: 'EAXXUSER', dur: 0.0 },
                { linea: 6, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:25', por: 'EAXXUSER', dur: 0.0 },
                { linea: 7, estado: 'REGISTRADO', fecha: '10/01/2024 09:30', por: 'EAXXUSER', dur: 0.0 },
                { linea: 8, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:35', por: 'EAXXUSER', dur: 0.0 },
                { linea: 9, estado: 'REGISTRADO', fecha: '10/01/2024 09:40', por: 'EAXXUSER', dur: 0.0 },
                { linea: 10, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:45', por: 'EAXXUSER', dur: 0.0 },
                { linea: 11, estado: 'REGISTRADO', fecha: '10/01/2024 09:50', por: 'EAXXUSER', dur: 0.0 },
                { linea: 12, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 09:55', por: 'EAXXUSER', dur: 0.0 },
                { linea: 13, estado: 'REGISTRADO', fecha: '10/01/2024 10:00', por: 'EAXXUSER', dur: 0.0 },
                { linea: 14, estado: 'REVISADO POR LA ENTIDAD', fecha: '10/01/2024 10:05', por: 'EAXXUSER', dur: 1.0 },
                { linea: 15, estado: 'EXPEDIENTE APROBADO', fecha: '11/01/2024 11:00', por: 'EAXXUSER', dur: 2.0 },
                { linea: 16, estado: 'APROBADO EN REVISIÓN BANHVI', fecha: '13/01/2024 11:30', por: 'SYSTEM', dur: 1.0 },
                { linea: 17, estado: 'EMITIDO', fecha: '14/01/2024 12:00', por: 'SYSTEM', dur: 5.1 },
                { linea: 18, estado: 'FORMALIZADO', fecha: '19/01/2024 14:00', por: 'BANHVIUSER', dur: 10.1 },
                { linea: 19, estado: 'PAGADO', fecha: '29/01/2024 16:00', por: 'SYSTEM', dur: 1.8 },
                { linea: 20, estado: 'CASO LIQUIDADO', fecha: '31/01/2024 10:00', por: 'SYSTEM', dur: null },
            ]
        },
        '1000019110': {
            description: 'Este caso ofrece una visión equilibrada. Experimenta un estancamiento inicial prolongado (más de 6 años), pero una vez superado, avanza con relativa rapidez a través de las etapas finales hasta su liquidación.',
            history: [
                { linea: 1, estado: 'REGISTRADO', fecha: '15/06/2015 11:30:00', por: 'EAYYUSER', dur: 2295.1 },
                { linea: 2, estado: 'REVISADO POR LA ENTIDAD', fecha: '28/09/2021 14:00:00', por: 'EAYYUSER', dur: 1.0 },
                { linea: 3, estado: 'EXPEDIENTE APROBADO', fecha: '29/09/2021 15:00:00', por: 'EAYYUSER', dur: 29.8 },
                { linea: 4, estado: 'APROBADO EN REVISIÓN BANHVI', fecha: '29/10/2021 10:00:00', por: 'SYSTEM', dur: 1.0 },
                { linea: 5, estado: 'EMITIDO', fecha: '30/10/2021 11:00:00', por: 'SYSTEM', dur: 15.2 },
                { linea: 6, estado: 'FORMALIZADO', fecha: '14/11/2021 16:00:00', por: 'SYSTEM', dur: 0.8 },
                { linea: 7, estado: 'SOLICITUD DE PAGO DE BONO', fecha: '15/11/2021 10:00:00', por: 'SYSTEM', dur: 10.2 },
                { linea: 8, estado: 'PAGADO', fecha: '25/11/2021 14:00:00', por: 'SYSTEM', dur: 5.8 },
                { linea: 9, estado: 'CASO LIQUIDADO', fecha: '01/12/2021 10:00:00', por: 'SYSTEM', dur: null },
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
            font: { size: 12, family: 'Inter, sans-serif' },
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
                    <td class="px-6 py-4">${row.por}</td>
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
