<script>
    import Footer from '$lib/components/footer.svelte';
    import { goto } from '$app/navigation';

    // Estado del formulario (Svelte 5 Runes)
    let email = $state('');
    let password = $state('');
    let recordar = $state(false);
    let cargando = $state(false);
    let mostrarPassword = $state(false);
    let mensajeError = $state('');
    let mensajeExito = $state('');

    // Credenciales de prueba asociadas a los 3 roles del sistema
    
        const usuariosPrueba = [
            {
                nombre: 'Jhon Jimenez',
                correo: 'docente@acta.edu.co',
                pass: 'docente123',
                rol: 'Docente',
                badge: 'bg-secondary'
            },
            {
                nombre: 'Elian Ramos',
                correo: 'jefe@acta.edu.co',
                pass: 'jefe123',
                rol: 'Jefe de Departamento',
                badge: 'bg-primary'
            },
            {
                nombre: 'Nicolas Zabala',
                correo: 'direccion@acta.edu.co',
                pass: 'direccion123',
                rol: 'Dirección Académica',
                badge: 'bg-dark'
            }
        ];

    const modulosPorRol = {
        'Docente': [
            'Carga Docente',
            'Horarios Grupos',
            'Evaluación Docente'
        ],

        'Jefe de Departamento': [
            'Carga Docente',
            'Estructura Académica',
            'Horarios Grupos',
            'Evaluación Docente'
        ],

        'Dirección Académica': [
            'Usuario',
            'Rol',
            'Carga Docente',
            'Estructura Académica',
            'Horarios Grupos',
            'Evaluación Docente',
            'Analítica'
        ]
    };

    // Función auxiliar para auto-completar desde los botones de acceso rápido
    function cargarCredencial(u) {
        email = u.correo;
        password = u.pass;
        mensajeError = '';
        mensajeExito = '';
    }

    function manejarLogin(e) {
        e.preventDefault();
        mensajeError = '';
        mensajeExito = '';

        if (!email.trim() || !password.trim()) {
            mensajeError = 'Por favor ingresa tu correo y contraseña.';
            return;
        }

        cargando = true;

        setTimeout(() => {
            cargando = false;
            
            // Buscar usuario coincidente en la lista de credenciales
            const usuarioEncontrado = usuariosPrueba.find(
                (u) => u.correo.toLowerCase() === email.trim().toLowerCase() && u.pass === password
            );

            if (usuarioEncontrado) {
                mensajeExito = `¡Bienvenido! Sesión iniciada como ${usuarioEncontrado.rol}. Redirigiendo...`;
                
                // Guardar rol o sesión simulada si es necesario (localStorage/sessionStorage)
                if (typeof window !== 'undefined') {
                    sessionStorage.setItem(
                        'usuario_rol',
                        usuarioEncontrado.rol
                    );

                    sessionStorage.setItem(
                        'usuario_correo',
                        usuarioEncontrado.correo
                    );

                    sessionStorage.setItem(
                        'usuario_nombre',
                        usuarioEncontrado.nombre
                    );

                    sessionStorage.setItem(
                        'usuario_modulos',
                        JSON.stringify(
                            modulosPorRol[usuarioEncontrado.rol]
                        )
                    );
                                    }

                setTimeout(() => {
                    goto('/inicio');
                }, 1000);
            } else {
                mensajeError = 'Credenciales incorrectas. Selecciona una cuenta de prueba o verifica tus datos.';
            }
        }, 800);
    }
</script>

<svelte:head>
    <title>Iniciar Sesión | Acta Docente</title>
</svelte:head>

