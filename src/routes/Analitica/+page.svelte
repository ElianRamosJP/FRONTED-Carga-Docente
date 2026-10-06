<script>
	import Navbar from '../../lib/components/navbar.svelte';
	import Footer from '../../lib/components/footer.svelte';
	import Header from '../../lib/components/header.svelte';    

	// Datos de prueba para KPIs
	const kpis = [
		{ titulo: 'Docentes Evaluados', valor: '142', subtexto: '94.6% del total', colorBorder: 'border-primary' },
		{ titulo: 'Carga Horaria Promedio', valor: '18.5 hrs', subtexto: 'Semanal / Docente', icono: '⏱️', colorBorder: 'border-info' },
		{ titulo: 'Índice de Satisfacción', valor: '91.8%', subtexto: '+2.4% vs periodo ant.', icono: '⭐', colorBorder: 'border-success' },
		{ titulo: 'Evaluaciones Pendientes', valor: '8', subtexto: 'Cierre en 3 días', icono: '⏳', colorBorder: 'border-warning' }
	];

	// Datos simulados para gráficos por facultad
	const facultadesPromedio = [
		{ nombre: 'Ingeniería', promedio: 4.8, porcentaje: 96 },
		{ nombre: 'Ciencias Económicas', promedio: 4.6, porcentaje: 92 },
		{ nombre: 'Ciencias Sociales', promedio: 4.4, porcentaje: 88 },
		{ nombre: 'Arquitectura y Diseño', promedio: 4.5, porcentaje: 90 },
		{ nombre: 'Ciencias de la Salud', promedio: 4.7, porcentaje: 94 }
	];
</script>

<Header />
<Navbar />

<main class="bg-light min-vh-100 p-4">
	<div class="container-fluid px-2 px-md-4">
		
		<!-- ENCABEZADO DE SECCIÓN CON ACCIONES -->
		<div class="card border-0 shadow-sm mb-4">
			<div class="card-body">
				<div class="d-flex flex-column flex-md-row justify-content-between align-items-md-center gap-3">
					<div class="d-flex align-items-center gap-3">
						<div class="rounded p-3 text-white fs-4" style="background-color: #0f3460;">
							📊
						</div>
						<div>
							<h2 class="fw-bold mb-1"> Analítica Institucional</h2>
							<p class="text-muted mb-0">Consolidado general de desempeño docente y distribución de carga académica</p>
						</div>
					</div>

					<!-- BOTONES DE EXPORTACIÓN -->
					<div class="d-flex gap-2">
						<button type="button" class="btn btn-outline-danger d-flex align-items-center gap-2 px-3 fw-semibold shadow-sm">
							Exportar PDF
						</button>
						
					</div>
				</div>
			</div>
		</div>

		<!-- FILA SUPERIOR: INDICADORES CLAVE (KPI CARDS) -->
		<div class="row g-3 mb-4">
			{#each kpis as kpi}
				<div class="col-12 col-sm-6 col-xl-3">
					<div class={`card border-0 border-start border-4 ${kpi.colorBorder} shadow-sm h-100`}>
						<div class="card-body d-flex align-items-center justify-content-between">
							<div>
								<span class="text-muted fw-semibold small text-uppercase d-block mb-1">{kpi.titulo}</span>
								<h3 class="fw-bold mb-0 text-dark">{kpi.valor}</h3>
								<small class="text-secondary">{kpi.subtexto}</small>
							</div>
							
						</div>
					</div>
				</div>
			{/each}
		</div>

		<!-- SECCIÓN CENTRAL DE ANALÍTICA (GRÁFICOS SIMULADOS) -->
		<div class="row g-4 mb-4">
			<!-- CONSOLIDADO DE DESEMPEÑO POR FACULTAD -->
			<div class="col-12 col-lg-6">
				<div class="card border-0 shadow-sm h-100">
					<div class="card-header bg-white border-bottom py-3 d-flex justify-content-between align-items-center">
						<h5 class="fw-bold mb-0 text-dark"> Desempeño Promedio por Facultad</h5>
						<span class="badge bg-light text-dark border">Escala 1.0 - 5.0</span>
					</div>
					<div class="card-body p-4">
						<div class="d-flex flex-column gap-3">
							{#each facultadesPromedio as fac}
								<div>
									<div class="d-flex justify-content-between align-items-center mb-1">
										<span class="fw-semibold text-dark small">{fac.nombre}</span>
										<span class="fw-bold text-dark small">{fac.promedio} / 5.0</span>
									</div>
									<div class="progress" style="height: 12px;">
										<div
											class="progress-bar"
											role="progressbar"
											style={`width: ${fac.porcentaje}%; background-color: #0f3460;`}
											aria-valuenow={fac.porcentaje}
											aria-valuemin="0"
											aria-valuemax="100"
										></div>
									</div>
								</div>
							{/each}
						</div>
					</div>
				</div>
			</div>

			<!-- DISTRIBUCIÓN DE CARGA ACADÉMICA -->
			<div class="col-12 col-lg-6">
				<div class="card border-0 shadow-sm h-100">
					<div class="card-header bg-white border-bottom py-3 d-flex justify-content-between align-items-center">
						<h5 class="fw-bold mb-0 text-dark"> Distribución de Carga Académica</h5>
						<span class="badge bg-light text-dark border">Horas Semanales</span>
					</div>
					<div class="card-body p-4 d-flex flex-column justify-content-center">
						<div class="row text-center g-3 mb-4">
							<div class="col-4">
								<div class="p-3 bg-light rounded border">
									<h4 class="fw-bold text-primary mb-1">62%</h4>
									<span class="small text-muted fw-semibold">Docencia Directa</span>
								</div>
							</div>
							<div class="col-4">
								<div class="p-3 bg-light rounded border">
									<h4 class="fw-bold text-info mb-1">23%</h4>
									<span class="small text-muted fw-semibold">Investigación</span>
								</div>
							</div>
							<div class="col-4">
								<div class="p-3 bg-light rounded border">
									<h4 class="fw-bold text-warning mb-1">15%</h4>
									<span class="small text-muted fw-semibold">Gestión / Extensión</span>
								</div>
							</div>
						</div>

						<!-- BARRA APILADA SIMULADA -->
						<div class="mb-2">
							<div class="progress-stacked" style="height: 20px;">
								<div class="progress" role="progressbar" style="width: 62%" aria-valuenow="62" aria-valuemin="0" aria-valuemax="100">
									<div class="progress-bar bg-primary">62%</div>
								</div>
								<div class="progress" role="progressbar" style="width: 23%" aria-valuenow="23" aria-valuemin="0" aria-valuemax="100">
									<div class="progress-bar bg-info text-dark">23%</div>
								</div>
								<div class="progress" role="progressbar" style="width: 15%" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100">
									<div class="progress-bar bg-warning text-dark">15%</div>
								</div>
							</div>
						</div>
						<span class="small text-muted text-center">Proporción general asignada para el periodo activo</span>
					</div>
				</div>
			</div>
		</div>

		

	</div>
</main>

<Footer />