<script>
    import { onMount } from 'svelte';
    import Swal from 'sweetalert2';

    import Header from '../../lib/components/header.svelte';
    import Navbar from '../../lib/components/navbar.svelte';
    import Footer from '../../lib/components/footer.svelte';

    const API_URL = 'https://api-carga-e4od.onrender.com';

    // =========================================================
    // DATOS (Svelte 5 Runes)
    // =========================================================
    let docentes = $state([]);
    let asignaturas = $state([]);
    let grupos = $state([]);
    let periodos = $state([]);
    let cargas = $state([]);
    let horarios = $state([]);

    let cargando = $state(false);
    let cargandoCatalogos = $state(false);
    let guardando = $state(false);

    let error = $state('');
    let editando = $state(false);

    // =========================================================
    // PAGINACIÓN Y FILTRADO (Svelte 5 Runes)
    // =========================================================
    let busqueda = $state('');
    let paginaActual = $state(1);
    let itemsPorPagina = $state(5);

    // =========================================================
    // FORMULARIO
    // =========================================================
    let form = $state({
        carga_id: null,
        user_id: '',
        periodo_id: '',
        id_grupo: '',
        estado: true,

        dia_laborales: '1',
        hora_inicio: '08:00',
        hora_finalizar: '10:00'
    });

    onMount(() => {
        cargarTodo();
    });

    async function cargarTodo() {
        await cargarCatalogos();
        await cargarCargas();
        await cargarHorarios();
    }

    async function obtenerDatos(response) {
        let data;
        try {
            data = await response.json();
        } catch {
            throw new Error(`HTTP ${response.status}`);
        }

        if (!response.ok) {
            throw new Error(data?.detail || data?.mensaje || `HTTP ${response.status}`);
        }

        if (Array.isArray(data)) return data;
        if (Array.isArray(data?.data)) return data.data;
        return data;
    }

    // =========================================================
    // PETICIONES API
    // =========================================================
    async function cargarCatalogos() {
        cargandoCatalogos = true;
        error = '';

        try {
            const [resUsuarios, resAsignaturas, resGrupos, resPeriodos] = await Promise.all([
                fetch(`${API_URL}/usuarios/`),
                fetch(`${API_URL}/asignaturas/`),
                fetch(`${API_URL}/grupos/`),
                fetch(`${API_URL}/periodos-academicos/`)
            ]);

            docentes = await obtenerDatos(resUsuarios);
            asignaturas = await obtenerDatos(resAsignaturas);
            grupos = await obtenerDatos(resGrupos);
            periodos = await obtenerDatos(resPeriodos);

            if (!Array.isArray(docentes)) docentes = [];
            if (!Array.isArray(asignaturas)) asignaturas = [];
            if (!Array.isArray(grupos)) grupos = [];
            if (!Array.isArray(periodos)) periodos = [];

        } catch (e) {
            console.error('Error cargando catálogos:', e);
            error = `No se pudieron cargar los catálogos: ${e.message}`;
            Swal.fire({ icon: 'error', title: 'Error', text: error });
        } finally {
            cargandoCatalogos = false;
        }
    }

    async function cargarCargas() {
        cargando = true;
        error = '';

        try {
            const response = await fetch(`${API_URL}/carga-docente/`);
            const data = await obtenerDatos(response);
            cargas = Array.isArray(data) ? data : [];
        } catch (e) {
            console.error('Error cargando cargas:', e);
            error = `No se pudieron cargar las cargas docentes. ${e.message}`;
            Swal.fire({ icon: 'error', title: 'Error', text: error });
        } finally {
            cargando = false;
        }
    }

    async function cargarHorarios() {
        try {
            const response = await fetch(`${API_URL}/horarios/`);
            if (!response.ok) {
                horarios = [];
                return;
            }
            const data = await response.json();

            if (Array.isArray(data)) {
                horarios = data;
            } else if (Array.isArray(data?.data)) {
                horarios = data.data;
            } else {
                horarios = [];
            }
        } catch (e) {
            console.warn('No se pudieron cargar los horarios:', e);
            horarios = [];
        }
    }

    // =========================================================
    // HELPERS & BÚSQUEDAS
    // =========================================================
    function obtenerDocente(userId) {
        return docentes.find(d => Number(d.usuario_id ?? d.user_id ?? d.id) === Number(userId));
    }

    function nombreDocente(userId) {
        const docente = obtenerDocente(userId);
        if (!docente) return `Usuario ${userId ?? '-'}`;

        return (
            docente.nombre_completo ||
            docente.nombre ||
            docente.nombres ||
            `${docente.nombre_usuario || ''} ${docente.apellido || ''}`.trim() ||
            docente.email ||
            `Usuario ${userId}`
        );
    }

    function obtenerGrupo(idGrupo) {
        return grupos.find(g => Number(g.grupos_id ?? g.id_grupo ?? g.id) === Number(idGrupo));
    }

    function nombreGrupo(idGrupo) {
        const grupo = obtenerGrupo(idGrupo);
        if (!grupo) return `Grupo ${idGrupo ?? '-'}`;

        return grupo.nombre_grupos || grupo.nombre || grupo.grupo || grupo.codigo || `Grupo ${idGrupo}`;
    }

    function obtenerAsignatura(idAsignatura) {
        return asignaturas.find(a => Number(a.asignatura_id ?? a.id) === Number(idAsignatura));
    }

    function nombreAsignatura(idAsignatura) {
        const asignatura = obtenerAsignatura(idAsignatura);
        if (!asignatura) return 'Asignatura no encontrada';

        return asignatura.nombres_asignaturas || asignatura.nombre || asignatura.nombre_asignatura || 'Sin nombre';
    }

    function nombrePeriodo(periodoId) {
        const periodo = periodos.find(p => Number(p.periodo_id ?? p.id) === Number(periodoId));
        if (!periodo) return `Período ${periodoId ?? '-'}`;

        return periodo.codigo_periodo || periodo.nombre || periodo.periodo || periodo.descripcion || `Período ${periodoId}`;
    }

    function nombreDia(dia) {
        const dias = { 1: 'Lunes', 2: 'Martes', 3: 'Miércoles', 4: 'Jueves', 5: 'Viernes', 6: 'Sábado', 7: 'Domingo' };
        return dias[Number(dia)] || 'Sin día';
    }

    function cargarHora(hora) {
        if (!hora) return '';
        if (typeof hora === 'string') return hora.substring(0, 5);
        return hora;
    }

    function obtenerHorario(cargaId) {
        if (!cargaId) return null;
        return horarios.find(h => Number(h.carga_id ?? h.id_carga) === Number(cargaId));
    }

    // =========================================================
    // LÓGICA DE DERIVADOS DE PAGINACIÓN (Svelte 5 $derived)
    // =========================================================
    let cargasFiltradas = $derived(
        cargas.filter(c => {
            const query = busqueda.toLowerCase().trim();
            if (!query) return true;

            const doc = nombreDocente(c.user_id ?? c.usuario_id).toLowerCase();
            const grp = nombreGrupo(c.id_grupo ?? c.grupos_id).toLowerCase();
            const per = nombrePeriodo(c.periodo_id).toLowerCase();

            const grupoObj = obtenerGrupo(c.id_grupo ?? c.grupos_id);
            const asig = grupoObj?.asignaturas_id ? nombreAsignatura(grupoObj.asignaturas_id).toLowerCase() : '';

            return doc.includes(query) || grp.includes(query) || per.includes(query) || asig.includes(query);
        })
    );

    let totalPaginas = $derived(Math.ceil(cargasFiltradas.length / itemsPorPagina) || 1);

    let cargasPaginadas = $derived(
        cargasFiltradas.slice(
            (paginaActual - 1) * itemsPorPagina,
            paginaActual * itemsPorPagina
        )
    );

    function cambiarPagina(nuevaPagina) {
        if (nuevaPagina >= 1 && nuevaPagina <= totalPaginas) {
            paginaActual = nuevaPagina;
        }
    }

    // =========================================================
    // ACCIONES (GUARDAR, EDITAR, ELIMINAR)
    // =========================================================
    async function guardarCarga() {
        error = '';

        if (!form.user_id) return Swal.fire({ icon: 'warning', title: 'Falta el docente', text: 'Selecciona un docente.' });
        if (!form.periodo_id) return Swal.fire({ icon: 'warning', title: 'Falta el período', text: 'Selecciona un período académico.' });
        if (!form.id_grupo) return Swal.fire({ icon: 'warning', title: 'Falta el grupo', text: 'Selecciona un grupo.' });

        if (form.hora_inicio && form.hora_finalizar && form.hora_inicio >= form.hora_finalizar) {
            return Swal.fire({ icon: 'warning', title: 'Horario inválido', text: 'La hora final debe ser mayor que la hora inicial.' });
        }

        guardando = true;

        try {
            const datos = {
                user_id: Number(form.user_id),
                periodo_id: Number(form.periodo_id),
                id_grupo: Number(form.id_grupo),
                estado: Boolean(form.estado)
            };

            let response;
            if (editando && form.carga_id) {
                response = await fetch(`${API_URL}/carga-docente/${form.carga_id}`, {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(datos)
                });
            } else {
                response = await fetch(`${API_URL}/carga-docente/`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(datos)
                });
            }

            let resultado = {};
            try { resultado = await response.json(); } catch { resultado = {}; }

            if (!response.ok) {
                throw new Error(resultado?.detail || resultado?.mensaje || `HTTP ${response.status}`);
            }

            const cargaId = form.carga_id || resultado?.id || resultado?.carga_id || resultado?.data?.carga_id || resultado?.data?.id;

            if (cargaId) {
                const horarioExistente = obtenerHorario(cargaId);
                const payloadHorario = {
                    carga_id: Number(cargaId),
                    dia_laborales: Number(form.dia_laborales),
                    hora_inicio: form.hora_inicio,
                    hora_finalizar: form.hora_finalizar
                };

                try {
                    if (horarioExistente) {
                        const horarioId = horarioExistente.horario_id ?? horarioExistente.id;
                        await fetch(`${API_URL}/horarios/${horarioId}`, {
                            method: 'PUT',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payloadHorario)
                        });
                    } else {
                        await fetch(`${API_URL}/horarios/`, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payloadHorario)
                        });
                    }
                } catch (eHorario) {
                    console.warn('Error gestionando horario:', eHorario);
                }
            }

            await Swal.fire({
                icon: 'success',
                title: editando ? 'Carga actualizada' : 'Carga registrada',
                text: editando ? 'La carga docente fue actualizada correctamente.' : 'La carga docente fue registrada correctamente.',
                timer: 1800,
                showConfirmButton: false
            });

            limpiarFormulario();
            await cargarCargas();
            await cargarHorarios();

        } catch (e) {
            console.error(e);
            error = e.message;
            await Swal.fire({ icon: 'error', title: 'No se pudo guardar', text: e.message });
        } finally {
            guardando = false;
        }
    }

    function editarCarga(carga) {
        editando = true;
        const idCargaActual = carga.carga_id ?? carga.id ?? null;

        form.carga_id = idCargaActual;
        form.user_id = carga.user_id ?? carga.usuario_id ?? '';
        form.periodo_id = carga.periodo_id ?? '';
        form.id_grupo = carga.id_grupo ?? carga.grupos_id ?? '';
        form.estado = carga.estado !== undefined ? Boolean(carga.estado) : true;

        const horario = obtenerHorario(idCargaActual);

        if (horario) {
            form.dia_laborales = String(horario.dia_laborales ?? horario.dia ?? 1);
            form.hora_inicio = cargarHora(horario.hora_inicio) || '08:00';
            form.hora_finalizar = cargarHora(horario.hora_finalizar) || '10:00';
        } else {
            form.dia_laborales = '1';
            form.hora_inicio = '08:00';
            form.hora_finalizar = '10:00';
        }

        window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    async function eliminarCarga(cargaId) {
        if (!cargaId) return;

        const confirmacion = await Swal.fire({
            icon: 'warning',
            title: '¿Eliminar carga?',
            text: 'Esta acción no se puede deshacer.',
            showCancelButton: true,
            confirmButtonColor: '#d33',
            confirmButtonText: 'Sí, eliminar',
            cancelButtonText: 'Cancelar'
        });

        if (!confirmacion.isConfirmed) return;

        try {
            const response = await fetch(`${API_URL}/carga-docente/${cargaId}`, { method: 'DELETE' });
            let resultado = {};
            try { resultado = await response.json(); } catch { resultado = {}; }

            if (!response.ok) {
                throw new Error(resultado?.detail || resultado?.mensaje || `HTTP ${response.status}`);
            }

            await Swal.fire({
                icon: 'success',
                title: 'Eliminada',
                text: 'La carga docente fue eliminada correctamente.',
                timer: 1500,
                showConfirmButton: false
            });

            await cargarCargas();
            await cargarHorarios();
        } catch (e) {
            console.error(e);
            await Swal.fire({ icon: 'error', title: 'Error', text: e.message });
        }
    }

    function limpiarFormulario() {
        editando = false;
        form.carga_id = null;
        form.user_id = '';
        form.periodo_id = '';
        form.id_grupo = '';
        form.estado = true;
        form.dia_laborales = '1';
        form.hora_inicio = '08:00';
        form.hora_finalizar = '10:00';
    }

    let gruposOptions = $derived(Array.isArray(grupos) ? grupos : []);
</script>

<svelte:head>
    <title>Carga Docente | Acta Docente</title>
</svelte:head>

<Header />
<Navbar />

<main class="container-fluid p-4 bg-light min-vh-100">

    <!-- ENCABEZADO -->
    <div class="card border-0 shadow-sm mb-4">
        <div class="card-body d-flex flex-wrap align-items-center justify-content-between gap-3">
            <div class="d-flex align-items-center gap-3">
                <div 
                    class="rounded p-3 text-white fs-4 d-flex align-items-center justify-content-center"
                    style="background-color: #0f3460; width: 50px; height: 50px;"
                >
                    👨‍🏫  
                </div>
                <div>
                    <h2 class="mb-0 fw-bold">Carga Docente</h2>
                    <small class="text-muted">Gestión de asignación de docentes, grupos y horarios</small>
                </div>
            </div>

            <button
                class="btn btn-outline-secondary"
                type="button"
                onclick={cargarTodo}
                disabled={cargando || cargandoCatalogos}
            >
                ↻ Actualizar
            </button>
        </div>
    </div>

    <!-- ERROR -->
    {#if error}
        <div class="alert alert-danger shadow-sm d-flex justify-content-between align-items-center mb-4">
            <div>⚠️ <strong>Error:</strong> {error}</div>
            <button class="btn btn-sm btn-danger" onclick={() => error = ''}>Cerrar</button>
        </div>
    {/if}

    <div class="row g-4">
        <!-- FORMULARIO -->
        <div class="col-lg-4">
            <div class="card border-0 shadow-sm">
                <div class="card-header text-white d-flex justify-content-between align-items-center" style="background-color: #0f3460;">
                    <h5 class="mb-0 fw-bold fs-6">
                        {editando ? '✏️ Editar Carga Docente' : '+ Registrar Carga Docente'}
                    </h5>
                    {#if editando}
                        <span class="badge bg-warning text-dark">Modo Edición</span>
                    {/if}
                </div>

                <div class="card-body">
                    {#if cargandoCatalogos}
                        <div class="text-center py-4">
                            <div class="spinner-border text-primary" role="status"></div>
                            <p class="mt-2 text-muted">Cargando catálogos...</p>
                        </div>
                    {:else}
                        <form onsubmit={(e) => { e.preventDefault(); guardarCarga(); }}>
                            <!-- DOCENTE -->
                            <div class="mb-3">
                                <label for="selectDocente" class="form-label fw-semibold text-secondary">Docente</label>
                                <select id="selectDocente" class="form-select" bind:value={form.user_id} required>
                                    <option value="">Seleccione un docente...</option>
                                    {#each docentes as docente}
                                        {@const idDoc = docente.usuario_id ?? docente.user_id ?? docente.id}
                                        <option value={idDoc}>{nombreDocente(idDoc)}</option>
                                    {/each}
                                </select>
                            </div>

                            <!-- PERÍODO -->
                            <div class="mb-3">
                                <label for="selectPeriodo" class="form-label fw-semibold text-secondary">Período Académico</label>
                                <select id="selectPeriodo" class="form-select" bind:value={form.periodo_id} required>
                                    <option value="">Seleccione un período...</option>
                                    {#each periodos as periodo}
                                        {@const idPer = periodo.periodo_id ?? periodo.id}
                                        <option value={idPer}>{periodo.codigo_periodo}</option>
                                    {/each}
                                </select>
                            </div>

                            <!-- GRUPO -->
                            <div class="mb-3">
                                <label for="selectGrupo" class="form-label fw-semibold text-secondary">Grupo</label>
                                <select id="selectGrupo" class="form-select" bind:value={form.id_grupo} required>
                                    <option value="">Seleccione un grupo...</option>
                                    {#each gruposOptions as grupo}
                                        {@const idGrp = grupo.grupos_id ?? grupo.id_grupo ?? grupo.id}
                                        <option value={idGrp}>{nombreGrupo(idGrp)}</option>
                                    {/each}
                                </select>
                            </div>

                            <!-- ESTADO -->
                            <div class="mb-3">
                                <label for="selectEstado" class="form-label fw-semibold text-secondary">Estado</label>
                                <select
                                    id="selectEstado"
                                    class="form-select"
                                    value={form.estado ? 'true' : 'false'}
                                    onchange={(e) => form.estado = e.currentTarget.value === 'true'}
                                >
                                    <option value="true">Activo</option>
                                    <option value="false">Inactivo</option>
                                </select>
                            </div>

                            <hr class="my-3" />
                            <h6 class="fw-bold text-dark mb-3">🕒 Configuración de Horario</h6>

                            <!-- DÍA -->
                            <div class="mb-3">
                                <label for="selectDia" class="form-label fw-semibold text-secondary">Día Laboral</label>
                                <select id="selectDia" class="form-select" bind:value={form.dia_laborales}>
                                    <option value="1">Lunes</option>
                                    <option value="2">Martes</option>
                                    <option value="3">Miércoles</option>
                                    <option value="4">Jueves</option>
                                    <option value="5">Viernes</option>
                                    <option value="6">Sábado</option>
                                    <option value="7">Domingo</option>
                                </select>
                            </div>

                            <!-- HORAS -->
                            <div class="row g-2 mb-3">
                                <div class="col-6">
                                    <label for="horaInicio" class="form-label fw-semibold text-secondary">Hora Inicio</label>
                                    <input id="horaInicio" type="time" class="form-control" bind:value={form.hora_inicio} required />
                                </div>
                                <div class="col-6">
                                    <label for="horaFinal" class="form-label fw-semibold text-secondary">Hora Final</label>
                                    <input id="horaFinal" type="time" class="form-control" bind:value={form.hora_finalizar} required />
                                </div>
                            </div>

                            <!-- BOTONES -->
                            <div class="d-flex gap-2 mt-4">
                                <button
                                    type="submit"
                                    class="btn text-white flex-grow-1 fw-semibold"
                                    style="background-color: #0f3460;"
                                    disabled={guardando}
                                >
                                    {#if guardando}
                                        <span class="spinner-border spinner-border-sm me-2"></span> Guardando...
                                    {:else if editando}
                                        Actualizar Carga
                                    {:else}
                                        + Registrar Carga
                                    {/if}
                                </button>

                                <button
                                    type="button"
                                    class="btn btn-outline-secondary"
                                    onclick={limpiarFormulario}
                                    disabled={guardando}
                                >
                                    {editando ? 'Cancelar' : 'Limpiar'}
                                </button>
                            </div>
                        </form>
                    {/if}
                </div>
            </div>
        </div>

        <!-- TABLA Y CONTROLES DE PAGINACIÓN -->
        <div class="col-lg-8">
            <div class="card border-0 shadow-sm">
                <div class="card-body">
                    
                    <!-- ENCABEZADO DE TABLA & CONTROLES -->
                    <div class="d-flex flex-wrap justify-content-between align-items-center gap-3 mb-3">
                        <div>
                            <h4 class="mb-0 fw-bold">Cargas Docentes Registradas</h4>
                            <small class="text-muted">Mostrando {cargasPaginadas.length} de {cargasFiltradas.length} registros</small>
                        </div>

                        <span class="badge text-white fs-6 px-3 py-2" style="background-color: #0f3460;">
                            Total: {cargas.length}
                        </span>
                    </div>

                    <!-- BARRA DE BÚSQUEDA Y SELECTOR DE REGISTROS -->
                    <div class="row g-2 mb-3">
                        <div class="col-md-8">
                            <input
                                type="text"
                                class="form-control"
                                placeholder="🔍 Buscar por docente, grupo, período o asignatura..."
                                bind:value={busqueda}
                                oninput={() => paginaActual = 1}
                            />
                        </div>
                        <div class="col-md-4 d-flex align-items-center justify-content-end gap-2">
                            <label for="selectItems" class="text-muted small fw-semibold text-nowrap">Mostrar:</label>
                            <select
                                id="selectItems"
                                class="form-select form-select-sm style-select"
                                style="width: 80px;"
                                bind:value={itemsPorPagina}
                                onchange={() => paginaActual = 1}
                            >
                                <option value={5}>5</option>
                                <option value={10}>10</option>
                                <option value={20}>20</option>
                                <option value={50}>50</option>
                            </select>
                        </div>
                    </div>

                    <!-- TABLA -->
                    <div class="table-responsive">
                        <table class="table table-hover align-middle">
                            <thead class="table-light">
                                <tr>
                                    <th>ID</th>
                                    <th>Docente</th>
                                    <th>Período</th>
                                    <th>Grupo / Asignatura</th>
                                    <th>Horario</th>
                                    <th class="text-center">Estado</th>
                                    <th class="text-center">Acciones</th>
                                </tr>
                            </thead>
                            <tbody>
                                {#if cargando}
                                    <tr>
                                        <td colspan="7" class="text-center py-5">
                                            <div class="spinner-border text-primary" role="status"></div>
                                            <p class="text-muted mt-2 mb-0">Cargando cargas docentes...</p>
                                        </td>
                                    </tr>
                                {:else if cargasPaginadas.length === 0}
                                    <tr>
                                        <td colspan="7" class="text-center py-4 text-muted">
                                            {#if busqueda}
                                                No se encontraron resultados para "{busqueda}".
                                            {:else}
                                                No se encontraron cargas docentes registradas.
                                            {/if}
                                        </td>
                                    </tr>
                                {:else}
                                    {#each cargasPaginadas as carga}
                                        {@const idCarga = carga.carga_id ?? carga.id}
                                        {@const idUsr = carga.user_id ?? carga.usuario_id}
                                        {@const idGrp = carga.id_grupo ?? carga.grupos_id}
                                        {@const grupoObj = obtenerGrupo(idGrp)}
                                        {@const horario = obtenerHorario(idCarga)}

                                        <tr>
                                            <td class="text-muted">#{idCarga}</td>
                                            <td>
                                                <div class="fw-semibold text-dark">{nombreDocente(idUsr)}</div>
                                                <small class="text-muted">ID: {idUsr ?? '-'}</small>
                                            </td>
                                            <td>
                                                <span class="badge text-bg-secondary">{nombrePeriodo(carga.periodo_id)}</span>
                                            </td>
                                            <td>
                                                <div class="fw-semibold">{nombreGrupo(idGrp)}</div>
                                                {#if grupoObj?.asignaturas_id}
                                                    <small class="text-muted">
                                                        Materia: {nombreAsignatura(grupoObj.asignaturas_id)}
                                                    </small>
                                                {/if}
                                            </td>
                                            <td>
                                                {#if horario}
                                                    <div class="fw-semibold text-dark">
                                                        {nombreDia(horario.dia_laborales ?? horario.dia)}
                                                    </div>
                                                    <small class="text-muted">
                                                        {cargarHora(horario.hora_inicio)} - {cargarHora(horario.hora_finalizar)}
                                                    </small>
                                                {:else}
                                                    <span class="text-muted small">Por definir</span>
                                                {/if}
                                            </td>
                                            <td class="text-center">
                                                {#if carga.estado}
                                                    <span class="badge bg-success-subtle text-success">Activo</span>
                                                {:else}
                                                    <span class="badge bg-secondary-subtle text-secondary">Inactivo</span>
                                                {/if}
                                            </td>
                                            <td class="text-center">
                                                <div class="d-flex justify-content-center gap-1">
                                                    <button
                                                        type="button"
                                                        class="btn btn-sm btn-outline-primary fw-semibold px-2"
                                                        title="Editar"
                                                        onclick={() => editarCarga(carga)}
                                                    >
                                                        Editar
                                                    </button>
                                                    <button
                                                        type="button"
                                                        class="btn btn-sm btn-outline-danger fw-semibold px-2"
                                                        title="Eliminar"
                                                        onclick={() => eliminarCarga(idCarga)}
                                                    >
                                                        Eliminar
                                                    </button>
                                                </div>
                                            </td>
                                        </tr>
                                    {/each}
                                {/if}
                            </tbody>
                        </table>
                    </div>

                    <!-- NAVEGACIÓN DE PAGINACIÓN -->
                    {#if totalPaginas > 1}
                        <div class="d-flex justify-content-between align-items-center mt-3 pt-2 border-top">
                            <small class="text-muted">
                                Página <strong>{paginaActual}</strong> de <strong>{totalPaginas}</strong>
                            </small>

                            <nav>
                                <ul class="pagination pagination-sm mb-0">
                                    <li class="page-item {paginaActual === 1 ? 'disabled' : ''}">
                                        <button class="page-item page-link" onclick={() => cambiarPagina(paginaActual - 1)}>
                                            Anterior
                                        </button>
                                    </li>

                                    {#each Array.from({ length: totalPaginas }, (_, i) => i + 1) as i}
                                        <li class="page-item {paginaActual === i ? 'active' : ''}">
                                            <button 
                                                class="page-link" 
                                                style={paginaActual === i ? 'background-color: #0f3460; border-color: #0f3460;' : ''}
                                                onclick={() => cambiarPagina(i)}
                                            >
                                                {i}
                                            </button>
                                        </li>
                                    {/each}

                                    <li class="page-item {paginaActual === totalPaginas ? 'disabled' : ''}">
                                        <button class="page-link" onclick={() => cambiarPagina(paginaActual + 1)}>
                                            Siguiente
                                        </button>
                                    </li>
                                </ul>
                            </nav>
                        </div>
                    {/if}

                </div>
            </div>
        </div>
    </div>
</main>

<Footer />