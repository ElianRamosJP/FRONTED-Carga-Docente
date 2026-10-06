<script>
    import { onMount } from 'svelte';
    import Swal from 'sweetalert2';

    import Navbar from '../../lib/components/navbar.svelte';
    import Footer from '../../lib/components/footer.svelte';
    import Header from '../../lib/components/header.svelte';

    const API_URL = 'https://api-carga-e4od.onrender.com';

    // =========================================================
    // DATOS (Svelte 5 Runes)
    // =========================================================
    let asignaturas = $state([]);
    let programas = $state([]);
    let facultades = $state([]);

    let cargando = $state(true);
    let guardando = $state(false);
    let error = $state('');

    let busqueda = $state('');
    let filtroFacultad = $state('Todas');
    let filtroPrograma = $state('Todos');

    let idEditando = $state(null);

    // Estados para Paginación
    let paginaActual = $state(1);
    let porPagina = $state(10);

    // =========================================================
    // FORMULARIO
    // =========================================================
    let form = $state({
        codigo_asignatura: '',
        nombres_asignaturas: '',
        programa_id: '',
        creditos: 3,
        estado: true
    });

    onMount(async () => {
        await cargarDatos();
    });

    async function cargarDatos() {
        cargando = true;
        error = '';

        try {
            const [asignaturasRes, programasRes, facultadesRes] = await Promise.all([
                fetch(`${API_URL}/asignaturas/`),
                fetch(`${API_URL}/programas-academicos/`),
                fetch(`${API_URL}/facultades/`)
            ]);

            if (!asignaturasRes.ok) throw new Error(`Error en asignaturas: ${asignaturasRes.status}`);
            if (!programasRes.ok) throw new Error(`Error en programas: ${programasRes.status}`);
            if (!facultadesRes.ok) throw new Error(`Error en facultades: ${facultadesRes.status}`);

            const asignaturasData = await asignaturasRes.json();
            const programasData = await programasRes.json();
            const facultadesData = await facultadesRes.json();

            asignaturas = Array.isArray(asignaturasData) ? asignaturasData : (asignaturasData.data || []);
            programas = Array.isArray(programasData) ? programasData : (programasData.data || []);
            facultades = Array.isArray(facultadesData) ? facultadesData : (facultadesData.data || []);

        } catch (err) {
            console.error('Error cargando datos:', err);
            error = err.message || 'Error de conexión';
        } finally {
            cargando = false;
        }
    }

    // =========================================================
// OBTENER PROGRAMA
// =========================================================
function obtenerPrograma(programaId) {
    if (!programaId) return null;
    return programas.find(p => 
        Number(p.programa_id || p.id) === Number(programaId)
    );
}

// =========================================================
// OBTENER NOMBRE DEL PROGRAMA
// =========================================================
function nombrePrograma(programaId) {
    const programa = obtenerPrograma(programaId);
    return programa?.nombre_programa || programa?.nombre || 'Sin programa';
}

