<script>
    import Swal from 'sweetalert2';
    import Navbar from '../../lib/components/navbar.svelte';
    import Footer from '../../lib/components/footer.svelte';
    import Header from '../../lib/components/header.svelte';

    // Módulos tomados del menú de navegación
    let modulos = $state([
        'Usuarios',
        'Rol',
        'Carga Docente',
        'Estructura Académica',
        'Horarios Grupos',
        'Evaluación Docente',
        'Analítica'
    ]);

    // Lista reactiva de roles ($state en Svelte 5)
    let roles = $state(['Dirección Académica', 'Jefes de Departamento', 'Docente']);

    // Estructura de permisos inicializada para los 7 módulos
    let permisos = $state({
        'Dirección Académica': {
            'Usuarios': { lectura: true, escritura: true },
            'Rol': { lectura: true, escritura: true },
            'Carga Docente': { lectura: true, escritura: true },
            'Estructura Académica': { lectura: true, escritura: true },
            'Horarios Grupos': { lectura: true, escritura: true },
            'Evaluación Docente': { lectura: true, escritura: true },
            'Analítica': { lectura: true, escritura: true }
        },
        'Jefes de Departamento': {
            'Usuarios': { lectura: true, escritura: false },
            'Rol': { lectura: false, escritura: false },
            'Carga Docente': { lectura: true, escritura: true },
            'Estructura Académica': { lectura: true, escritura: true },
            'Horarios Grupos': { lectura: true, escritura: true },
            'Evaluación Docente': { lectura: true, escritura: false },
            'Analítica': { lectura: true, escritura: false }
        },
        'Docente': {
            'Usuarios': { lectura: false, escritura: false },
            'Rol': { lectura: false, escritura: false },
            'Carga Docente': { lectura: true, escritura: false },
            'Estructura Académica': { lectura: true, escritura: false },
            'Horarios Grupos': { lectura: true, escritura: false },
            'Evaluación Docente': { lectura: true, escritura: true },
            'Analítica': { lectura: false, escritura: false }
        }
    });

    // Función para agregar un nuevo Rol usando SweetAlert2
    async function modalNuevoRol() {
        const { value: nombreRol } = await Swal.fire({
            title: 'Crear Nuevo Rol',
            input: 'text',
            inputLabel: 'Nombre del rol',
            inputPlaceholder: 'Ej: Coordinador Académico',
            showCancelButton: true,
            confirmButtonColor: '#0f3460',
            cancelButtonColor: '#6c757d',
            confirmButtonText: 'Crear Rol',
            cancelButtonText: 'Cancelar',
            inputValidator: (value) => {
                if (!value || !value.trim()) {
                    return 'Debes ingresar un nombre para el rol';
                }
                if (roles.includes(value.trim())) {
                    return 'Este rol ya existe en el sistema';
                }
            }
        });

        if (nombreRol) {
            const rolLimpio = nombreRol.trim();
            
            // Inicializar los 7 módulos para el nuevo rol
            let nuevosPermisosRol = {};
            modulos.forEach(mod => {
                nuevosPermisosRol[mod] = { lectura: true, escritura: false };
            });

            roles.push(rolLimpio);
            permisos[rolLimpio] = nuevosPermisosRol;

            Swal.fire({
                icon: 'success',
                title: '¡Rol creado!',
                text: `El rol "${rolLimpio}" se ha añadido a la matriz de permisos.`,
                confirmButtonColor: '#0f3460',
                timer: 2000
            });
        }
    }

    // Función para eliminar un Rol
    async function confirmarEliminarRol(rol) {
        const result = await Swal.fire({
            title: `¿Eliminar rol "${rol}"?`,
            text: 'Esta acción removerá los permisos asignados a este rol.',
            icon: 'warning',
            showCancelButton: true,
            confirmButtonColor: '#dc3545',
            cancelButtonColor: '#6c757d',
            confirmButtonText: 'Sí, eliminar',
            cancelButtonText: 'Cancelar'
        });

        if (result.isConfirmed) {
            const index = roles.indexOf(rol);
            if (index !== -1) roles.splice(index, 1);
            delete permisos[rol];

            Swal.fire({
                icon: 'success',
                title: 'Eliminado',
                text: `El rol "${rol}" ha sido removido.`,
                confirmButtonColor: '#0f3460',
                timer: 1800
            });
        }
    }

    // Guardar Cambios
    async function guardarMatriz() {
        const result = await Swal.fire({
            title: '¿Guardar Cambios?',
            text: 'Se actualizarán las políticas de acceso para todos los roles.',
            icon: 'question',
            showCancelButton: true,
            confirmButtonColor: '#0f3460',
            cancelButtonColor: '#6c757d',
            confirmButtonText: 'Guardar Permisos',
            cancelButtonText: 'Cancelar'
        });

        if (result.isConfirmed) {
            Swal.fire({
                title: 'Guardando...',
                text: 'Actualizando matriz de permisos en el servidor',
                allowOutsideClick: false,
                didOpen: () => Swal.showLoading()
            });

            setTimeout(() => {
                Swal.fire({
                    icon: 'success',
                    title: '¡Matriz Actualizada!',
                    text: 'Los cambios de permisos han sido guardados exitosamente.',
                    confirmButtonColor: '#0f3460',
                    timer: 2000
                });
            }, 1000);
        }
    }
