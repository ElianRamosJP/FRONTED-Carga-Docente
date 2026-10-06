<script>
    import Navbar from '$lib/components/navbar.svelte';
    import Footer from '$lib/components/footer.svelte';
    import Header from '$lib/components/header.svelte';

    // Datos estáticos de docentes para evaluación
    const docentes = [
        { id: 1, nombre: 'Mariana Torres', facultad: 'Ingeniería' },
        { id: 2, nombre: 'Carlos Ruíz', facultad: 'Ciencias Económicas' },
        { id: 3, nombre: 'Sofía Herrera', facultad: 'Ciencias Sociales' }
    ];

    // Tipos de evaluación
    const tiposEvaluacion = [
        { id: 'par', nombre: 'Par Académico' },
        { id: 'auto', nombre: 'Autoevaluación' }
    ];

    // Criterios de evaluación agrupados por dimensión
    let dimensiones = $state([
        {
            titulo: '1. Dominio Disciplinar y Pedagogía',
            criterios: [
                { id: 1, nombre: 'Dominio de la materia', desc: 'Demuestra actualización y rigor conceptual en los temas impartidos.', nota: 0 },
                { id: 2, nombre: 'Claridad en la explicación', desc: 'Utiliza metodologías claras que facilitan la aprehensión del conocimiento.', nota: 0 },
                { id: 3, nombre: 'Uso de TICs y recursos', desc: 'Incorpora plataformas y herramientas digitales en el desarrollo de la clase.', nota: 0 }
            ]
        },
        {
            titulo: '2. Responsabilidad y Gestión Académica',
            criterios: [
                { id: 4, nombre: 'Puntualidad y asistencia', desc: 'Cumple rigurosamente con los horarios establecidos para las sesiones.', nota: 0 },
                { id: 5, nombre: 'Retroalimentación oportuna', desc: 'Entrega calificaciones e impresiones de trabajos en los tiempos institucionales.', nota: 0 }
            ]
        },
        {
            titulo: '3. Ética y Relación con Estudiantes',
            criterios: [
                { id: 6, nombre: 'Respeto y empatía', desc: 'Mantiene un trato digno, equitativo e inclusivo con toda la comunidad.', nota: 0 },
                { id: 7, nombre: 'Atención a dudas', desc: 'Muestra disposición para acompañar asesorías y responder consultas.', nota: 0 }
            ]
        }
    ]);

    // Estado inicial de campos
    let docenteSeleccionadoId = $state(1);
    let tipoSeleccionado = $state('par');
    let periodo = $state('2026-2');

    // Estado para la única justificación/observación general al final del cuestionario
    let observacionGeneral = $state('');
</script>

<svelte:head>
    <title>Evaluación de Desempeño | Acta Docente</title>
</svelte:head>

<Header />
<Navbar />