// =========================================================
// OBTENER FACULTAD (CORREGIDA)
// =========================================================
function nombreFacultad(programaId) {
    const programa = obtenerPrograma(programaId);

    if (!programa) {
        return 'Sin facultad';
    }

    // Extraemos la llave foránea hacia facultades
    const idFacultad = programa.facultades_id || programa.facultad_id || programa.id_facultad;

    if (!idFacultad) {
        return 'Sin facultad';
    }

    // Buscamos la facultad comparando IDs sin importar el tipo
    const facultad = facultades.find(f => 
        Number(f.facultades_id || f.facultad_id || f.id) === Number(idFacultad)
    );

    // Retornamos el nombre según la columna real de Neon/FastAPI
    return facultad?.nombre_facultades || facultad?.nombre_facultad || facultad?.nombre || 'Sin facultad';
}

    // =========================================================
    // FILTROS Y PAGINACIÓN ($derived)
    // =========================================================
    let asignaturasFiltradas = $derived(
        asignaturas.filter(a => {
            const texto = busqueda.toLowerCase().trim();
            const nombre = (a.nombres_asignaturas || '').toLowerCase();
            const codigo = (a.codigo_asignatura || '').toLowerCase();

            const coincideTexto = !texto || nombre.includes(texto) || codigo.includes(texto);
            const programa = nombrePrograma(a.programa_id);
            const facultad = nombreFacultad(a.programa_id);

            const coincidePrograma = filtroPrograma === 'Todos' || programa === filtroPrograma;
            const coincideFacultad = filtroFacultad === 'Todas' || facultad === filtroFacultad;

            return coincideTexto && coincidePrograma && coincideFacultad;
        })
    );

    let programasFiltrados = $derived(
        filtroFacultad === 'Todas'
            ? programas
            : programas.filter(p => nombreFacultad(p.programa_id) === filtroFacultad)
    );

    let totalPaginas = $derived(Math.ceil(asignaturasFiltradas.length / porPagina) || 1);

    let asignaturasPaginadas = $derived(
        asignaturasFiltradas.slice((paginaActual - 1) * porPagina, paginaActual * porPagina)
    );

    function cambiarPagina(nuevaPagina) {
        if (nuevaPagina >= 1 && nuevaPagina <= totalPaginas) {
            paginaActual = nuevaPagina;
        }
    }

    // =========================================================
    // GUARDAR Y ACCIONES
    // =========================================================
    async function guardarAsignatura(e) {
        e.preventDefault();

        if (!form.codigo_asignatura.trim()) {
            return Swal.fire('Atención', 'Ingresa el código de la asignatura.', 'warning');
        }
        if (!form.nombres_asignaturas.trim()) {
            return Swal.fire('Atención', 'Ingresa el nombre de la asignatura.', 'warning');
        }
        if (!form.programa_id) {
            return Swal.fire('Atención', 'Selecciona un programa académico.', 'warning');
        }

        guardando = true;

        try {
            const payload = {
                codigo_asignatura: form.codigo_asignatura.trim(),
                nombres_asignaturas: form.nombres_asignaturas.trim(),
                programa_id: Number(form.programa_id),
                creditos: Number(form.creditos),
                estado: Boolean(form.estado)
            };

            const url = idEditando !== null ? `${API_URL}/asignaturas/${idEditando}` : `${API_URL}/asignaturas/`;
            const method = idEditando !== null ? 'PUT' : 'POST';

            const response = await fetch(url, {
                method,
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            const data = await response.json().catch(() => null);

            if (!response.ok) {
                throw new Error(data?.detail || data?.mensaje || `Error del servidor: ${response.status}`);
            }

            await Swal.fire({
                icon: 'success',
                title: idEditando !== null ? 'Asignatura actualizada' : 'Asignatura creada',
                text: data?.mensaje || 'Operación realizada correctamente',
                timer: 1500,
                showConfirmButton: false
            });

            limpiarFormulario();
            await cargarDatos();

        } catch (err) {
            console.error('Error guardando asignatura:', err);
            Swal.fire('Error', err.message || 'No se pudo guardar la asignatura', 'error');
        } finally {
            guardando = false;
        }
    }

    function prepararEdicion(asignatura) {
        idEditando = asignatura.asignatura_id;
        form = {
            codigo_asignatura: asignatura.codigo_asignatura || '',
            nombres_asignaturas: asignatura.nombres_asignaturas || '',
            programa_id: asignatura.programa_id ? String(asignatura.programa_id) : '',
            creditos: asignatura.creditos || 3,
            estado: asignatura.estado ?? true
        };

        window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    async function eliminarAsignatura(id) {
        const confirmacion = await Swal.fire({
            title: '¿Eliminar asignatura?',
            text: 'Esta acción no se puede deshacer.',
            icon: 'warning',
            showCancelButton: true,
            confirmButtonColor: '#d33',
            confirmButtonText: 'Sí, eliminar',
            cancelButtonText: 'Cancelar'
        });

        if (!confirmacion.isConfirmed) return;

        try {
            const response = await fetch(`${API_URL}/asignaturas/${id}`, { method: 'DELETE' });
            const data = await response.json().catch(() => null);

            if (!response.ok) {
                throw new Error(data?.detail || data?.mensaje || `Error al eliminar: ${response.status}`);
            }

            asignaturas = asignaturas.filter(a => a.asignatura_id !== id);

            await Swal.fire({
                icon: 'success',
                title: 'Eliminada',
                text: data?.mensaje || 'Asignatura eliminada correctamente',
                timer: 1500,
                showConfirmButton: false
            });

        } catch (err) {
            Swal.fire('Error', err.message, 'error');
        }
    }

    function limpiarFormulario() {
        idEditando = null;
        form = {
            codigo_asignatura: '',
            nombres_asignaturas: '',
            programa_id: '',
            creditos: 3,
            estado: true
        };
    }

    function limpiarFiltros() {
        busqueda = '';
        filtroFacultad = 'Todas';
        filtroPrograma = 'Todos';
        paginaActual = 1;
    }

    function cambiarFacultad() {
        paginaActual = 1;
        if (
            filtroPrograma !== 'Todos' &&
            !programasFiltrados.some(p => p.nombre_programa === filtroPrograma)
        ) {
            filtroPrograma = 'Todos';
        }
    }
</script>

<svelte:head>
    <title>Asignaturas | Acta Docente</title>
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
                    📚
                </div>
                <div>
                    <h2 class="mb-0 fw-bold">Estructura Académica</h2>
                    <small class="text-muted">Administración de asignaturas y programas académicos</small>
                </div>
            </div>
        </div>
    </div>

    <!-- ERROR -->
    {#if error}
        <div class="alert alert-danger shadow-sm">
            ⚠️ <strong>Error de conexión:</strong> {error}
        </div>
    {/if}

    <div class="row g-4">
        <!-- FORMULARIO -->
        <div class="col-lg-4">
            <div class="card border-0 shadow-sm">
                <div class="card-header text-white d-flex justify-content-between align-items-center" style="background-color: #0f3460;">
                    <h5 class="mb-0 fw-bold fs-6">
                        {idEditando !== null ? 'Editar Asignatura' : 'Nueva Asignatura'}
                    </h5>
                    {#if idEditando !== null}
                        <span class="badge bg-warning text-dark">Modo Edición</span>
                    {/if}
                </div>

                <div class="card-body">
                    <form onsubmit={guardarAsignatura}>
                        <div class="mb-3">
                            <label for="codigo" class="form-label fw-semibold text-secondary">Código</label>
                            <input
                                id="codigo"
                                type="text"
                                class="form-control"
                                placeholder="Ej: MAT101"
                                maxlength="20"
                                bind:value={form.codigo_asignatura}
                                required
                            />
                        </div>

                        <div class="mb-3">
                            <label for="nombre" class="form-label fw-semibold text-secondary">Nombre</label>
                            <input
                                id="nombre"
                                type="text"
                                class="form-control"
                                placeholder="Ej: Cálculo I"
                                maxlength="100"
                                bind:value={form.nombres_asignaturas}
                                required
                            />
                        </div>

                        <div class="mb-3">
                            <label for="programa" class="form-label fw-semibold text-secondary">Programa</label>
                            <select id="programa" class="form-select" bind:value={form.programa_id} required>
                                <option value="">Seleccionar...</option>
                                {#each programas as programa}
                                    <option value={String(programa.programa_id)}>{programa.nombre_programa}</option>
                                {/each}
                            </select>
                        </div>

                        <div class="mb-3">
                            <label for="creditos" class="form-label fw-semibold text-secondary">Créditos</label>
                            <input
                                id="creditos"
                                type="number"
                                min="1"
                                class="form-control"
                                bind:value={form.creditos}
                                required
                            />
                        </div>

                        <div class="mb-3">
                            <label for="estado" class="form-label fw-semibold text-secondary">Estado</label>
                            <select id="estado" class="form-select" bind:value={form.estado}>
                                <option value={true}>Activa</option>
                                <option value={false}>Inactiva</option>
                            </select>
                        </div>

                        <div class="d-flex gap-2">
                            <button
                                type="submit"
                                class="btn text-white flex-grow-1 fw-semibold"
                                style="background-color: #0f3460;"
                                disabled={guardando}
                            >
                                {#if guardando}
                                    <span class="spinner-border spinner-border-sm me-2"></span> Guardando...
                                {:else}
                                    {idEditando !== null ? 'Actualizar' : 'Crear'}
                                {/if}
                            </button>

                            <button
                                type="button"
                                class="btn btn-outline-secondary"
                                onclick={limpiarFormulario}
                                disabled={guardando}
                            >
                                Cancelar
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </div>

        <!-- TABLA CON FILTROS Y PAGINACIÓN -->
        <div class="col-lg-8">
            <div class="card border-0 shadow-sm">
                <div class="card-body">
                    <div class="d-flex justify-content-between align-items-center mb-3">
                        <div>
                            <h4 class="mb-0 fw-bold">Asignaturas Registradas</h4>
                            <small class="text-muted">Listado completo de materias</small>
                        </div>
                        <span class="badge text-white fs-6 px-3 py-2" style="background-color: #0f3460;">
                            {asignaturasFiltradas.length} resultados
                        </span>
                    </div>

                    <!-- FILTROS Y CANTIDAD POR PÁGINA -->
                    <div class="row g-2 mb-2">
                        <div class="col-md-5">
                            <input
                                type="text"
                                class="form-control"
                                placeholder="🔍 Buscar por nombre o código..."
                                bind:value={busqueda}
                                oninput={() => paginaActual = 1}
                            />
                        </div>

                        <div class="col-md-3">
                            <select
                                class="form-select"
                                bind:value={filtroFacultad}
                                onchange={cambiarFacultad}
                            >
                                <option value="Todas">Todas las facultades</option>
                                {#each facultades as facultad}
                                    <option value={facultad.nombre_facultad || facultad.nombre}>
                                        {facultad.nombre_facultad || facultad.nombre}
                                    </option>
                                {/each}
                            </select>
                        </div>

                        <div class="col-md-4">
                            <select
                                class="form-select"
                                bind:value={filtroPrograma}
                                onchange={() => paginaActual = 1}
                            >
                                <option value="Todos">Todos los programas</option>
                                {#each programasFiltrados as programa}
                                    <option value={programa.nombre_programa}>{programa.nombre_programa}</option>
                                {/each}
                            </select>
                        </div>
                    </div>

                    <div class="d-flex justify-content-between align-items-center mb-3">
                        <button
                            type="button"
                            class="btn btn-sm btn-link text-decoration-none p-0"
                            style="color: #0f3460;"
                            onclick={limpiarFiltros}
                        >
                            ↻ Limpiar filtros
                        </button>

                        <div class="d-flex align-items-center gap-2">
                            <label for="porPaginaSelect" class="text-muted small fw-semibold">Mostrar:</label>
                            <select id="porPaginaSelect" class="form-select form-select-sm" bind:value={porPagina} onchange={() => paginaActual = 1}>
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
                                    <th>Código</th>
                                    <th>Asignatura</th>
                                    <th>Facultad</th>
                                    <th>Programa</th>
                                    <th class="text-center">Créditos</th>
                                    <th class="text-center">Estado</th>
                                    <th class="text-center">Acciones</th>
                                </tr>
                            </thead>
                            <tbody>
                                {#if cargando}
                                    <tr>
                                        <td colspan="8" class="text-center py-5">
                                            <div class="spinner-border text-primary" role="status"></div>
                                            <p class="text-muted mt-2 mb-0">Cargando datos desde Render...</p>
                                        </td>
                                    </tr>
                                {:else}
                                    {#each asignaturasPaginadas as a (a.asignatura_id)}
                                        <tr>
                                            <td class="text-muted">#{a.asignatura_id}</td>
                                            <td><span class="badge text-bg-secondary">{a.codigo_asignatura}</span></td>
                                            <td class="fw-semibold">{a.nombres_asignaturas}</td>
                                            <td>{nombreFacultad(a.programa_id)}</td>
                                            <td>{nombrePrograma(a.programa_id)}</td>
                                            <td class="text-center">
                                                <span class="badge text-white" style="background-color: #0f3460;">
                                                    {a.creditos}
                                                </span>
                                            </td>
                                            <td class="text-center">
                                                <span class="badge {a.estado ? 'bg-success-subtle text-success' : 'bg-secondary-subtle text-secondary'}">
                                                    {a.estado ? 'Activa' : 'Inactiva'}
                                                </span>
                                            </td>
                                            <td class="text-center">
                                                <div class="d-flex justify-content-center gap-1">
                                                    <button
                                                        type="button"
                                                        class="btn btn-sm btn-outline-primary fw-semibold px-2"
                                                        title="Editar"
                                                        onclick={() => prepararEdicion(a)}
                                                    >
                                                        Editar
                                                    </button>

                                                    <button
                                                        type="button"
                                                        class="btn btn-sm btn-outline-danger fw-semibold px-2"
                                                        title="Eliminar"
                                                        onclick={() => eliminarAsignatura(a.asignatura_id)}
                                                    >
                                                        Eliminar
                                                    </button>
                                                </div>
                                            </td>
                                        </tr>
                                    {:else}
                                        <tr>
                                            <td colspan="8" class="text-center py-4 text-muted">
                                                No se encontraron asignaturas con los criterios ingresados.
                                            </td>
                                        </tr>
                                    {/each}
                                {/if}
                            </tbody>
                        </table>
                    </div>

                    <!-- PAGINACIÓN -->
                    {#if !cargando && totalPaginas > 1}
                        <div class="d-flex flex-column flex-md-row justify-content-between align-items-center gap-2 mt-3 pt-3 border-top">
                            <span class="text-muted small">
                                Página <strong>{paginaActual}</strong> de <strong>{totalPaginas}</strong> (Mostrando {asignaturasPaginadas.length} de {asignaturasFiltradas.length} resultados)
                            </span>
                            <nav>
                                <ul class="pagination pagination-sm mb-0">
                                    <li class="page-item {paginaActual === 1 ? 'disabled' : ''}">
                                        <button class="page-link" onclick={() => cambiarPagina(paginaActual - 1)}>Anterior</button>
                                    </li>

                                    {#each Array(totalPaginas) as _, i}
                                        <li class="page-item {paginaActual === i + 1 ? 'active' : ''}">
                                            <button 
                                                class="page-link" 
                                                onclick={() => cambiarPagina(i + 1)}
                                                style={paginaActual === i + 1 ? 'background-color: #0f3460; border-color: #0f3460;' : ''}
                                            >
                                                {i + 1}
                                            </button>
                                        </li>
                                    {/each}

                                    <li class="page-item {paginaActual === totalPaginas ? 'disabled' : ''}">
                                        <button class="page-link" onclick={() => cambiarPagina(paginaActual + 1)}>Siguiente</button>
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