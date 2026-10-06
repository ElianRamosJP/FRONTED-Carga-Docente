<script>
	import Navbar from '$lib/components/navbar.svelte';
	import Footer from '$lib/components/footer.svelte';
	import Header from '$lib/components/header.svelte';

	// Configuración de las tarjetas informativas del sistema
	const tarjetas = [
		{
			titulo: 'Carga Docente',
			descripcion: 'Asigna horas y cursos a cada docente por semestre.',
			icono: 'bi-journal-bookmark-fill'
		},
		{
			titulo: 'Evaluación de Desempeño',
			descripcion: 'Encuestas de par académico, autoevaluación y evaluación estudiantil.',
			icono: 'bi-star-fill'
		},
		{
			titulo: 'Estructura Académica',
			descripcion: 'Gestión de facultades, programas, planes de estudio y asignaturas.',
			icono: 'bi-diagram-3-fill'
		},
		{
			titulo: 'Analítica ',
			descripcion: 'Consolidados por facultad con indicadores visuales y exportación.',
			icono: 'bi-bar-chart-fill'
		}
	];
</script>

<svelte:head>
	<title>Inicio | Carga Docente</title>
</svelte:head>

<Header />
<Navbar />

<main class="bg-light min-vh-100 py-5">
	<div class="container">
		<!-- BLOQUE DE BIENVENIDA -->
		<div class="text-center mb-5">
			<div
				class="d-inline-flex align-items-center justify-content-center rounded-circle text-white mb-3 shadow-sm"
				style="width: 70px; height: 70px; background-color: #0f3460;"
			>
				<i class="bi bi-mortarboard-fill fs-1"></i>
			</div>
			<h2 class="fw-bold mb-2 text-dark">Bienvenido a Acta Docente</h2>
			<p class="text-secondary mx-auto mb-0" style="max-width: 600px;">
				Plataforma institucional para la gestión de carga académica, estructura universitaria y evaluación de desempeño docente.
			</p>
		</div>

		<!-- CUADRÍCULA DE 4 TARJETAS RESTRUCTURADA -->
		<div class="row g-4 justify-content-center">
			{#each tarjetas as item}
				<div class="col-12 col-sm-6 col-lg-3">
					<div class="card border-0 shadow-sm h-100 transition-hover">
						<div class="card-body text-center p-4 d-flex flex-column align-items-center">
							<div class="mb-3" style="color: #0f3460;">
								<i class={`bi ${item.icono} fs-1`}></i>
							</div>
							<h6 class="fw-bold text-dark mb-2">{item.titulo}</h6>
							<p class="small text-muted mb-0">{item.descripcion}</p>
						</div>
					</div>
				</div>
			{/each}
		</div>
	</div>
</main>

<Footer />

<style>
	/* Transición sutil al pasar el mouse sobre las tarjetas */
	.transition-hover {
		transition: transform 0.2s ease, box-shadow 0.2s ease;
	}
	.transition-hover:hover {
		transform: translateY(-4px);
		box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.1) !important;
	}
</style>