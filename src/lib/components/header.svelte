
<script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    // Información del usuario actualmente conectado
    let usuario = $state({
        nombre: 'Usuario',
        correo: '',
        rol: 'Invitado'
    });

    // Estados de los menús
    let menuAbierto = $state(false);
    let notificacionesAbiertas = $state(false);

    // Lista de notificaciones
    // Por ahora está vacía para mostrar "No tienes notificaciones"
    let notificaciones = $state([]);

    // Cargar información de la sesión
    onMount(() => {
        const rolGuardado = sessionStorage.getItem('usuario_rol');
        const correoGuardado = sessionStorage.getItem('usuario_correo');
        const nombreGuardado = sessionStorage.getItem('usuario_nombre');

        const datosGuardados = sessionStorage.getItem('usuario');

        let datosUsuario = {};

        if (datosGuardados) {
            try {
                datosUsuario = JSON.parse(datosGuardados);
            } catch (e) {
                console.error('Error al leer los datos del usuario:', e);
            }
        }

        usuario = {
            nombre: nombreGuardado || 'Usuario',
            correo: correoGuardado || '',
            rol: rolGuardado || 'Invitado'
        };
    });


    // Obtener un nombre a partir del correo
    function obtenerNombre(correo) {
        if (!correo) {
            return 'Usuario';
        }

        const parteNombre = correo.split('@')[0];

        return parteNombre
            .replace(/[._-]/g, ' ')
            .replace(/\b\w/g, letra => letra.toUpperCase());
    }


    // Abrir / cerrar menú del perfil
    function toggleMenu() {
        menuAbierto = !menuAbierto;

        // Cerrar notificaciones
        notificacionesAbiertas = false;
    }


    // Abrir / cerrar notificaciones
    function toggleNotificaciones() {
        notificacionesAbiertas = !notificacionesAbiertas;

        // Cerrar perfil
        menuAbierto = false;
    }


    // Cerrar menús
    function cerrarMenu() {
        menuAbierto = false;
        notificacionesAbiertas = false;
    }


    // Cerrar sesión
    function cerrarSesion() {
        if (typeof window !== 'undefined') {
            sessionStorage.removeItem('usuario');
            sessionStorage.removeItem('usuario_rol');
            sessionStorage.removeItem('usuario_correo');
            sessionStorage.removeItem('usuario_modulos');
        }

        goto('/');
    }
</script>


<!-- Cerrar menús al hacer clic fuera -->
<svelte:window onclick={cerrarMenu} />


