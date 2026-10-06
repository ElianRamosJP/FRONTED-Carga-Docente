<script>
    import { onMount } from 'svelte';
    import Swal from 'sweetalert2';

    import Navbar from '../../lib/components/navbar.svelte';
    import Footer from '../../lib/components/footer.svelte';
    import Header from '../../lib/components/header.svelte';

    const BASE_URL = 'https://api-carga-e4od.onrender.com';
    const API_URL = `${BASE_URL}/usuarios/`;

    // Estados reactivos (Svelte 5 Runes)
    let usuarios = $state([]);
    let loading = $state(true);
    let error = $state(null);
    let busqueda = $state('');

    // Paginación (Svelte 5 Runes)
    let paginaActual = $state(1);
    let itemsPorPagina = $state(5);

    // Estado del Modal de Edición
    let modalEdicionAbierto = $state(false);
    let guardandoEdicion = $state(false);
    let usuarioForm = $state({
        usuario_id: null,
        primer_nombre: '',
        segundo_nombre: '',
        primer_apellido: '',
        segundo_apellido: '',
        correo: '',
        codigo_personal: '',
        titulo_academico: '',
        estado: true
    });

    // Cargar datos
    onMount(async () => {
        await cargarUsuarios();
    });

    async function cargarUsuarios() {
        loading = true;
        error = null;
        try {
            const response = await fetch(API_URL);

            if (!response.ok) {
                throw new Error(`Servidor respondió con código ${response.status}`);
            }

            const data = await response.json();
            
            // Ordenar numéricamente por ID por defecto
            usuarios = data.sort((a, b) => a.usuario_id - b.usuario_id);
        } catch (e) {
            error = e.message;
            Swal.fire({
                icon: 'error',
                title: 'Error de conexión',
                text: `No se pudieron cargar los usuarios: ${e.message}`
            });
        } finally {
            loading = false;
        }
    }

    // Filtro reactivo en tiempo real ($derived)
    let usuariosFiltrados = $derived(
        usuarios.filter((u) => {
            const termino = busqueda.toLowerCase().trim();
            const nombreCompleto = `${u.primer_nombre || ''} ${u.segundo_nombre || ''} ${u.primer_apellido || ''} ${u.segundo_apellido || ''}`.toLowerCase();
            const correo = (u.correo || '').toLowerCase();
            const codigo = (u.codigo_personal || '').toLowerCase();

            return (
                nombreCompleto.includes(termino) ||
                correo.includes(termino) ||
                codigo.includes(termino)
            );
        })
    );

    // Lógica de Paginación ($derived)
    let totalPaginas = $derived(Math.ceil(usuariosFiltrados.length / itemsPorPagina) || 1);

    let usuariosPaginados = $derived(
        usuariosFiltrados.slice(
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
    // ACCIÓN: EDITAR USUARIO
    // =========================================================
    function abrirModalEditar(usuario) {
        usuarioForm = {
            usuario_id: usuario.usuario_id,
            primer_nombre: usuario.primer_nombre || '',
            segundo_nombre: usuario.segundo_nombre || '',
            primer_apellido: usuario.primer_apellido || '',
            segundo_apellido: usuario.segundo_apellido || '',
            correo: usuario.correo || '',
            codigo_personal: usuario.codigo_personal || '',
            titulo_academico: usuario.titulo_academico || '',
            estado: usuario.estado !== undefined ? Boolean(usuario.estado) : true
        };
        modalEdicionAbierto = true;
    }

    function cerrarModalEditar() {
        modalEdicionAbierto = false;
        guardandoEdicion = false;
    }

    async function guardarCambiosUsuario() {
        if (!usuarioForm.primer_nombre.trim() || !usuarioForm.primer_apellido.trim()) {
            return Swal.fire('Campo requerido', 'El primer nombre y el primer apellido son obligatorios.', 'warning');
        }

        guardandoEdicion = true;

        try {
            const response = await fetch(`${API_URL}${usuarioForm.usuario_id}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(usuarioForm)
            });

            const resData = await response.json().catch(() => ({}));

            if (!response.ok) {
                throw new Error(resData?.detail || resData?.mensaje || `Error HTTP ${response.status}`);
            }

            // Actualizar arreglo en memoria
            usuarios = usuarios.map(u => u.usuario_id === usuarioForm.usuario_id ? { ...u, ...usuarioForm } : u);

            cerrarModalEditar();

            Swal.fire({
                icon: 'success',
                title: 'Usuario actualizado',
                text: 'Los datos del usuario fueron modificados correctamente.',
                timer: 1800,
                showConfirmButton: false
            });

        } catch (err) {
            console.error('Error guardando usuario:', err);
            Swal.fire('Error al actualizar', err.message, 'error');
        } finally {
            guardandoEdicion = false;
        }
    }

    // =========================================================
    // ACCIÓN: ELIMINAR USUARIO
    // =========================================================
    function confirmarEliminacion(id, nombre) {
        Swal.fire({
            title: `¿Eliminar a ${nombre}?`,
            text: 'Esta acción eliminará el registro o deshabilitará su acceso al sistema.',
            icon: 'warning',
            showCancelButton: true,
            confirmButtonColor: '#d33',
            cancelButtonColor: '#6c757d',
            confirmButtonText: 'Sí, eliminar',
            cancelButtonText: 'Cancelar'
        }).then(async (result) => {
            if (result.isConfirmed) {
                try {
                    const res = await fetch(`${API_URL}${id}`, {
                        method: 'DELETE'
                    });

                    if (res.ok) {
                        usuarios = usuarios.filter((u) => u.usuario_id !== id);

                        Swal.fire({
                            title: 'Eliminado',
                            text: 'El usuario ha sido eliminado correctamente.',
                            icon: 'success',
                            timer: 1800,
                            showConfirmButton: false
                        });
                    } else {
                        // Intento de desactivación lógica si falla la eliminación directa por FK
                        const deshabilitarRes = await fetch(`${API_URL}${id}`, {
                            method: 'PUT',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify({ estado: false })
                        });

                        if (deshabilitarRes.ok) {
                            usuarios = usuarios.map(u => u.usuario_id === id ? { ...u, estado: false } : u);
                            Swal.fire('Usuario Desactivado', 'Debido a registros asociados, el usuario fue marcado como Inactivo.', 'info');
                        } else {
                            throw new Error('No se pudo eliminar ni deshabilitar el registro en el servidor.');
                        }
                    }
                } catch (err) {
                    Swal.fire('Error', err.message, 'error');
                }
            }
        });
    }
</script>

<svelte:head>
    <title>Usuarios | Acta Docente</title>
</svelte:head>

<Header />
<Navbar />

<main class="bg-light min-vh-100 p-4">
    <div class="container-fluid px-2 px-md-4">

        <!-- ENCABEZADO -->
        <div class="card border-0 shadow-sm mb-4">
            <div class="card-body">
                <div class="d-flex flex-column flex-md-row justify-content-between align-items-md-center gap-3">
                    <div>
                        <h2 class="fw-bold mb-1">Gestión de Usuarios</h2>
                        <p class="text-muted mb-0">
                            Administración de los usuarios registrados en el sistema
                        </p>
                    </div>

                    <button
                        type="button"
                        class="btn text-white px-4 py-2 shadow-sm fw-semibold"
                        style="background-color: #0f3460;"
                        onclick={cargarUsuarios}
                    >
                        Actualizar
                    </button>
                </div>
            </div>
        </div>

        <!-- CONTENIDO -->
        <div class="card shadow-sm border-0">
            <div class="card-body p-4">

                <!-- BUSCADOR Y SELECTOR DE ITEMS -->
                <div class="row g-2 mb-4 align-items-center justify-content-between">
                    <div class="col-12 col-md-6 col-lg-5">
                        <input
                            type="text"
                            class="form-control"
                            placeholder="Buscar por nombre, correo o código..."
                            bind:value={busqueda}
                            oninput={() => paginaActual = 1}
                        />
                    </div>

                    <div class="col-12 col-md-auto d-flex align-items-center justify-content-md-end gap-3">
                        <div class="d-flex align-items-center gap-2">
                            <label for="selectItems" class="text-muted small fw-semibold text-nowrap">Mostrar:</label>
                            <select
                                id="selectItems"
                                class="form-select form-select-sm"
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

                        <span
                            class="badge text-white px-3 py-2 fs-6"
                            style="background-color: #0f3460;"
                        >
                            {usuariosFiltrados.length} usuarios encontrados
                        </span>
                    </div>
                </div>

                <!-- ESTADO DE CARGA / ERROR -->
                {#if loading}
                    <div class="text-center py-5">
                        <div class="spinner-border text-primary" role="status">
                            <span class="visually-hidden">Cargando...</span>
                        </div>
                        <p class="mt-2 text-muted">Cargando datos desde el servidor...</p>
                    </div>
                {:else if error}
                    <div class="alert alert-danger text-center" role="alert">
                        Ocurrió un error al cargar los datos: <strong>{error}</strong>
                    </div>
                {:else}
                    <!-- TABLA -->
                    <div class="table-responsive">
                        <table class="table table-hover align-middle mb-0 w-100">
                            <thead class="table-light text-secondary">
                                <tr>
                                    <th class="ps-3">ID</th>
                                    <th>USUARIO</th>
                                    <th>CORREO</th>
                                    <th>CÓDIGO / TÍTULO</th>
                                    <th class="text-center">ESTADO</th>
                                    <th class="text-center pe-3">ACCIONES</th>
                                </tr>
                            </thead>

                            <tbody>
                                {#each usuariosPaginados as usuario (usuario.usuario_id)}
                                    {@const nombreCompleto = `${usuario.primer_nombre || ''} ${usuario.primer_apellido || ''}`.trim()}
                                    <tr>
                                        <td class="ps-3 fw-bold text-secondary">
                                            #{usuario.usuario_id}
                                        </td>
                                        <td class="fw-bold text-dark">
                                            {usuario.primer_nombre} {usuario.segundo_nombre || ''} {usuario.primer_apellido} {usuario.segundo_apellido || ''}
                                        </td>
                                        <td class="text-secondary">
                                            {usuario.correo}
                                        </td>
                                        <td>
                                            <span class="badge bg-secondary-subtle text-secondary border border-secondary-subtle">
                                                {usuario.codigo_personal || 'N/A'}
                                            </span>
                                            <small class="d-block text-muted">{usuario.titulo_academico || ''}</small>
                                        </td>
                                        <td class="text-center">
                                            {#if usuario.estado}
                                                <span class="badge bg-success-subtle text-success border border-success-subtle px-3 py-1">
                                                    Activo
                                                </span>
                                            {:else}
                                                <span class="badge bg-danger-subtle text-danger border border-danger-subtle px-3 py-1">
                                                    Inactivo
                                                </span>
                                            {/if}
                                        </td>
                                        <td class="text-center pe-3">
                                            <div class="d-flex justify-content-center gap-1">
                                                <button
                                                    type="button"
                                                    class="btn btn-sm btn-outline-primary fw-semibold px-2"
                                                    title="Editar Usuario"
                                                    onclick={() => abrirModalEditar(usuario)}
                                                >
                                                    Editar
                                                </button>

                                                <button
                                                    type="button"
                                                    class="btn btn-sm btn-outline-danger fw-semibold px-2"
                                                    title="Eliminar Usuario"
                                                    onclick={() => confirmarEliminacion(usuario.usuario_id, nombreCompleto)}
                                                >
                                                    Eliminar
                                                </button>
                                            </div>
                                        </td>
                                    </tr>
                                {:else}
                                    <tr>
                                        <td colspan="6" class="text-center py-5 text-muted">
                                            <h5 class="fw-bold">No se encontraron usuarios</h5>
                                            <p class="mb-0">Prueba cambiando los criterios de búsqueda.</p>
                                        </td>
                                    </tr>
                                {/each}
                            </tbody>
                        </table>
                    </div>

                    <!-- PAGINACIÓN -->
                    {#if totalPaginas > 1}
                        <div class="d-flex flex-column flex-md-row justify-content-between align-items-center mt-4 pt-3 border-top gap-2">
                            <small class="text-muted">
                                Mostrando <strong>{usuariosPaginados.length}</strong> de <strong>{usuariosFiltrados.length}</strong> resultados — Página <strong>{paginaActual}</strong> de <strong>{totalPaginas}</strong>
                            </small>

                            <nav>
                                <ul class="pagination pagination-sm mb-0">
                                    <li class="page-item {paginaActual === 1 ? 'disabled' : ''}">
                                        <button class="page-link" onclick={() => cambiarPagina(paginaActual - 1)}>
                                            Anterior
                                        </button>
                                    </li>

                                    {#each Array.from({ length: totalPaginas }, (_, i) => i + 1) as page}
                                        <li class="page-item {paginaActual === page ? 'active' : ''}">
                                            <button
                                                class="page-link"
                                                style={paginaActual === page ? 'background-color: #0f3460; border-color: #0f3460;' : ''}
                                                onclick={() => cambiarPagina(page)}
                                            >
                                                {page}
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
                {/if}

            </div>
        </div>

    </div>
</main>

<!-- =========================================================
     MODAL REACTIVO DE EDICIÓN
========================================================= -->
{#if modalEdicionAbierto}
    <div class="modal fade show d-block" style="background-color: rgba(0, 0, 0, 0.5);">
        <div class="modal-dialog modal-dialog-centered modal-lg">
            <div class="modal-content border-0 shadow">
                <div class="modal-header text-white" style="background-color: #0f3460;">
                    <h5 class="modal-title fw-bold">Editar Usuario #{usuarioForm.usuario_id}</h5>
                    <button 
                        type="button" 
                        class="btn-close btn-close-white" 
                        aria-label="Cerrar modal"
                        onclick={cerrarModalEditar}
                    ></button>
                </div>

                <form onsubmit={(e) => { e.preventDefault(); guardarCambiosUsuario(); }}>
                    <div class="modal-body p-4">
                        <div class="row g-3">
                            <div class="col-md-6">
                                <label for="uPrimerNombre" class="form-label fw-semibold text-secondary">Primer Nombre</label>
                                <input id="uPrimerNombre" type="text" class="form-control" bind:value={usuarioForm.primer_nombre} required />
                            </div>

                            <div class="col-md-6">
                                <label for="uSegundoNombre" class="form-label fw-semibold text-secondary">Segundo Nombre</label>
                                <input id="uSegundoNombre" type="text" class="form-control" bind:value={usuarioForm.segundo_nombre} />
                            </div>

                            <div class="col-md-6">
                                <label for="uPrimerApellido" class="form-label fw-semibold text-secondary">Primer Apellido</label>
                                <input id="uPrimerApellido" type="text" class="form-control" bind:value={usuarioForm.primer_apellido} required />
                            </div>

                            <div class="col-md-6">
                                <label for="uSegundoApellido" class="form-label fw-semibold text-secondary">Segundo Apellido</label>
                                <input id="uSegundoApellido" type="text" class="form-control" bind:value={usuarioForm.segundo_apellido} />
                            </div>

                            <div class="col-md-6">
                                <label for="uCorreo" class="form-label fw-semibold text-secondary">Correo Electrónico</label>
                                <input id="uCorreo" type="email" class="form-control" bind:value={usuarioForm.correo} required />
                            </div>

                            <div class="col-md-6">
                                <label for="uCodigo" class="form-label fw-semibold text-secondary">Código Personal</label>
                                <input id="uCodigo" type="text" class="form-control" bind:value={usuarioForm.codigo_personal} />
                            </div>

                            <div class="col-md-8">
                                <label for="uTitulo" class="form-label fw-semibold text-secondary">Título Académico</label>
                                <input id="uTitulo" type="text" class="form-control" bind:value={usuarioForm.titulo_academico} />
                            </div>

                            <div class="col-md-4">
                                <label for="uEstado" class="form-label fw-semibold text-secondary">Estado</label>
                                <select
                                    id="uEstado"
                                    class="form-select"
                                    value={usuarioForm.estado ? 'true' : 'false'}
                                    onchange={(e) => usuarioForm.estado = e.currentTarget.value === 'true'}
                                >
                                    <option value="true">Activo</option>
                                    <option value="false">Inactivo</option>
                                </select>
                            </div>
                        </div>
                    </div>

                    <div class="modal-footer bg-light">
                        <button type="button" class="btn btn-outline-secondary" onclick={cerrarModalEditar} disabled={guardandoEdicion}>
                            Cancelar
                        </button>
                        <button type="submit" class="btn text-white fw-semibold px-4" style="background-color: #0f3460;" disabled={guardandoEdicion}>
                            {#if guardandoEdicion}
                                <span class="spinner-border spinner-border-sm me-2"></span> Guardando...
                            {:else}
                                Guardar Cambios
                            {/if}
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>
{/if}

<Footer />