<main class="bg-light min-vh-100 p-4">
    <div class="container-fluid px-2 px-md-4">
        
        <!-- ENCABEZADO DE SECCIÓN -->
        <div class="card border-0 shadow-sm mb-4">
            <div class="card-body">
                <div class="d-flex align-items-center gap-3">
                    <div class="rounded p-3 text-white fs-4" style="background-color: #0f3460;">
                        📝
                    </div>
                    <div>
                        <h2 class="fw-bold mb-1">Evaluación de Desempeño Docente</h2>
                        <p class="text-muted mb-0">Formulario institucional de ponderación cualitativa y cuantitativa</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- SECCIÓN SUPERIOR: SELECCIÓN DE DOCENTE Y TIPO -->
        <div class="card border-0 shadow-sm mb-4">
            <div class="card-header text-white" style="background-color: #0f3460;">
                <h5 class="fw-bold mb-0"> Parámetros de la Evaluación</h5>
            </div>
            <div class="card-body">
                <div class="row g-3">
                    <!-- SELECCIÓN DOCENTE -->
                    <div class="col-md-5">
                        <label for="docenteSelect" class="form-label fw-semibold text-secondary">Docente a Evaluar</label>
                        <select id="docenteSelect" class="form-select form-select-lg" bind:value={docenteSeleccionadoId}>
                            {#each docentes as d}
                                <option value={d.id}>{d.nombre} — {d.facultad}</option>
                            {/each}
                        </select>
                    </div>

                    <!-- TIPO EVALUACIÓN -->
                    <div class="col-md-4">
                        <label for="tipoSelect" class="form-label fw-semibold text-secondary">Tipo de Evaluación</label>
                        <select id="tipoSelect" class="form-select form-select-lg" bind:value={tipoSeleccionado}>
                            {#each tiposEvaluacion as t}
                                <option value={t.id}>{t.nombre}</option>
                            {/each}
                        </select>
                    </div>

                    <!-- PERIODO ACADÉMICO -->
                    <div class="col-md-3">
                        <label for="periodoInput" class="form-label fw-semibold text-secondary">Periodo Activo</label>
                        <input id="periodoInput" type="text" class="form-control form-select-lg bg-white" value={periodo} readonly />
                    </div>
                </div>
            </div>
        </div>

        <!-- BANCO DE PREGUNTAS -->
        <form onsubmit={(e) => e.preventDefault()}>
            {#each dimensiones as dim}
                <div class="card border-0 shadow-sm mb-4">
                    <div class="card-header bg-white border-bottom py-3">
                        <h5 class="fw-bold mb-0 text-dark">
                            <span class="me-2">{dim.icono || ''}</span>{dim.titulo}
                        </h5>
                    </div>
                    <div class="card-body p-3 p-md-4">
                        <div class="d-flex flex-column gap-3">
                            {#each dim.criterios as criterio (criterio.id)}
                                <div class="p-3 rounded border bg-white shadow-sm">
                                    
                                    <!-- SOLAMENTE LAS PREGUNTAS / CRITERIOS MEJORADOS EN VISIBILIDAD -->
                                    <div class="d-flex flex-column flex-md-row justify-content-between align-items-md-start gap-2 mb-2">
                                        <div>
                                            <h5 class="fw-bold mb-1 fs-5" style="color: #111111;">
                                                {criterio.nombre}
                                            </h5>
                                            <p class="fs-6 fw-semibold mb-0" style="color: #333333; line-height: 1.4;">
                                                {criterio.desc}
                                            </p>
                                        </div>
                                    </div>

                                    <!-- ESCALA DE CALIFICACIÓN 1 A 5 -->
                                    <div class="mt-3">
                                        <div class="small fw-semibold text-secondary mb-2">
                                            Escala de calificación: <span class="fw-normal">1 = Deficiente · 5 = Excelente</span>
                                        </div>
                                        <div class="btn-group w-100" role="group" aria-label="Escala de calificación">
                                            {#each [1, 2, 3, 4, 5] as valor}
                                                <input
                                                    type="radio"
                                                    class="btn-check"
                                                    name={`criterio-${criterio.id}`}
                                                    id={`btn-${criterio.id}-${valor}`}
                                                    value={valor}
                                                    checked={criterio.nota === valor}
                                                    onchange={() => (criterio.nota = valor)}
                                                />
                                                <label
                                                    class="btn btn-outline-primary py-2 fw-semibold"
                                                    for={`btn-${criterio.id}-${valor}`}
                                                >
                                                    {valor}
                                                </label>
                                            {/each}
                                        </div>
                                    </div>
                                </div>
                            {/each}
                        </div>
                    </div>
                </div>
            {/each}

            <!-- SECCIÓN ÚNICA DE JUSTIFICACIÓN / OBSERVACIONES GENERALES -->
            <div class="card border-0 shadow-sm mb-4">
                <div class="card-header bg-white border-bottom py-3">
                    <h5 class="fw-bold mb-0 text-dark">
                        💬 Justificación y Observaciones Generales
                    </h5>
                </div>
                <div class="card-body">
                    <label for="observacionGeneral" class="form-label text-muted small">
                        Ingrese aquí cualquier comentario, soporte o justificación cualitativa sobre la evaluación realizada (opcional):
                    </label>
                    <textarea
                        id="observacionGeneral"
                        class="form-control"
                        rows="4"
                        placeholder="Escriba las observaciones generales de la evaluación..."
                        bind:value={observacionGeneral}
                    ></textarea>
                </div>
            </div>

            <!-- BOTONES PRINCIPALES DE ACCIÓN -->
            <div class="card border-0 shadow-sm mb-4">
                <div class="card-body d-flex flex-column flex-sm-row justify-content-end gap-2">
                    <button type="submit" class="btn text-white px-4 py-2 fw-semibold" style="background-color: #0f3460;">
                        Finalizar y Enviar Evaluación
                    </button>
                </div>
            </div>
        </form>

    </div>
</main>

<Footer />