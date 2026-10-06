<script>
    import { onMount } from 'svelte';
    import Swal from 'sweetalert2';
    import Navbar from '$lib/components/navbar.svelte';
    import Footer from '$lib/components/footer.svelte';
    import Header from '$lib/components/header.svelte';

    const API_URL = 'https://api-carga-e4od.onrender.com';

    // Estados reactivos (Svelte 5 Runes)
    let asignaturas = $state([]), periodos = $state([]), grupos =$state([]);
    let cargando = $state(true), guardando =$state(false);
    let error = $state(''), idEditando = $state(null), periodoSeleccionado =$state('');

    // Estados para Búsqueda y Paginación
    let busqueda = $state('');
    let paginaActual = $state(1);
    let porPagina = $state(5);

    let form = $state({
        asignaturas_id: '',
        nombre_grupos: '',
        capacidad: 30,
        estado: true
    });

    onMount(async () => { await cargarTodo(); });

    async function cargarTodo() {
        cargando = true;
        error = '';
        try {
            const fetchSafe = async (endpoint) => {
                const res = await fetch(`${API_URL}/${endpoint}/`);
                if (!res.ok) throw new Error(`Error en ${endpoint}: ${res.status}`);
                const data = await res.json();
                return Array.isArray(data) ? data : (data.data || []);
            };

            const [asigData, perData, grupData] = await Promise.all([
                fetchSafe('asignaturas'),
                fetchSafe('periodos-academicos'),
                fetchSafe('grupos')
            ]);

            asignaturas = asigData;
            periodos = perData;
            grupos = grupData;

            if (periodos.length > 0 && !periodoSeleccionado) {
                periodoSeleccionado = String(periodos[0].periodo_id);
            }
        } catch (err) {
            console.error('Error cargando datos:', err);
            error = err.message || 'Error de conexión';
        } finally {
            cargando = false;
        }
    }

    // LÓGICA DE FILTRADO Y PAGINACIÓN
    let gruposFiltrados = $derived(
        grupos.filter(g => {
            const txt = busqueda.toLowerCase().trim();
            const coincideNombre = String(g.nombre_grupos || '').toLowerCase().includes(txt);
            const coincideAsig = String(g.nombre_asignatura || g.asignatura || '').toLowerCase().includes(txt);
            const coincidePeriodo = String(g.codigo_periodo || '').toLowerCase().includes(txt);
            return coincideNombre || coincideAsig || coincidePeriodo;
        })
    );

    let totalPaginas = $derived(Math.ceil(gruposFiltrados.length / porPagina) || 1);
    
    let gruposPaginados = $derived(
        gruposFiltrados.slice((paginaActual - 1) * porPagina, paginaActual * porPagina)
    );

    function cambiarPagina(nuevaPagina) {
        if (nuevaPagina >= 1 && nuevaPagina <= totalPaginas) {
            paginaActual = nuevaPagina;
        }
    }

    async function guardarGrupo(e) {
        e.preventDefault();

        if (!form.asignaturas_id) return Swal.fire('Atención', 'Selecciona una asignatura.', 'warning');
        if (!periodoSeleccionado) return Swal.fire('Atención', 'Selecciona un período académico.', 'warning');
        if (!form.nombre_grupos.trim()) return Swal.fire('Atención', 'Ingresa el nombre del grupo.', 'warning');
        if (Number(form.capacidad) <= 0) return Swal.fire('Atención', 'La capacidad debe ser mayor a 0.', 'warning');

        guardando = true;
        try {
            const payload = {
                asignaturas_id: Number(form.asignaturas_id),
                periodos_id: Number(periodoSeleccionado),
                nombre_grupos: form.nombre_grupos.trim(),
                capacidad: Number(form.capacidad),
                estado: Boolean(form.estado)
            };

            const url = idEditando !== null ? `${API_URL}/grupos/${idEditando}` : `${API_URL}/grupos/`;
            const method = idEditando !== null ? 'PUT' : 'POST';

            const res = await fetch(url, {
                method,
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            const respuesta = await res.json().catch(() => null);
            if (!res.ok) throw new Error(respuesta?.detail || respuesta?.mensaje || `Error: ${res.status}`);

            await Swal.fire({
                icon: 'success',
                title: idEditando !== null ? '¡Grupo actualizado!' : '¡Grupo creado!',
                text: respuesta?.mensaje || 'Operación realizada correctamente',
                timer: 1500,
                showConfirmButton: false
            });

            limpiarFormulario();
            await cargarTodo();
        } catch (err) {
            Swal.fire('Error', err.message || 'No se pudo guardar el grupo', 'error');
        } finally {
            guardando = false;
        }
    }

    function prepararEdicion(g) {
        idEditando = g.grupos_id ?? g.id_grupo ?? g.id ?? null;
        form = {
            asignaturas_id: String(g.asignaturas_id ?? ''),
            nombre_grupos: g.nombre_grupos ?? '',
            capacidad: g.capacidad ?? 30,
            estado: g.estado ?? true
        };
        if (g.periodos_id) periodoSeleccionado = String(g.periodos_id);
        window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    async function eliminarGrupo(id) {
        const confirmacion = await Swal.fire({
            title: '¿Eliminar grupo?',
            text: 'Esta acción no se puede deshacer.',
            icon: 'warning',
            showCancelButton: true,
            confirmButtonColor: '#d33',
            confirmButtonText: 'Sí, eliminar',
            cancelButtonText: 'Cancelar'
        });

        if (!confirmacion.isConfirmed) return;

        try {
            const res = await fetch(`${API_URL}/grupos/${id}`, { method: 'DELETE' });
            const respuesta = await res.json().catch(() => null);

            if (!res.ok) throw new Error(respuesta?.detail || respuesta?.mensaje || `Error ${res.status}`);

            grupos = grupos.filter(g => (g.grupos_id ?? g.id_grupo ?? g.id) !== id);
            Swal.fire('Eliminado', respuesta?.mensaje || 'El grupo ha sido eliminado.', 'success');
        } catch (err) {
            Swal.fire('Error', err.message, 'error');
        }
    }

    function limpiarFormulario() {
        idEditando = null;
        form = { asignaturas_id: '', nombre_grupos: '', capacidad: 30, estado: true };
    }
</script>

<svelte:head>
    <title>Grupos y Horarios | Acta Docente</title>
</svelte:head>

<Header />
<Navbar />

<main class="container-fluid px-4 py-4 bg-light min-vh-100">
    <!-- ENCABEZADO -->
    <div class="card border-0 shadow-sm mb-4">
        <div class="card-body">
            <div class="row align-items-center">
                <div class="col-md-8">
                    <div class="d-flex align-items-center gap-3">
                        <div class="rounded p-3 text-white fs-4" style="background-color: #0f3460;">📅</div>
                        <div>
                            <h2 class="fw-bold mb-1">Gestión de Grupos y Horarios</h2>
                            <p class="text-muted mb-0">Programación académica y asignación de franjas horarias</p>
                        </div>
                    </div>
                </div>
                <div class="col-md-4 mt-3 mt-md-0">
                    <label for="periodoSelect" class="form-label fw-semibold">Periodo Académico Activo</label>
                    <select id="periodoSelect" class="form-select" bind:value={periodoSeleccionado}>
                        <option value="">Seleccionar período...</option>
                        {#each periodos as p}
                            <option value={String(p.periodo_id)}>{p.codigo_periodo} — {p.fecha_inicio} a {p.fecha_finalizar}</option>
                        {:else}
                            <option value="">Sin períodos registrados</option>
                        {/each}
                    </select>
                </div>
            </div>
        </div>
    </div>

    {#if error}
        <div class="alert alert-danger shadow-sm mb-4">
            ⚠️ <strong>Error de conexión:</strong> {error}
        </div>
    {/if}

    <!-- FORMULARIO -->
    <div class="card border-0 shadow-sm mb-4">
        <div class="card-header text-white d-flex justify-content-between align-items-center" style="background-color: #0f3460;">
            <h5 class="mb-0 fw-bold">{idEditando !== null ? 'Editar Grupo' : 'Programar Nuevo Grupo'}</h5>
            <span class="badge bg-warning text-dark">{idEditando !== null ? 'Modo Edición' : 'Nuevo Registro'}</span>
        </div>
        <div class="card-body">
            <form onsubmit={guardarGrupo}>
                <div class="row g-3 align-items-end">
                    <div class="col-lg-4 col-md-6">
                        <label for="asignaturaSelect" class="form-label fw-semibold">Asignatura</label>
                        <select id="asignaturaSelect" class="form-select" bind:value={form.asignaturas_id} required>
                            <option value="">Seleccionar asignatura...</option>
                            {#each asignaturas as asig}
                                <option value={String(asig.asignatura_id)}>{asig.codigo_asignatura} — {asig.nombres_asignaturas}</option>
                            {/each}
                        </select>
                    </div>

                    <div class="col-lg-4 col-md-6">
                        <label for="nombreGrupo" class="form-label fw-semibold">Nombre del Grupo</label>
                        <input id="nombreGrupo" type="text" class="form-control" placeholder="Ej: A" maxlength="10" bind:value={form.nombre_grupos} required />
                    </div>

                    <div class="col-lg-2 col-md-6">
                        <label for="capacidadInput" class="form-label fw-semibold">Cupo Máx.</label>
                        <input id="capacidadInput" type="number" min="1" class="form-control" bind:value={form.capacidad} required />
                    </div>

                    <div class="col-lg-2 col-md-6">
                        <label for="estadoGrupo" class="form-label fw-semibold">Estado</label>
                        <select id="estadoGrupo" class="form-select" bind:value={form.estado}>
                            <option value={true}>Abierto</option>
                            <option value={false}>Cerrado</option>
                        </select>
                    </div>
                </div>

                <div class="d-flex justify-content-end gap-2 mt-4">
                    <button type="button" class="btn btn-outline-secondary" onclick={limpiarFormulario} disabled={guardando}>Cancelar / Limpiar</button>
                    <button type="submit" class="btn text-white px-4 fw-semibold" style="background-color: #0f3460;" disabled={guardando}>
                        {#if guardando}
                            <span class="spinner-border spinner-border-sm me-2"></span> Guardando...
                        {:else}
                            {idEditando !== null ? 'Actualizar Grupo' : 'Guardar Grupo'}
                        {/if}
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- TABLA CON BUSCADOR, PAGINACIÓN Y BOTONES CON TEXTO -->
    <div class="card border-0 shadow-sm">
        <div class="card-body">
            <!-- FILTROS Y CONTROLES -->
            <div class="row g-2 mb-4 align-items-center justify-content-between">
                <div class="col-12 col-md-5">
                    <div class="input-group">
                        <span class="input-group-text bg-white text-muted">🔍</span>
                        <input 
                            type="text" 
                            class="form-control border-start-0" 
                            placeholder="Buscar por grupo, asignatura o período..." 
                            bind:value={busqueda}
                            oninput={() => paginaActual = 1} 
                        />
                    </div>
                </div>

                <div class="col-12 col-md-auto d-flex align-items-center gap-3">
                    <div class="d-flex align-items-center gap-2">
                        <label for="selectCant" class="text-muted small fw-semibold">Mostrar:</label>
                        <select id="selectCant" class="form-select form-select-sm" bind:value={porPagina} onchange={() => paginaActual = 1}>
                            <option value={5}>5</option>
                            <option value={10}>10</option>
                            <option value={20}>20</option>
                            <option value={50}>50</option>
                        </select>
                    </div>
                    <span class="badge fs-6 text-white px-3 py-2" style="background-color: #0f3460;">
                        {gruposFiltrados.length} grupos
                    </span>
                </div>
            </div>

            <!-- TABLA -->
            <div class="table-responsive">
                <table class="table table-hover align-middle mb-0">
                    <thead class="table-light text-secondary">
                        <tr>
                            <th>ID</th>
                            <th>Asignatura</th>
                            <th>Grupo</th>
                            <th class="text-center">Período</th>
                            <th class="text-center">Cupo Máx.</th>
                            <th class="text-center">Estado</th>
                            <th class="text-center">Acciones</th>
                        </tr>
                    </thead>
                    <tbody>
                        {#if cargando}
                            <tr>
                                <td colspan="7" class="text-center py-5">
                                    <div class="spinner-border text-primary" role="status"></div>
                                    <p class="text-muted mt-2 mb-0">Cargando datos desde Render...</p>
                                </td>
                            </tr>
                        {:else}
                            {#each gruposPaginados as g (g.grupos_id ?? g.id_grupo ?? g.id)}
                                <tr>
                                    <td><span class="badge text-bg-secondary">ID-{g.grupos_id}</span></td>
                                    <td class="fw-semibold">{g.nombre_asignatura || g.asignatura || `Asignatura #${g.asignaturas_id}`}</td>
                                    <td><span class="fw-bold text-dark">{g.nombre_grupos}</span></td>
                                    <td class="text-center">{g.codigo_periodo || `Período #${g.periodos_id}`}</td>
                                    <td class="text-center">
                                        <span class="badge text-white" style="background-color: #0f3460;">👥 {g.capacidad}</span>
                                    </td>
                                    <td class="text-center">
                                        <span class="badge {g.estado ? 'bg-success-subtle text-success border-success-subtle' : 'bg-secondary-subtle text-secondary border-secondary-subtle'} border">
                                            ● {g.estado ? 'Abierto' : 'Cerrado'}
                                        </span>
                                    </td>
                                    <td class="text-center">
                                        <div class="d-flex justify-content-center gap-1">
                                            <button 
                                                type="button" 
                                                class="btn btn-sm btn-outline-primary fw-semibold px-2" 
                                                onclick={() => prepararEdicion(g)}
                                            >
                                                Editar
                                            </button>
                                            <button 
                                                type="button" 
                                                class="btn btn-sm btn-outline-danger fw-semibold px-2" 
                                                onclick={() => eliminarGrupo(g.grupos_id)}
                                            >
                                                Eliminar
                                            </button>
                                        </div>
                                    </td>
                                </tr>
                            {:else}
                                <tr>
                                    <td colspan="7" class="text-center py-5 text-muted">
                                        <div class="fs-1 mb-2">📅</div>
                                        <h5 class="fw-bold">No se encontraron grupos</h5>
                                        <p class="mb-0">Prueba cambiando los criterios de búsqueda o añade un grupo nuevo.</p>
                                    </td>
                                </tr>
                            {/each}
                        {/if}
                    </tbody>
                </table>
            </div>

            <!-- NAVEGACIÓN DE PAGINACIÓN -->
            {#if !cargando && totalPaginas > 1}
                <div class="d-flex flex-column flex-md-row justify-content-between align-items-center gap-2 mt-4 pt-3 border-top">
                    <span class="text-muted small">
                        Página <strong>{paginaActual}</strong> de <strong>{totalPaginas}</strong> (Mostrando {gruposPaginados.length} de {gruposFiltrados.length} registros)
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
</main>

<Footer />