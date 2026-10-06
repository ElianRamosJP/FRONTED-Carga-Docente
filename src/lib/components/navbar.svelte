
<script>
    import { onMount } from 'svelte';

    // Módulos disponibles para el usuario que inició sesión
    let modulos = $state([]);

    onMount(() => {
        const modulosGuardados = sessionStorage.getItem('usuario_modulos');

        if (modulosGuardados) {
            try {
                modulos = JSON.parse(modulosGuardados);
            } catch (error) {
                console.error('Error al cargar los módulos:', error);
                modulos = [];
            }
        }
    });

    // Comprobar si el usuario tiene acceso a un módulo
    function tieneModulo(nombreModulo) {
        return modulos.includes(nombreModulo);
    }
</script>

<nav class="navbar navbar-expand-lg bg-body-tertiary border-bottom shadow-sm">
    <div class="container-fluid">

        <div class="collapse navbar-collapse justify-content-center" id="navbarNav">

            <ul class="navbar-nav mx-auto text-center gap-1">

                <!-- INICIO -->
                <li class="nav-item">
                    <a
                        class="nav-link nav-estilizado"
                        href="/inicio"
                    >
                        Inicio
                    </a>
                </li>


                <!-- USUARIOS Y ROLES -->
                {#if tieneModulo('Usuario')}
                    <li class="nav-item">
                        <a
                            class="nav-link nav-estilizado"
                            href="/usuario"
                        >
                            Usuarios
                        </a>
                    </li>
                {/if}


                <!-- ROL -->
                {#if tieneModulo('Rol')}
                    <li class="nav-item">
                        <a
                            class="nav-link nav-estilizado"
                            href="/rol"
                        >
                            Rol
                        </a>
                    </li>
                {/if}


                <!-- CARGA DOCENTE -->
                {#if tieneModulo('Carga Docente')}
                    <li class="nav-item">
                        <a
                            class="nav-link nav-estilizado"
                            href="/carga_docente"
                        >
                            Carga Docente
                        </a>
                    </li>
                {/if}


                <!-- ESTRUCTURA ACADÉMICA -->
                {#if tieneModulo('Estructura Académica')}
                    <li class="nav-item">
                        <a
                            class="nav-link nav-estilizado"
                            href="/Estructura_Académica"
                        >
                            Estructura Académica
                        </a>
                    </li>
                {/if}


                <!-- GRUPOS Y HORARIOS -->
                {#if tieneModulo('Horarios Grupos')}
                    <li class="nav-item">
                        <a
                            class="nav-link nav-estilizado"
                            href="/Grupos_Horarios"
                        >
                            Horarios Grupos
                        </a>
                    </li>
                {/if}


                <!-- EVALUACIÓN DOCENTE -->
                {#if tieneModulo('Evaluación Docente')}
                    <li class="nav-item">
                        <a
                            class="nav-link nav-estilizado"
                            href="/Evaluación_Desempeño"
                        >
                            Evaluación Docente
                        </a>
                    </li>
                {/if}


                <!-- ANALÍTICA -->
                {#if tieneModulo('Analítica')}
                    <li class="nav-item">
                        <a
                            class="nav-link nav-estilizado"
                            href="/Analitica"
                        >
                            Analítica
                        </a>
                    </li>
                {/if}

            </ul>
        </div>
    </div>
</nav>


<style>
    .nav-estilizado {
        position: relative;
        font-family: 'Segoe UI', system-ui, -apple-system, Roboto, sans-serif;
        font-weight: 600;
        font-size: 0.95rem;
        letter-spacing: 0.3px;
        color: #495057 !important;
        padding: 8px 16px !important;
        border-radius: 6px;
        transition: all 0.25s ease-in-out;
    }

    .nav-estilizado:hover {
        color: #0f3460 !important;
        background-color: rgba(15, 52, 96, 0.08);
        transform: translateY(-1px);
    }

    .nav-estilizado::after {
        content: '';
        position: absolute;
        bottom: 2px;
        left: 50%;
        width: 0;
        height: 2px;
        background-color: #0f3460;
        transition: width 0.25s ease, left 0.25s ease;
        border-radius: 2px;
    }

    .nav-estilizado:hover::after {
        width: 60%;
        left: 20%;
    }
</style>