<!-- COMPONENTE DE ENCABEZADO -->
<header class="p-3">

    <nav
        class="navbar navbar-expand navbar-dark rounded-4 px-4 py-3 shadow-sm"
        style="background-color: #0f3460;"
    >

        <div class="container-fluid position-relative">


            <!-- LOGO Y TÍTULO -->
            <div class="d-flex align-items-center gap-3">

                <div
                    class="bg-white rounded-3 d-flex align-items-center justify-content-center shadow-sm"
                    style="width: 46px; height: 46px;"
                >

                    <i
                        class="bi bi-mortarboard-fill fs-4"
                        style="color: #0f3460;"
                    ></i>

                </div>


                <div class="d-none d-sm-block">

                    <h1 class="h5 text-white fw-semibold mb-0">
                        Acta Docente
                    </h1>

                    <small class="text-white-50">
                        Sistema de carga y evaluación docente
                    </small>

                </div>

            </div>


            <!-- BUSCADOR CENTRADO -->
            <div
                class="position-absolute top-50 start-50 translate-middle d-none d-md-block"
                style="width: 40%; max-width: 450px;"
            >

                <div class="position-relative">

                    <i
                        class="bi bi-search position-absolute top-50 start-0 translate-middle-y ms-3 text-white-50"
                    ></i>

                    <input
                        type="search"
                        class="form-control border-0 text-white ps-5 py-2 rounded-pill shadow-none"
                        style="background-color: rgba(255, 255, 255, 0.15);"
                        placeholder="Buscar docente, materia, grupo..."
                    />

                </div>

            </div>


            <!-- NOTIFICACIONES Y PERFIL -->
            <div class="d-flex align-items-center gap-3 ms-auto">


                <!-- ================================= -->
                <!-- NOTIFICACIONES -->
                <!-- ================================= -->

                <div class="position-relative">

                    <button
                        type="button"
                        class="btn text-white position-relative rounded-circle d-flex align-items-center justify-content-center"
                        style="width: 42px; height: 42px; background-color: rgba(255,255,255,0.1);"
                        title="Notificaciones"
                        onclick={(e) => {
                            e.stopPropagation();
                            toggleNotificaciones();
                        }}
                        aria-expanded={notificacionesAbiertas}
                    >

                        <i class="bi bi-bell fs-5"></i>


                        <!-- INDICADOR DE NOTIFICACIONES -->
                        {#if notificaciones.length > 0}

                            <span
                                class="position-absolute top-0 start-100 translate-middle p-1 bg-danger border border-light rounded-circle"
                            >

                                <span class="visually-hidden">
                                    Tienes notificaciones
                                </span>

                            </span>

                        {/if}

                    </button>


                    <!-- PANEL DE NOTIFICACIONES -->
                    {#if notificacionesAbiertas}

                        <!-- svelte-ignore a11y_no_noninteractive_element_interactions -->
                        <div
                            class="position-absolute end-0 mt-2 bg-white rounded-4 shadow-lg border"
                            style="width: 340px; z-index: 1050;"
                            role="dialog"
                            aria-label="Panel de notificaciones"
                            tabindex="-1"
                            onclick={(e) => e.stopPropagation()}
                            onkeydown={(e) => e.stopPropagation()}
                        >

                            <!-- CABECERA --> 
                            <div 
                                class="d-flex justify-content-between align-items-center px-3 py-3 border-bottom"
                            > 

                                <div> 

                                    <h6 class="mb-0 fw-bold text-dark"> 
                                        Notificaciones 
                                    </h6> 

                                    <small class="text-muted"> 
                                        Avisos del sistema 
                                    </small> 

                                </div> 

                                <i 
                                    class="bi bi-bell text-secondary fs-5"
                                    aria-hidden="true"
                                ></i> 

                            </div> 


                            <!-- CONTENIDO --> 
                            {#if notificaciones.length === 0} 

                                <div class="text-center px-4 py-5"> 

                                    <div 
                                        class="bg-light rounded-circle d-flex align-items-center justify-content-center mx-auto mb-3" 
                                        style="width: 64px; height: 64px;"
                                    > 

                                        <i 
                                            class="bi bi-bell-slash text-secondary fs-3"
                                            aria-hidden="true"
                                        ></i> 

                                    </div> 


                                    <h6 class="fw-semibold text-dark mb-1"> 
                                        No tienes notificaciones 
                                    </h6> 

                                    <p class="text-muted small mb-0"> 
                                        Aquí aparecerán los avisos importantes 
                                        del sistema. 
                                    </p> 

                                </div> 

                            {:else} 

                                <!-- LISTA DE NOTIFICACIONES --> 
                                <div class="list-group list-group-flush"> 

                                    {#each notificaciones as notificacion} 

                                        <button 
                                            type="button" 
                                            class="list-group-item list-group-item-action px-3 py-3 border-0"
                                        > 

                                            <div class="d-flex gap-3"> 

                                                <div> 
                                                    <i 
                                                        class="bi bi-info-circle text-primary fs-5"
                                                        aria-hidden="true"
                                                    ></i> 
                                                </div> 

                                                <div> 

                                                    <div class="fw-semibold"> 
                                                        {notificacion.titulo} 
                                                    </div> 

                                                    <small class="text-muted"> 
                                                        {notificacion.mensaje} 
                                                    </small> 

                                                </div> 

                                            </div> 

                                        </button> 

                                    {/each} 

                                </div> 

                            {/if} 


                            <!-- PIE --> 
                            <div class="border-top px-3 py-2 text-center"> 

                                <small class="text-muted"> 
                                    Acta Docente 
                                </small> 

                            </div> 

                        </div>

                    {/if}

                </div>


                <!-- SEPARADOR -->
                <div
                    class="vr bg-white opacity-25"
                    style="height: 48px;"
                ></div>


                <!-- ================================= -->
                <!-- PERFIL -->
                <!-- ================================= -->

                <div class="dropdown position-relative">

                    <button
                        class="btn border-0 text-white d-flex align-items-center gap-2 p-1 rounded-3"
                        type="button"
                        onclick={(e) => {
                            e.stopPropagation();
                            toggleMenu();
                        }}
                        aria-expanded={menuAbierto}
                    >

                        <!-- ICONO -->
                        <i class="bi bi-person-circle fs-3"></i>


                        <!-- NOMBRE Y ROL -->
                        <div class="text-start d-none d-lg-block">

                            <div class="fw-semibold small lh-1">
                                {usuario.nombre}
                            </div>

                            <div
                                class="text-white-50 mt-1"
                                style="font-size: 11px;"
                            >
                                {usuario.rol}
                            </div>

                        </div>


                        <i
                            class="bi bi-chevron-down small text-white-50 ms-1 d-none d-lg-inline"
                        ></i>

                    </button>


                    <!-- MENÚ DESPLEGABLE -->
                    {#if menuAbierto}

                        <!-- svelte-ignore a11y_no_noninteractive_element_interactions -->
                        <ul
                            class="dropdown-menu dropdown-menu-end shadow-lg border-0 mt-2 show position-absolute end-0"
                            onclick={(e) => e.stopPropagation()}
                            onkeydown={(e) => e.stopPropagation()}
                        >

                            <!-- INFORMACIÓN DEL USUARIO -->
                            <li>

                                <div class="px-3 py-3 border-bottom">

                                    <div class="d-flex align-items-center gap-2">

                                        <i
                                            class="bi bi-person-circle fs-2 text-secondary"
                                        ></i>

                                        <div>

                                            <div class="fw-bold text-dark">
                                                {usuario.nombre}
                                            </div>

                                            <small class="text-muted">
                                                {usuario.correo}
                                            </small>

                                        </div>

                                    </div>

                                </div>

                            </li>


                            <!-- ROL -->
                            <li>

                                <div class="px-3 py-2">

                                    <small class="text-muted d-block">
                                        Rol actual
                                    </small>

                                    <span class="badge bg-primary mt-1">

                                        <i class="bi bi-shield-check me-1"></i>

                                        {usuario.rol}

                                    </span>

                                </div>

                            </li>


                            <li>
                                <hr class="dropdown-divider" />
                            </li>


                            <!-- CUENTA -->
                            <li>
                                <h6 class="dropdown-header">
                                    Cuenta
                                </h6>
                            </li>


                            <!-- PERFIL -->
                            <li>

                                <a
                                    class="dropdown-item d-flex align-items-center"
                                    href="/perfil"
                                    onclick={cerrarMenu}
                                >

                                    <i class="bi bi-person me-2 fs-6"></i>

                                    Mi Perfil

                                </a>

                            </li>


                            <!-- CONFIGURACIÓN -->
                            <li>

                                <a
                                    class="dropdown-item d-flex align-items-center"
                                    href="/configuracion"
                                    onclick={cerrarMenu}
                                >

                                    <i class="bi bi-gear me-2 fs-6"></i>

                                    Configuración

                                </a>

                            </li>


                            <li>
                                <hr class="dropdown-divider" />
                            </li>


                            <!-- CERRAR SESIÓN -->
                            <li>

                                <button
                                    type="button"
                                    class="dropdown-item text-danger fw-semibold d-flex align-items-center"
                                    onclick={cerrarSesion}
                                >

                                    <i
                                        class="bi bi-box-arrow-right me-2 fs-6"
                                    ></i>

                                    Cerrar sesión

                                </button>

                            </li>

                        </ul>

                    {/if}

                </div>

            </div>

        </div>

    </nav>

</header>