<div class="min-vh-100 d-flex flex-column bg-light">
    <div class="row g-0 flex-grow-1">
        
        <!-- PANEL LATERAL IZQUIERDO (Branding & Info) -->
        <div
            class="col-lg-6 d-none d-lg-flex flex-column justify-content-between p-5 text-white"
            style="background-color: #0f3460;"
        >
            <div class="d-flex align-items-center gap-3">
                <div
                    class="bg-white rounded-3 d-flex align-items-center justify-content-center shadow-sm"
                    style="width: 48px; height: 48px;"
                >
                    <i class="bi bi-mortarboard-fill fs-4" style="color: #0f3460;"></i>
                </div>
                <span class="fs-4 fw-bold tracking-wide">Acta Docente</span>
            </div>

            <!-- Mensaje central -->
            <div class="my-auto py-5 pe-lg-5">
                <h1 class="display-5 fw-bold mb-3 lh-sm">Plataforma de Gestión Académica</h1>
                <p class="lead text-white-50">
                    Administra asignaturas, programas, grupos y horarios docentes en un solo lugar de manera eficiente y centralizada.
                </p>
            </div>

            
        </div>

        <!-- PANEL DERECHO (Formulario de Login) -->
        <div class="col-lg-6 d-flex align-items-center justify-content-center p-4 p-sm-5 bg-white">
            <div class="w-100" style="max-width: 440px;">
                
                <!-- Header Móvil -->
                <div class="d-lg-none text-center mb-4">
                    <div
                        class="rounded-circle d-inline-flex align-items-center justify-content-center text-white fs-3 mb-2 shadow-sm"
                        style="width: 60px; height: 60px; background-color: #0f3460;"
                    >
                        <i class="bi bi-mortarboard-fill"></i>
                    </div>
                    <h3 class="fw-bold">Acta Docente</h3>
                </div>

                <!-- Título -->
                <div class="mb-4">
                    <h3 class="fw-bold mb-1 text-dark">Iniciar Sesión</h3>
                    <p class="text-muted">Ingresa tus credenciales para acceder al sistema</p>
                </div>

                <!-- BOTONES DE ACCESO RÁPIDO PARA PRUEBAS -->
                <div class="p-3 bg-light rounded-3 border mb-4">
                    <div class="d-flex align-items-center justify-content-between mb-2">
                        <small class="fw-bold text-secondary">Cuentas de Prueba :</small>
                    </div>
                    <div class="d-flex flex-wrap gap-2">
                        {#each usuariosPrueba as u}
                            <button
                                type="button"
                                class="btn btn-sm btn-outline-secondary bg-white text-dark d-flex align-items-center gap-1 shadow-sm fs-7"
                                onclick={() => cargarCredencial(u)}
                            >
                                <span class="badge {u.badge} rounded-pill">{u.rol}</span>
                            </button>
                        {/each}
                    </div>
                </div>

                <!-- Alerta de Error -->
                {#if mensajeError}
                    <div class="alert alert-danger alert-dismissible fade show small d-flex align-items-center gap-2" role="alert">
                        <i class="bi bi-exclamation-triangle-fill flex-shrink-0"></i>
                        <div>{mensajeError}</div>
                        <button type="button" class="btn-close" onclick={() => (mensajeError = '')} aria-label="Cerrar"></button>
                    </div>
                {/if}

                <!-- Alerta de Éxito -->
                {#if mensajeExito}
                    <div class="alert alert-success alert-dismissible fade show small d-flex align-items-center gap-2" role="alert">
                        <i class="bi bi-check-circle-fill flex-shrink-0"></i>
                        <div>{mensajeExito}</div>
                    </div>
                {/if}

                <!-- Formulario -->
                <form onsubmit={manejarLogin}>
                    <!-- CORREO -->
                    <div class="mb-3">
                        <label for="email" class="form-label fw-semibold text-secondary small">Correo Electrónico</label>
                        <div class="input-group">
                            <span class="input-group-text bg-light border-end-0 text-muted">
                                <i class="bi bi-envelope"></i>
                            </span>
                            <input
                                id="email"
                                type="email"
                                class="form-control border-start-0 ps-0"
                                placeholder="usuario@institucion.edu.co"
                                bind:value={email}
                                autocomplete="email"
                                required
                            />
                        </div>
                    </div>

                    <!-- CONTRASEÑA -->
                    <div class="mb-3">
                        <div class="d-flex justify-content-between align-items-center mb-1">
                            <label for="password" class="form-label fw-semibold text-secondary small mb-0">Contraseña</label>
                            <a href="#olvide" class="small text-decoration-none fw-semibold" style="color: #0f3460;">
                                ¿Olvidaste tu contraseña?
                            </a>
                        </div>
                        <div class="input-group">
                            <span class="input-group-text bg-light border-end-0 text-muted">
                                <i class="bi bi-lock"></i>
                            </span>
                            <input
                                id="password"
                                type={mostrarPassword ? 'text' : 'password'}
                                class="form-control border-start-0 border-end-0 px-0"
                                placeholder="••••••••"
                                bind:value={password}
                                autocomplete="current-password"
                                required
                            />
                            <button
                                type="button"
                                class="btn btn-light border border-start-0 text-muted"
                                onclick={() => (mostrarPassword = !mostrarPassword)}
                                tabindex="-1"
                                title={mostrarPassword ? 'Ocultar contraseña' : 'Mostrar contraseña'}
                            >
                                <i class={mostrarPassword ? 'bi bi-eye-slash' : 'bi bi-eye'}></i>
                            </button>
                        </div>
                    </div>

                    <!-- RECORDAR SESIÓN -->
                    <div class="form-check mb-4">
                        <input
                            id="recordar"
                            type="checkbox"
                            class="form-check-input"
                            bind:checked={recordar}
                        />
                        <label for="recordar" class="form-check-label text-secondary small user-select-none">
                            Recordar mi sesión en este dispositivo
                        </label>
                    </div>

                    <!-- BOTÓN INGRESAR -->
                    <button
                        type="submit"
                        class="btn w-100 text-white py-2.5 fw-semibold shadow-sm"
                        style="background-color: #0f3460;"
                        disabled={cargando}
                    >
                        {#if cargando}
                            <span class="spinner-border spinner-border-sm me-2" role="status" aria-hidden="true"></span>
                            Autenticando...
                        {:else}
                            Ingresar al Sistema
                        {/if}
                    </button>
                </form>

                <!-- Pie de ayuda -->
                <div class="mt-4 text-center">
                    <small class="text-muted">
                        ¿Problemas para acceder?
                        <a href="#soporte" class="text-decoration-none fw-semibold" style="color: #0f3460;">
                            Contacta con Soporte Técnico
                        </a>
                    </small>
                </div>

            </div>
        </div>

    </div>
</div>

<Footer />