</script>

<svelte:head>
    <title>Roles | Acta Docente</title>
</svelte:head>

<Header />
<Navbar />

<main class="bg-light min-vh-100 p-4">
    <div class="container-fluid px-2 px-md-4">

        <!-- ENCABEZADO -->
        <div class="card border-0 shadow-sm mb-4">
            <div class="card-body">
                <div class="d-flex flex-column flex-md-row justify-content-between align-items-md-center gap-3">
                    <div class="d-flex align-items-center gap-3">
                        <div class="rounded p-3 text-white fs-4" style="background-color: #0f3460;">
                            🔐
                        </div>
                        <div>
                            <h2 class="fw-bold mb-1">Gestión de Roles</h2>
                            <p class="text-muted mb-0">Administración de roles y permisos del sistema</p>
                        </div>
                    </div>

                    <button
                        type="button"
                        class="btn text-white px-4 py-2 shadow-sm fw-semibold"
                        style="background-color: #0f3460;"
                        onclick={modalNuevoRol}
                    >
                        + Nuevo Rol
                    </button>
                </div>
            </div>
        </div>

        <!-- MATRIZ DE PERMISOS -->
        <div class="card shadow-sm border-0">
            <div class="card-body p-4">

                <div class="mb-4">
                    <h5 class="fw-bold mb-1">Matriz de Permisos</h5>
                    <p class="text-muted mb-0">
                        Define qué acciones puede realizar cada rol dentro de los módulos del sistema.
                    </p>
                </div>

                <!-- TABLA REACTIVA -->
                <div class="table-responsive">
                    <table class="table table-bordered table-hover align-middle text-center mb-0">
                        <thead class="table-light">
                            <tr>
                                <th rowspan="2" class="text-start align-middle ps-3 bg-white">
                                    MÓDULO
                                </th>
                                {#each roles as rol}
                                    <th colspan="2" class="text-white" style="background-color: #0f3460;">
                                        <div class="d-flex justify-content-between align-items-center px-2">
                                            <span>{rol}</span>
                                            {#if roles.length > 1}
                                                <button 
                                                    type="button" 
                                                    class="btn btn-sm text-white-50 p-0 border-0 fs-6" 
                                                    title="Eliminar Rol"
                                                    onclick={() => confirmarEliminarRol(rol)}
                                                >
                                                    ❌
                                                </button>
                                            {/if}
                                        </div>
                                    </th>
                                {/each}
                            </tr>
                            <tr class="text-uppercase text-secondary small">
                                {#each roles as _}
                                    <th>Lectura</th>
                                    <th>Escritura</th>
                                {/each}
                            </tr>
                        </thead>

                        <tbody>
                            {#each modulos as modulo}
                                <tr>
                                    <td class="text-start fw-bold ps-3 text-dark bg-white">
                                        {modulo}
                                    </td>

                                    {#each roles as rol}
                                        <!-- LECTURA -->
                                        <td>
                                            <div class="form-check d-flex justify-content-center m-0">
                                                <input
                                                    class="form-check-input fs-5"
                                                    type="checkbox"
                                                    bind:checked={permisos[rol][modulo].lectura}
                                                />
                                            </div>
                                        </td>

                                        <!-- ESCRITURA -->
                                        <td>
                                            <div class="form-check d-flex justify-content-center m-0">
                                                <input
                                                    class="form-check-input fs-5"
                                                    type="checkbox"
                                                    bind:checked={permisos[rol][modulo].escritura}
                                                />
                                            </div>
                                        </td>
                                    {/each}
                                </tr>
                            {/each}
                        </tbody>
                    </table>
                </div>

                <!-- BOTÓN GUARDAR -->
                <div class="d-flex justify-content-end mt-4 pt-3 border-top">
                    <button
                        type="button"
                        class="btn text-white px-4 py-2 fw-semibold shadow-sm"
                        style="background-color: #0f3460;"
                        onclick={guardarMatriz}
                    >
                        Guardar Cambios
                    </button>
                </div>

            </div>
        </div>

    </div>
</main>

<Footer />